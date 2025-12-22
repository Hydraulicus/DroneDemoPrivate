# Video Pipeline Specification

**Status:** In Progress
**Last Updated:** December 2025
**Phase:** Phase 2 - Video Pipeline

## Overview

The video pipeline captures frames from the camera using GStreamer, and displays them in a GLFW window using OpenGL textures. This provides the foundation for OSD overlay in Phase 3.

## Requirements

From PROJECT-SPEC.md:
- **FR-1**: Capture video at 30 FPS minimum, 1280x720 default
- **FR-2**: Display video in real-time window, <100ms latency
- **NFR-1**: Frame processing latency <33ms

## Architecture

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Camera     │───▶│  GStreamer   │───▶│   AppSink    │
│  (Hardware)  │    │  Pipeline    │    │  (Frames)    │
└──────────────┘    └──────────────┘    └──────┬───────┘
                                               │
                                               ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Display    │◀───│   OpenGL     │◀───│    Frame     │
│   (Window)   │    │   Texture    │    │    Buffer    │
└──────────────┘    └──────────────┘    └──────────────┘
```

## Interface Design

### Video Pipeline Interface

```cpp
// src/core/video_pipeline.h

struct FrameData {
    std::vector<uint8_t> pixels;  // RGB pixel data
    int width;
    int height;
    uint64_t timestamp;           // Frame timestamp (nanoseconds)
    uint32_t frame_number;
};

struct PipelineConfig {
    int width = 1280;
    int height = 720;
    int fps = 30;
    std::string device = "";      // Empty = auto-detect
};

class IVideoPipeline {
public:
    virtual ~IVideoPipeline() = default;

    // Lifecycle
    virtual bool initialize(const PipelineConfig& config) = 0;
    virtual bool start() = 0;
    virtual void stop() = 0;

    // Frame access
    virtual std::shared_ptr<FrameData> getLatestFrame() = 0;
    virtual bool hasNewFrame() const = 0;

    // State
    virtual bool isRunning() const = 0;
    virtual std::string getState() const = 0;
    virtual std::string getLastError() const = 0;
};

std::unique_ptr<IVideoPipeline> createVideoPipeline(IPlatform& platform);
```

### Window Interface

```cpp
// src/core/window.h

struct WindowConfig {
    int width = 1280;
    int height = 720;
    std::string title = "Robot Vision Demo";
    bool resizable = true;
};

class IWindow {
public:
    virtual ~IWindow() = default;

    virtual bool initialize(const WindowConfig& config) = 0;
    virtual bool shouldClose() const = 0;
    virtual void pollEvents() = 0;
    virtual void swapBuffers() = 0;

    virtual int getWidth() const = 0;
    virtual int getHeight() const = 0;
    virtual void* getNativeHandle() const = 0;  // GLFWwindow*
};

std::unique_ptr<IWindow> createWindow();
```

## Implementation Details

### GStreamer Pipeline Flow

1. **Initialize GStreamer** (done in Phase 1)
2. **Create pipeline** from platform's pipeline string
3. **Get appsink element** for frame access
4. **Set pipeline to PLAYING state**
5. **Pull frames** from appsink in main loop
6. **Upload to OpenGL texture**
7. **Render textured quad**

### Frame Callback (Pull Model)

```cpp
// In video_pipeline.cpp
GstSample* sample = gst_app_sink_try_pull_sample(appsink, timeout_ns);
if (sample) {
    GstBuffer* buffer = gst_sample_get_buffer(sample);
    GstMapInfo map;
    gst_buffer_map(buffer, &map, GST_MAP_READ);

    // Copy pixel data
    frame->pixels.assign(map.data, map.data + map.size);

    gst_buffer_unmap(buffer, &map);
    gst_sample_unref(sample);
}
```

### OpenGL Texture Rendering

```cpp
// Upload frame to texture
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height,
             0, GL_RGB, GL_UNSIGNED_BYTE, pixels.data());

// Render full-screen quad with texture
```

## File Structure

```
src/
├── core/
│   ├── video_pipeline.h      # IVideoPipeline interface
│   └── window.h              # IWindow interface
├── video/
│   ├── gstreamer_pipeline.cpp  # GStreamer implementation
│   └── gstreamer_pipeline.h
├── rendering/
│   ├── glfw_window.cpp       # GLFW window implementation
│   ├── glfw_window.h
│   └── texture_renderer.cpp  # OpenGL texture rendering
└── main.cpp                  # Main loop
```

## Main Loop Design

```cpp
int main() {
    // 1. Create platform & window
    auto platform = createPlatform();
    auto window = createWindow();
    window->initialize({1280, 720, "Robot Vision Demo"});

    // 2. Create and start pipeline
    auto pipeline = createVideoPipeline(*platform);
    pipeline->initialize({1280, 720, 30});
    pipeline->start();

    // 3. Create texture renderer
    TextureRenderer renderer;
    renderer.initialize(1280, 720);

    // 4. Main loop
    while (!window->shouldClose()) {
        window->pollEvents();

        if (pipeline->hasNewFrame()) {
            auto frame = pipeline->getLatestFrame();
            renderer.updateTexture(frame->pixels);
        }

        renderer.render();
        window->swapBuffers();
    }

    // 5. Cleanup
    pipeline->stop();
    return 0;
}
```

## Error Handling

| Error Condition | Behavior | Recovery |
|-----------------|----------|----------|
| Camera not found | Log error, return false from initialize() | Show error message |
| Pipeline fails to start | Log GStreamer error, return false | User retry |
| Frame timeout | Return nullptr from getLatestFrame() | Skip frame, continue |
| Window creation fails | Log error, exit | Cannot recover |

## Testing Strategy

### Manual Tests
- [ ] Window opens at correct size
- [ ] Camera feed displays in window
- [ ] Window close button works
- [ ] No memory leaks (run for 5 minutes)
- [ ] Frame rate is smooth (~30 FPS)

### Verification Commands
```bash
# Check for memory leaks
leaks --atExit -- ./build/robot_vision

# Monitor frame rate (add FPS counter in Phase 4)
```

## Performance Targets

| Metric | Target | Notes |
|--------|--------|-------|
| Frame rate | 30 FPS | Consistent, no drops |
| Latency | <100ms | Capture to display |
| CPU usage | <20% | On Mac |
| Memory | <100MB | Stable, no growth |

## Implementation Log

- [Dec 2025] Created specification
- [Dec 2025] Starting implementation

---

## Next Steps After This Spec

1. OSD Renderer interface (`docs/specs/osd-rendering.md`)
