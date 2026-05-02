# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Linux.do forum automation tool (v8.5.0) that auto-browses topics, likes posts, and tracks trust-level progress. Written in Python using DrissionPage for browser automation and Tkinter for the GUI.

## Running

```bash
# Install dependencies
pip install DrissionPage pystray pillow

# GUI version (primary)
python linux_do_gui.py

# Headless version (CLI, for servers/GitHub Actions)
python linux_do_headless.py -u USERNAME -p PASSWORD
python linux_do_headless.py -u USERNAME -p PASSWORD --topics 50 --like-rate 20 --proxy 127.0.0.1:7897

# Docker (from docker/ directory)
cd docker && cp .env.example .env  # edit .env first
docker-compose up -d

# Build executable
python build.py
```

## Architecture

### Entry Points

- `linux_do_gui.py` — Main application. Contains two classes: `Bot` (line 230) handles all browser automation logic; `GUI` (line 1419) is the Tkinter interface. Bot is not standalone — it's tightly coupled to GUI for status callbacks and logging.
- `linux_do_headless.py` — Standalone headless version. `LinuxDoBot` (line 161) is a self-contained bot with its own `Logger`. Accepts CLI args via `argparse` or environment variables (`LINUXDO_USERNAME`, `LINUXDO_PASSWORD`).
- `docker/linux_do_docker.py` — Docker-specific version with `LinuxDoBot` (line 83) and `RandomScheduler` (line 323) for randomized daily scheduling. Depends on the `schedule` package.

### Legacy/Testing

- `linux_do_auto_browse.py` — Older v2.0 script, simpler single-file automation. Not used by current versions.
- `test_floor_climbing.py` — Manual test script for the floor-climbing (deep reading) mode.

### Key Patterns

- **Browser automation**: All scripts use DrissionPage (`ChromiumPage`, `ChromiumOptions`). Anti-detection via `--disable-blink-features=AutomationControlled`. Page interactions use a mix of CSS selectors and `page.run_js()` for JavaScript execution.
- **Reading progress**: Fetched from `connect.linux.do` using JavaScript injection. Floor tracking uses `.timeline-replies` (wide window) or `#topic-progress .nums` (narrow window) selectors.
- **Unread topic detection**: v8.5 uses `badge-notification new-topic` CSS class to identify unread topics and verifies the badge disappears after reading.
- **Categories**: Forum sections are hardcoded as `CATEGORIES` lists with `name`/`url` pairs. Some scripts also use Discourse category IDs in URLs.

### CI/CD

- `.github/workflows/build-pyinstaller.yml` — Multi-platform build (Linux, macOS-ARM, Windows) via PyInstaller. Auto-creates GitHub Release on tag push (`v*`).
- `.github/workflows/run-schedule.yml` — Manual-trigger workflow that runs `linux_do_headless.py` on `ubuntu-latest`. Scheduled cron triggers are currently commented out.

## Dependencies

- `DrissionPage>=4.0.0` — Browser automation (Chromium-based)
- `Pillow>=10.0.0` — Image processing for system tray icons
- `pystray>=0.19.0` — System tray support (disabled on macOS)
- `schedule` — Only used by the Docker version for daily scheduling
