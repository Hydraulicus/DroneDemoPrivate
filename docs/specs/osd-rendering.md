# OSD Rendering Specification

**Status:** Implemented
**Last Updated:** December 2025
**Phase:** Phase 3 - Graphics Overlay

## Overview

The On-Screen Display (OSD) renderer provides real-time vector graphics overlay on top of the video feed. It uses NanoVG for hardware-accelerated, anti-aliased rendering of text, shapes, and telemetry data.

## Requirements

From PROJECT-SPEC.md:
- **FR-3**: Overlay graphics on video stream (rectangles, circles, text, lines) ✅
- **FR-3**: Update rate 30 Hz minimum ✅
- **FR-3**: Anti-aliased rendering ✅
- **NFR-1**: No impact on frame rate (<33ms frame budget) ✅

## Architecture

```
┌──────────────────────────────────────────────────────────┐
│                      Main Loop                            │
└────────────────────────┬─────────────────────────────────┘
                         │
         ┌───────────────┴───────────────┐
         ▼                               ▼
┌─────────────────┐             ┌─────────────────┐
│ TextureRenderer │             │   OSDRenderer   │
│  (Video Frame)  │             │    (NanoVG)     │
└────────┬────────┘             └────────┬────────┘
         │                               │
         │      ┌───────────────┐        │
         └─────▶│   OpenGL      │◀───────┘
                │  Framebuffer  │
                └───────────────┘
                         │
                         ▼
                ┌───────────────┐
                │    Display    │
                └───────────────┘
```

**Rendering Order:**
1. Video texture rendered first (background)
2. OSD rendered on top (overlay with alpha blending)
3. Buffers swapped to display

## Interface Design

### OSD Interface

```cpp
// src/core/osd.h

enum class TextAlign { Left, Center, Right };

struct Color {
    float r, g, b, a;
    static Color white();
    static Color black();
    static Color red();
    static Color green();
    static Color blue();
    static Color yellow();
    static Color cyan();
    static Color transparent(float alpha = 0.5f);
};

struct OSDConfig {
    std::string font_path;           // Path to TTF font file
    std::string font_bold_path;      // Path to bold TTF font (optional)
    float default_font_size = 18.0f; // Default font size in pixels
};

class IOSD {
public:
    virtual ~IOSD() = default;

    // Lifecycle
    virtual bool initialize(const OSDConfig& config) = 0;
    virtual void shutdown() = 0;
    virtual bool isInitialized() const = 0;

    // Frame Management
    virtual void beginFrame(int width, int height, float device_pixel_ratio = 1.0f) = 0;
    virtual void endFrame() = 0;

    // Text Rendering
    virtual void drawText(float x, float y, const std::string& text,
                          Color color, float size, TextAlign align) = 0;
    virtual void drawTextWithBackground(float x, float y, const std::string& text,
                                        Color text_color, Color bg_color,
                                        float padding, float size) = 0;

    // Shape Rendering
    virtual void drawRect(float x, float y, float w, float h, Color color) = 0;
    virtual void drawRectOutline(float x, float y, float w, float h,
                                 Color color, float stroke_width) = 0;
    virtual void drawRoundedRect(float x, float y, float w, float h,
                                 float radius, Color color) = 0;
    virtual void drawLine(float x1, float y1, float x2, float y2,
                          Color color, float width) = 0;
    virtual void drawCircle(float cx, float cy, float radius,
                            Color color, bool filled) = 0;

    // Convenience Methods
    virtual void drawFPS(float fps, int width) = 0;
    virtual void drawTimestamp(float x, float y) = 0;
    virtual void drawFrameCounter(uint32_t frame_number, float x, float y) = 0;
};

std::unique_ptr<IOSD> createOSD();
```

## Implementation Details

### NanoVG Integration

NanoVG is included as a git submodule in `third_party/nanovg/`. The library provides:
- Hardware-accelerated vector graphics
- Anti-aliased rendering
- TrueType font support via stb_truetype
- OpenGL 2.1 (macOS) and OpenGL ES 2.0 (Jetson) backends

### Platform-Specific Backend Selection

```cpp
// In osd_renderer.cpp
#include "core/opengl.h"

#ifdef PLATFORM_MACOS
    #define NANOVG_GL2_IMPLEMENTATION
#else
    #define NANOVG_GLES2_IMPLEMENTATION
#endif

#include <nanovg.h>
#include <nanovg_gl.h>
```

### Font Loading

Fonts are bundled with the application in `assets/fonts/`:
- `RobotoMono-Regular.ttf` - For general text display
- `RobotoMono-Bold.ttf` - For emphasis (FPS counter, titles)

The `ASSETS_PATH` macro (defined in CMakeLists.txt) provides the path at compile time.

### FPS Counter Color Coding

| FPS Range | Color | Meaning |
|-----------|-------|---------|
| >= 28 | Green | Good performance |
| 20-27 | Yellow | Marginal performance |
| < 20 | Red | Poor performance |

## File Structure

```
src/
├── core/
│   ├── osd.h              # IOSD interface ✅
│   └── opengl.h           # Platform-agnostic OpenGL includes ✅
├── osd/
│   ├── osd_renderer.h     # OSDRenderer header ✅
│   └── osd_renderer.cpp   # NanoVG implementation ✅
assets/
└── fonts/
    ├── RobotoMono-Regular.ttf  ✅
    └── RobotoMono-Bold.ttf     ✅
third_party/
└── nanovg/                # Git submodule ✅
```

## Main Loop Integration

```cpp
// In main.cpp

// After window and texture renderer initialization:
auto osd = createOSD();
OSDConfig osd_config;
osd_config.font_path = std::string(ASSETS_PATH) + "/fonts/RobotoMono-Regular.ttf";
osd_config.font_bold_path = std::string(ASSETS_PATH) + "/fonts/RobotoMono-Bold.ttf";
osd->initialize(osd_config);

// In main loop, after rendering video texture:
osd->beginFrame(fb_width, fb_height, pixel_ratio);
osd->drawFPS(current_fps, fb_width);
osd->drawTimestamp(10.0f, 10.0f);
osd->drawFrameCounter(total_frames, 10.0f, fb_height - 30.0f);
osd->endFrame();

// Before cleanup:
osd->shutdown();
```

## Error Handling

| Error Condition | Behavior | Recovery |
|-----------------|----------|----------|
| NanoVG context creation fails | Log error, return false | Cannot recover |
| Font file not found | Log error, return false | Check assets path |
| Font loading fails | Log warning, use fallback | Continue with other font |
| Drawing outside frame | Silently ignore | Begin new frame |

## Testing Strategy

### Manual Tests
- [x] OSD elements visible on video
- [x] FPS counter updates correctly
- [x] Timestamp shows current time with milliseconds
- [x] Frame counter increments
- [x] Text is readable and anti-aliased
- [x] Colors display correctly
- [x] Retina display support (pixel ratio)

### Verification Commands
```bash
# Build and run
cmake -B build && cmake --build build && ./build/robot_vision

# Expected output:
#   OSD renderer initialized
#   Loaded font 'regular' from: .../assets/fonts/RobotoMono-Regular.ttf
#   Loaded font 'bold' from: .../assets/fonts/RobotoMono-Bold.ttf
```

## Performance

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| OSD render time | <5ms | <1ms | ✅ |
| Memory (fonts) | <1MB | ~50KB | ✅ |
| No frame drops | Yes | Yes | ✅ |

## Implementation Log

- [Dec 2025] Created specification
- [Dec 2025] Added NanoVG as git submodule
- [Dec 2025] Downloaded and bundled Roboto Mono fonts
- [Dec 2025] Implemented IOSD interface
- [Dec 2025] Implemented OSDRenderer with NanoVG
- [Dec 2025] Integrated OSD into main loop
- [Dec 2025] Tested on macOS - working correctly
- [Dec 2025] Refactored OpenGL includes to core/opengl.h
- [Dec 2025] **Phase 3 Complete**

## Future Enhancements (Phase 4+)

- Detection/tracking boxes for computer vision
- Crosshair/reticle for targeting
- Telemetry panel (speed, position, battery)
- Interactive OSD elements
- Custom gauges and indicators

---

## Next Steps

1. Phase 4: OSD Features - Custom graphics elements
2. Phase 5: File Operations - Frame saving, configuration
