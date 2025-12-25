# Robot Vision Demo - Project Specification

## Project Overview

A cross-platform robot vision application demonstrating video capture, real-time processing, on-screen display (OSD) overlay, and data persistence. Designed for development on macOS with seamless deployment to embedded Linux platforms (Jetson Nano).

**Version:** 1.0  
**Date:** December 2025  
**Status:** Specification Phase

---

## 1. Purpose and Goals

### Primary Purpose
Create a demonstration application showcasing a production-ready robot vision pipeline with real-time graphics overlay and data logging capabilities.

### Key Goals
1. **Cross-platform compatibility**: Develop on macOS, deploy to Jetson Nano with minimal changes
2. **Real-time performance**: Low-latency video processing and rendering
3. **Modular architecture**: Clean separation of concerns for maintainability
4. **Hardware acceleration**: Utilize GPU capabilities on both platforms
5. **Production patterns**: Demonstrate best practices for robotics software

---

## 2. Target Platforms

### Development Platform
- **OS**: macOS (current development machine)
- **Architecture**: x86_64 or ARM64 (Apple Silicon)
- **Graphics**: OpenGL 2.1+ (macOS native)
- **Camera**: Built-in webcam or USB camera

### Target Deployment Platform
- **Device**: NVIDIA Jetson Nano
- **OS**: Linux (JetPack SDK)
- **Architecture**: ARM64 (Cortex-A57)
- **Graphics**: OpenGL ES 2.0 (GLES2)
- **Camera**: CSI camera or USB camera

---

## 3. Technical Stack

### Core Technologies
- **Language**: C++17 (minimum)
- **Build System**: CMake 3.15+
- **Platform**: Linux (target), macOS (development)

### Video Processing
- **Framework**: GStreamer 1.14+
- **Pipeline Construction**: `gst_parse_launch()` for dynamic plugin composition
- **Plugins**: Platform-specific plugins for optimal performance
  - macOS: `autovideosrc`, `videoconvert`, standard plugins
  - Jetson: `nvarguscamerasrc`, `nvvidconv`, NVIDIA accelerated plugins

### Graphics Rendering
- **OSD Library**: NanoVG (vector graphics)
- **Graphics Engine**: 
  - macOS: OpenGL 2.1+
  - Jetson Nano: OpenGL ES 2.0 (GLES2)
- **Use Case**: Overlay telemetry, debug info, UI elements on video

### Data Persistence
- **Storage**: File system I/O
- **Formats**: 
  - Configuration: JSON/XML
  - Video/Images: Standard formats (MP4, PNG, JPEG)
  - Logs: Plain text or structured logs

---

## 4. System Architecture

### Component Overview

```
┌─────────────────────────────────────────────────────────┐
│                    Application Layer                     │
│                      (main.cpp)                          │
└────────────────────┬────────────────────────────────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
┌───────▼──────────┐    ┌────────▼─────────┐
│  Video Pipeline  │    │   OSD Renderer   │
│   (GStreamer)    │◄───┤    (NanoVG)      │
└───────┬──────────┘    └────────┬─────────┘
        │                        │
        │                        │
┌───────▼────────────────────────▼─────────┐
│        Platform Abstraction Layer        │
│  (Graphics Backend, Camera, Display)     │
└──────────────────────────────────────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
┌───────▼──────┐        ┌────────▼────────┐
│  File System │        │  Hardware/OS    │
│   (Storage)  │        │   (Video/GPU)   │
└──────────────┘        └─────────────────┘
```

### Component Responsibilities

#### 4.1 Video Pipeline Component
**Responsibilities:**
- Initialize GStreamer framework
- Create and manage video pipeline
- Capture frames from camera source
- Provide frame buffer access to other components
- Handle pipeline state transitions

**Key Interfaces:**
- `initialize(PipelineConfig)` - Setup platform-specific pipeline
- `start()` - Begin video capture
- `getFrame()` - Retrieve current frame buffer
- `stop()` - Shutdown pipeline gracefully

#### 4.2 OSD Renderer Component
**Responsibilities:**
- Initialize NanoVG rendering context
- Draw vector graphics overlays
- Render text, shapes, telemetry data
- Composite graphics onto video frames
- Manage rendering performance

**Key Interfaces:**
- `initialize(GraphicsContext)` - Setup rendering backend
- `drawOverlay(Frame, OSDData)` - Render graphics on frame
- `setStyle(StyleConfig)` - Configure visual appearance
- `cleanup()` - Release graphics resources

#### 4.3 Platform Abstraction Layer
**Responsibilities:**
- Detect current platform (macOS vs Linux/Jetson)
- Provide platform-specific implementations
- Abstract graphics backend differences (OpenGL vs GLES2)
- Handle camera source variations
- Manage display output methods

**Key Interfaces:**
- `Platform::detect()` - Identify current platform
- `Platform::getGraphicsBackend()` - Return appropriate backend
- `Platform::getCameraSource()` - Return camera pipeline string
- `Platform::getDisplaySink()` - Return display pipeline string

#### 4.4 Storage Component
**Responsibilities:**
- Save processed frames to files
- Manage configuration files
- Handle logging and telemetry data
- Ensure data integrity

**Key Interfaces:**
- `saveFrame(Frame, filename)` - Write image to disk
- `saveConfig(Config, filename)` - Write configuration
- `appendLog(LogEntry)` - Add log entry

---

## 5. Core Requirements

### Functional Requirements

**FR-1: Video Capture**
- System SHALL capture video from camera source
- Frame rate: 30 FPS minimum
- Resolution: Configurable, default 1280x720

**FR-2: Video Display**
- System SHALL display video in real-time window
- Latency: <100ms from capture to display
- Window size: Resizable

**FR-3: OSD Overlay**
- System SHALL overlay graphics on video stream
- Graphics types: Rectangles, circles, text, lines
- Update rate: 30 Hz minimum
- Anti-aliased rendering

**FR-4: Data Persistence**
- System SHALL save processed frames on demand
- System SHALL save configuration to file
- System SHALL log operational events

**FR-5: Cross-Platform Operation**
- System SHALL run on macOS during development
- System SHALL run on Jetson Nano in deployment
- Code changes between platforms: <10% of codebase

### Non-Functional Requirements

**NFR-1: Performance**
- CPU usage: <40% on Jetson Nano
- Memory footprint: <256MB
- Frame processing latency: <33ms (for 30 FPS)

**NFR-2: Reliability**
- Graceful handling of camera disconnection
- Pipeline recovery from errors
- No memory leaks during extended operation

**NFR-3: Maintainability**
- Modular architecture with clear interfaces
- Comprehensive error messages
- Code documentation (comments + docs)

**NFR-4: Portability**
- Single codebase for both platforms
- Platform-specific code isolated in abstraction layer
- CMake-based build system

---

## 6. GStreamer Pipeline Specifications

### macOS Development Pipeline
```bash
# Basic video capture and display
autovideosrc ! videoconvert ! video/x-raw,format=RGB ! appsink

# With OSD overlay
autovideosrc ! videoconvert ! video/x-raw,format=RGB ! 
  appsink name=sink emit-signals=true
```

### Jetson Nano Deployment Pipeline
```bash
# CSI camera with hardware acceleration
nvarguscamerasrc ! 'video/x-raw(memory:NVMM),width=1280,height=720,
  format=NV12,framerate=30/1' ! nvvidconv ! 
  'video/x-raw,format=RGB' ! appsink name=sink

# USB camera fallback
v4l2src device=/dev/video0 ! videoconvert ! 
  video/x-raw,format=RGB ! appsink name=sink
```

### Pipeline Requirements
- Must use `appsink` for frame access
- RGB format for NanoVG compatibility
- Configurable resolution and framerate
- Error handling for missing cameras

---

## 7. NanoVG Integration Specifications

### Graphics Context Setup

**macOS:**
```cpp
#define NANOVG_GL2_IMPLEMENTATION
NVGcontext* vg = nvgCreateGL2(NVG_ANTIALIAS | NVG_STENCIL_STROKES);
```

**Jetson Nano:**
```cpp
#define NANOVG_GLES2_IMPLEMENTATION
NVGcontext* vg = nvgCreateGLES2(NVG_ANTIALIAS | NVG_STENCIL_STROKES);
```

### OSD Drawing Requirements
- Coordinate system: Matches video frame dimensions
- Drawing primitives: Lines, rectangles, circles, text
- Color format: RGBA with alpha blending
- Font rendering: TrueType font support
- Performance: All drawing within single frame budget

### Typical OSD Elements
- Frame counter
- Timestamp
- FPS indicator
- Detection boxes/regions (for future CV integration)
- Status text
- Debug information

---

## 8. File Storage Specifications

### Output Directory Structure
```
output/
├── frames/
│   ├── frame_0001.png
│   ├── frame_0002.png
│   └── ...
├── logs/
│   └── app.log
└── config/
    └── settings.json
```

### Frame Saving
- **Format**: PNG (lossless) or JPEG (compressed)
- **Naming**: Sequential or timestamp-based
- **Metadata**: Optional EXIF data (timestamp, camera info)
- **Trigger**: On-demand (keyboard/API) or periodic

### Configuration Files
- **Format**: JSON for easy editing
- **Contents**: Pipeline config, OSD settings, camera parameters
- **Location**: `config/` directory
- **Auto-save**: On application exit

### Logging
- **Format**: Plain text with timestamps
- **Levels**: INFO, WARNING, ERROR
- **Rotation**: Optional log rotation for long-running apps

---

## 9. Platform Abstraction Design

### Abstraction Strategy

Use conditional compilation and runtime detection:

```cpp
// Platform detection
#ifdef __APPLE__
    #define PLATFORM_MACOS
#elif defined(__linux__)
    #define PLATFORM_LINUX
    // Further detection for Jetson
    #ifdef __aarch64__
        #define PLATFORM_JETSON
    #endif
#endif

// Graphics backend selection
#ifdef PLATFORM_MACOS
    #define NANOVG_GL2_IMPLEMENTATION
    #define GRAPHICS_BACKEND "OpenGL 2.1"
#else
    #define NANOVG_GLES2_IMPLEMENTATION
    #define GRAPHICS_BACKEND "OpenGL ES 2.0"
#endif
```

### Platform-Specific Modules

**Header (platform.h):**
```cpp
class Platform {
public:
    static PlatformType detect();
    static std::string getCameraPipeline();
    static std::string getDisplayPipeline();
    static NVGcontext* createGraphicsContext();
};
```

**Implementations:**
- `platform_macos.cpp` - macOS specifics
- `platform_linux.cpp` - Generic Linux
- `platform_jetson.cpp` - Jetson optimizations

---

## 10. Build System Specifications

### CMake Requirements
- Minimum version: 3.15
- Platform detection
- Dependency management
- Cross-compilation support

### Build Configuration Example
```cmake
cmake_minimum_required(VERSION 3.15)
project(RobotVisionDemo CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Platform detection
if(APPLE)
    set(PLATFORM_MACOS TRUE)
    find_package(OpenGL REQUIRED)
elseif(UNIX)
    set(PLATFORM_LINUX TRUE)
    find_package(OpenGLES2 REQUIRED)
endif()

# Dependencies
find_package(PkgConfig REQUIRED)
pkg_check_modules(GSTREAMER REQUIRED gstreamer-1.0)
pkg_check_modules(GSTREAMER_APP REQUIRED gstreamer-app-1.0)

# Source files
add_executable(robot_vision_demo
    src/main.cpp
    src/video_pipeline.cpp
    src/osd_renderer.cpp
    src/storage.cpp
    src/platform/${PLATFORM_NAME}.cpp
)

# Link libraries
target_link_libraries(robot_vision_demo
    ${GSTREAMER_LIBRARIES}
    ${OPENGL_LIBRARIES}
    nanovg
)
```

### Build Targets
- `robot_vision_demo` - Main executable
- `tests` - Unit tests (optional)
- `install` - Installation target

---

## 11. Development Phases

### Phase 1: Foundation (Week 1)
- ✅ Create project structure
- ✅ Setup CMake build system
- ✅ Implement platform detection
- ✅ Basic GStreamer initialization

### Phase 2: Video Pipeline (Week 1-2)
- ✅ Implement video capture
- ✅ Display video in window
- ✅ Platform-specific pipeline configuration
- ✅ Error handling

### Phase 3: Graphics Overlay (Week 2) ✅ COMPLETE
- ✅ Integrate NanoVG
- ✅ Initialize graphics context
- ✅ Draw basic shapes on video
- ✅ Text rendering

### Phase 4: OSD Features (Week 3) - IN PROGRESS
- ✅ Frame counter
- ✅ FPS display
- ⬜ Custom graphics elements (detection boxes, crosshair)
- ⬜ Dynamic data display (telemetry panel)

### Phase 5: File Operations (Week 3)
- ⬜ Frame saving functionality
- ⬜ Configuration management
- ⬜ Logging system

### Phase 6: Testing & Optimization (Week 4)
- ⬜ Test on macOS
- ⬜ Test on Jetson Nano
- ⬜ Performance profiling
- ⬜ Documentation

---

## 12. Success Criteria

### Minimum Viable Product (MVP)
- ✅ Displays camera video in window on both platforms
- ✅ Overlays simple graphics (rectangle + text)
- ⬜ Saves frames to file on command
- ✅ Less than 50 lines of platform-specific code

### Full Success
- ⬜ All functional requirements met
- ⬜ All non-functional requirements met
- ✅ Clean, documented codebase
- ⬜ Build instructions for both platforms
- ⬜ Performance benchmarks documented

---

## 13. Dependencies and Prerequisites

### Development Environment
- **macOS**: Xcode Command Line Tools, Homebrew
- **CLion IDE**: JetBrains CLion (free trial / educational license)
- **CMake**: 3.15+
- **Git**: Version control

### Libraries (macOS)
```bash
brew install cmake pkg-config
brew install gstreamer gst-plugins-base gst-plugins-good
brew install glfw3  # For window creation
```

### Libraries (Jetson Nano)
```bash
# GStreamer (pre-installed with JetPack)
# OpenGL ES 2.0 (pre-installed)
# Additional packages
sudo apt install libglfw3-dev cmake build-essential
```

### NanoVG
- Source: https://github.com/memononen/nanovg
- Build from source or include as submodule

---

## 14. Risk Assessment

### Technical Risks

**Risk 1: Graphics Backend Compatibility**
- **Impact**: High
- **Likelihood**: Medium
- **Mitigation**: Use platform abstraction, test early on both platforms

**Risk 2: GStreamer Pipeline Differences**
- **Impact**: Medium
- **Likelihood**: High
- **Mitigation**: Document pipelines, create platform-specific configs

**Risk 3: Performance on Jetson Nano**
- **Impact**: High
- **Likelihood**: Medium
- **Mitigation**: Profile early, use hardware acceleration, optimize drawing

**Risk 4: Camera Compatibility**
- **Impact**: Medium
- **Likelihood**: Medium
- **Mitigation**: Support multiple camera sources, provide fallbacks

---

## 15. Future Enhancements

### Potential Extensions
1. **Computer Vision Integration**: Add OpenCV for object detection
2. **Network Streaming**: Stream video over network (RTP/RTSP)
3. **Multiple Cameras**: Support multi-camera input
4. **Recording**: Continuous video recording to file
5. **ROS Integration**: Integrate with Robot Operating System
6. **Touch Interface**: Interactive OSD elements
7. **GPU Compute**: Use CUDA on Jetson for CV processing

---

## 16. References and Resources

### Documentation
- GStreamer: https://gstreamer.freedesktop.org/documentation/
- NanoVG: https://github.com/memononen/nanovg
- Jetson Developer Guide: https://developer.nvidia.com/embedded/jetson-nano-developer-kit

### Examples
- GStreamer tutorials: https://gstreamer.freedesktop.org/documentation/tutorials/
- NanoVG examples: https://github.com/memononen/nanovg/tree/master/example

---

## 17. Glossary

- **OSD**: On-Screen Display - Graphics overlaid on video
- **GStreamer**: Multimedia framework for building pipelines
- **NanoVG**: Small antialiased vector graphics rendering library
- **GLES2**: OpenGL ES 2.0 - Graphics API for embedded systems
- **CSI**: Camera Serial Interface - Hardware camera interface on Jetson
- **Appsink**: GStreamer element for accessing frames in application
- **Pipeline**: Chain of GStreamer elements processing video

---

## Document Control

**Author**: Project Team  
**Reviewers**: TBD  
**Approved By**: TBD  
**Last Updated**: December 2025  
**Version History**:
- v1.0 (Dec 2025) - Initial specification

---

**End of Specification Document**
