# DroneDemo - Private Developer Notes

Personal reference for repository management and development commands.

---

## Quick Start - Both Services

```bash
# Terminal 1: Start vision-detector
cd /Users/olexii/work/CLionProjects/vision-detector
./build/vision_detector -m private/models/custom_model_lite_tanks/detect.tflite -l private/models/custom_model_lite_tanks/labelmap.txt

# Terminal 2: Start robot_vision
cd /Users/olexii/work/CLionProjects/drone1
./build/robot_vision
```

---

## Build Commands

### Robot Vision (drone1)

```bash
# Full build
cd /Users/olexii/work/CLionProjects/drone1
cmake -B build && cmake --build build

# Run
./build/robot_vision
```

### Vision Detector

```bash
# Build with TFLite
cd /Users/olexii/work/CLionProjects/vision-detector
cmake -B build -DUSE_TFLITE=ON && cmake --build build

# Build without TFLite (for testing IPC only)
cmake -B build -DUSE_TFLITE=OFF && cmake --build build

# Run
./build/vision_detector -m private/models/custom_model_lite_tanks/detect.tflite -l private/models/custom_model_lite_tanks/labelmap.txt
```

### Quick Rebuild Aliases

Add to `~/.zshrc` or `~/.bashrc`:

```bash
alias rb='cd /Users/olexii/work/CLionProjects/drone1 && cmake --build build && ./build/robot_vision'
alias vd='cd /Users/olexii/work/CLionProjects/vision-detector && cmake --build build && ./build/vision_detector -m private/models/custom_model_lite_tanks/detect.tflite -l private/models/custom_model_lite_tanks/labelmap.txt'
```

---

## Repository Commands

### Public Repo (DroneDemo)

```bash
# Pull latest (including submodule updates)
cd /Users/olexii/work/CLionProjects/drone1
git pull --recurse-submodules

# Push changes
git add .
git commit -m "Your message"
git push

# Update submodule to latest private repo version
git submodule update --remote
git add private
git commit -m "Update private submodule"
git push
```

### Private Repo (DroneDemoPrivate)

```bash
# Pull latest
cd /Users/olexii/work/CLionProjects/drone1/private
git pull

# Push changes
git add .
git commit -m "Your message"
git push
```

### Vision Detector Repo

```bash
cd /Users/olexii/work/CLionProjects/vision-detector
git pull
git add .
git commit -m "Your message"
git push
```

### Both Repos - Quick Sync

```bash
# Push both repos at once
cd /Users/olexii/work/CLionProjects/drone1/private && git add . && git commit -m "Update specs" && git push && cd .. && git add . && git commit -m "Sync" && git push
```

---

## Claude Code Commands

### Switch Model

```bash
/model                    # Show current model, select new one
/model opus               # Switch to Opus
/model sonnet             # Switch to Sonnet
/model haiku              # Switch to Haiku
```

### Check Tokens / Usage

```bash
/cost                     # Show tokens used and cost in current session
```

### Other Useful Commands

```bash
/help                     # Show all available commands
/clear                    # Clear conversation history
/compact                  # Compact conversation to save context
/config                   # Show configuration
```

---

## Project Phases

- [x] Phase 1: Foundation - Platform abstraction, GStreamer init
- [x] Phase 2: Video Pipeline - Camera capture, GLFW window, display
- [x] Phase 3: Graphics Overlay - NanoVG OSD implementation
- [ ] Phase 4: Object Detection - TFLite integration, IPC communication
  - [x] Milestone 1: IPC connection, handshake, heartbeat
  - [ ] Milestone 2: Frame transfer and detection results
  - [ ] Milestone 3: OSD bounding box rendering
- [ ] Phase 5: File Operations - Frame saving, config, logging
- [ ] Phase 6: Testing & Optimization

---

## Architecture

```
┌─────────────────┐         IPC          ┌──────────────────┐
│  robot_vision   │◄───────────────────►│  vision_detector  │
│  (drone1)       │   Unix Socket        │                   │
│                 │   Shared Memory      │   TFLite Model    │
│  - Camera       │                      │   - detect.tflite │
│  - Display      │                      │   - Inference     │
│  - OSD          │                      │                   │
└─────────────────┘                      └──────────────────┘
```

---

## Quick Reference

| Action | Command |
|--------|---------|
| Build robot_vision | `cmake -B build && cmake --build build` |
| Run robot_vision | `./build/robot_vision` |
| Build vision_detector | `cmake -B build -DUSE_TFLITE=ON && cmake --build build` |
| Run vision_detector | `./build/vision_detector -m private/models/.../detect.tflite` |
| Check tokens | `/cost` |
| Switch to Opus | `/model opus` |
| Pull all | `git pull --recurse-submodules` |

---

*Last updated: December 2025*
