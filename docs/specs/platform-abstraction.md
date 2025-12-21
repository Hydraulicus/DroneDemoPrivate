# Platform Abstraction Specification

**Status:** Complete
**Last Updated:** December 2025
**Phase:** Foundation (Phase 1)

## Overview

The platform abstraction layer isolates all platform-specific code (macOS vs Jetson/Linux) behind clean interfaces. This allows the rest of the application to be platform-agnostic.

## Requirements

From PROJECT-SPEC.md:
- **FR-5**: Code changes between platforms < 10% of codebase
- **NFR-4**: Single codebase for both platforms, platform-specific code isolated

## Interface Design

### Platform Detection

```cpp
// src/core/platform.h

enum class PlatformType {
    MacOS,      // Development platform
    Jetson,     // NVIDIA Jetson Nano (ARM64 Linux)
    Linux,      // Generic Linux (fallback)
    Unknown
};

struct PlatformInfo {
    PlatformType type;
    std::string name;           // "macOS", "Jetson Nano", etc.
    std::string graphics_api;   // "OpenGL 2.1", "OpenGL ES 2.0"
    bool has_gpu_acceleration;
};

class IPlatform {
public:
    virtual ~IPlatform() = default;

    // Get platform information
    virtual PlatformInfo getInfo() const = 0;
    virtual std::string getName() const = 0;

    // Get GStreamer pipeline strings for this platform
    virtual std::string getCameraPipeline(int width, int height, int fps) const = 0;
    virtual std::string getDisplayPipeline() const = 0;

    // Check capabilities
    virtual bool hasCamera() const = 0;
    virtual bool supportsResolution(int width, int height) const = 0;

    // Graphics context (for NanoVG)
    virtual GraphicsAPI getGraphicsAPI() const = 0;
    virtual void* createGraphicsContext() const = 0;      // Returns NVGcontext*
    virtual void destroyGraphicsContext(void* ctx) const = 0;
};

// Factory function - returns platform-appropriate implementation
std::unique_ptr<IPlatform> createPlatform();
```

## Implementation Details

### Compile-Time Platform Detection

```cpp
#if defined(__APPLE__)
    #define PLATFORM_MACOS 1
#elif defined(__linux__)
    #if defined(__aarch64__)
        // ARM64 Linux - likely Jetson
        #define PLATFORM_JETSON 1
    #else
        #define PLATFORM_LINUX 1
    #endif
#endif
```

### Platform-Specific GStreamer Pipelines

| Platform | Camera Source | Video Convert | Notes |
|----------|--------------|---------------|-------|
| macOS | `autovideosrc` | `videoconvert` | Uses AVFoundation |
| Jetson (CSI) | `nvarguscamerasrc` | `nvvidconv` | Hardware accelerated |
| Jetson (USB) | `v4l2src` | `videoconvert` | Fallback |

### Platform-Specific Graphics

| Platform | API | NanoVG Backend |
|----------|-----|----------------|
| macOS | OpenGL 2.1+ | `nvgCreateGL2()` |
| Jetson | OpenGL ES 2.0 | `nvgCreateGLES2()` |

## File Structure

```
src/
├── core/
│   └── platform.h           # IPlatform interface
├── platform/
│   ├── platform_factory.cpp # Factory implementation
│   ├── platform_macos.cpp   # macOS implementation
│   └── platform_linux.cpp   # Linux/Jetson implementation
```

## Testing Strategy

### Unit Tests
- [x] Platform detection returns correct type on macOS
- [x] Camera pipeline string is valid GStreamer syntax
- [x] Factory creates correct implementation

### Manual Tests
- [x] Build and run on macOS
- [ ] (Later) Build and run on Jetson Nano

## Error Handling

| Error Condition | Behavior | Recovery |
|-----------------|----------|----------|
| Unknown platform | Return `PlatformType::Unknown` | Log warning, use safe defaults |
| No camera | `hasCamera()` returns false | Application can show test pattern |

## Open Questions

- [x] Should we detect Jetson model (Nano vs Xavier)? **Decision: Not needed for MVP**
- [ ] Should platform provide audio pipeline strings too?

## Implementation Log

- [Dec 2025] Created specification
- [Dec 2025] Implemented IPlatform interface with factory pattern
- [Dec 2025] Implemented macOS platform (platform_macos.cpp)
- [Dec 2025] Implemented Linux/Jetson platform stub (platform_linux.cpp)
- [Dec 2025] Added GStreamer initialization to main.cpp
- [Dec 2025] Added createGraphicsContext()/destroyGraphicsContext() stubs
- [Dec 2025] **Phase 1 Complete** - All foundation requirements met

---

## Next Steps After This Spec

1. Video Pipeline interface (`docs/specs/video-pipeline.md`)
2. OSD Renderer interface (`docs/specs/osd-rendering.md`)
