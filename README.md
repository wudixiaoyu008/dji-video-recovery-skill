# Claude Skill: Video Recovery

A Claude Code skill for recovering corrupted MP4 video files caused by premature camera or drone shutdown. Works with DJI, GoPro, Sony, and any camera that produces truncated MP4s with a missing or broken moov atom.

## What it does

When you power off a camera mid-recording, the video file gets written without its metadata header (moov atom), making it unplayable. This skill walks Claude through diagnosing the file, installing the necessary tools, and reconstructing the video using a healthy reference clip from the same camera.

## Supported cameras

- DJI drones (Mini 3, Mini 4 Pro, Air 3, Mavic series, etc.)
- GoPro (Hero series)
- Sony cameras
- Any camera that records MP4 and was shut down mid-recording

## Requirements

- macOS with Homebrew
- A working reference video from the same camera, same recording settings (resolution, FPS)
- The corrupted file copied to your computer

> Windows users: use WSL2 and follow the same steps inside it.

## Installation

1. Clone or download this repo
2. Copy `SKILL.md` into your Claude Code skills directory:

```bash
cp SKILL.md ~/.claude/skills/video-recovery/SKILL.md
```

3. That's it. Claude Code will pick it up automatically.

## Usage

Just tell Claude:

> "I have a corrupted video file I can't open, can you help me recover it?"

Claude will ask for the broken file path, a reference file, and your camera model, then handle the rest — including installing tools and building from source if needed.

## How it works

Under the hood, the skill uses **untrunc** (built from source) with ffmpeg to reconstruct the broken file's container by borrowing codec parameters from the healthy reference file. The recovered file is saved next to the original.

## Limitations

- If the reference file has different codec settings than the broken file, recovery may only produce 1–2 seconds of footage
- If the file was deleted from the SD card without being copied first, recovery is not possible
- Large file size does not guarantee footage exists — cameras pre-allocate storage, so a 2GB file may contain only seconds of actual data

## License

MIT
