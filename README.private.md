# DroneDemo - Private Developer Notes

Personal reference for repository management and development commands.

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

### Both Repos - Quick Sync

```bash
# Push both repos at once
cd /Users/olexii/work/CLionProjects/drone1/private && git add . && git commit -m "Update specs" && git push && cd .. && git add . && git commit -m "Sync" && git push
```

---

## Build & Run

### One-Line Build and Run

```bash
cd /Users/olexii/work/CLionProjects/drone1 && cmake -B build && cmake --build build && ./build/robot_vision
```

### Quick Rebuild Alias

Add to `~/.zshrc` or `~/.bashrc`:

```bash
alias rb='cmake --build build && ./build/robot_vision'
```

Then just type `rb` in project directory to rebuild and run.

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
- [ ] Phase 2: Video Pipeline - Camera capture, GLFW window, display
- [ ] Phase 3: Graphics Overlay - NanoVG integration
- [ ] Phase 4: OSD Features - Frame counter, FPS, telemetry
- [ ] Phase 5: File Operations - Frame saving, config, logging
- [ ] Phase 6: Testing & Optimization

---

## Quick Reference

| Action | Command |
|--------|---------|
| Build & run | `cmake -B build && cmake --build build && ./build/robot_vision` |
| Quick rebuild | `rb` (after adding alias) |
| Check tokens | `/cost` |
| Switch to Opus | `/model opus` |
| Pull all | `git pull --recurse-submodules` |

---

*Last updated: December 2025*
