# Video Recovery Skill

## Purpose
Recover corrupted MP4 video files caused by premature shutdown (powered off mid-recording). Works for DJI drones, GoPro, Sony, and any camera that produces truncated MP4s with a missing or broken moov atom.

## When to trigger
Use this skill when a user:
- Has a video file that won't open or play
- Mentions they turned off their camera/drone before stopping recording
- Reports "moov atom not found" or "invalid data" errors
- Has a large file (hundreds of MB to several GB) that appears corrupted

---

## Step 1: Gather information upfront

Before doing anything, ask the user these questions in a single message:

1. **What camera/drone recorded the file?** (DJI Mini 3, GoPro Hero 12, Sony ZV-E10, etc.)
2. **What is the full file path of the broken video?**
3. **Do you have another working video file from the same camera, ideally recorded at the same settings (resolution, FPS)?** If yes, get the full path. This is the reference file and is required for the best recovery method.
4. **Is the file still on the SD card, or already copied to your computer?** (If still on SD card, warn them not to write anything else to it.)
5. **What OS are you on?** (Mac or Windows — this skill covers Mac via Homebrew; Windows users need WSL or manual installs)

If the user provides all this upfront, skip straight to Step 2.

---

## Step 2: Diagnose the file

Run FFmpeg to confirm the corruption type:

```bash
ffmpeg -err_detect ignore_err -i [BROKEN_FILE_PATH] -c copy [OUTPUT_PATH]
```

Where `[OUTPUT_PATH]` is the broken file's directory + filename + `_recovered.MP4`.

**Interpret the output:**
- `moov atom not found` → classic premature shutdown. Proceed to Step 3.
- File recovers successfully → done, share the output path with the user.
- Other errors → attempt Step 3 anyway, it handles most cases.

---

## Step 3: Check for FFmpeg

```bash
ffmpeg -version
```

If not installed:
```bash
brew install ffmpeg
```

---

## Step 4: Attempt recovery with untrunc

untrunc is the most effective tool for moov atom corruption. It requires a healthy reference file from the same camera with the same codec settings.

### 4a. Check if untrunc is installed

```bash
which untrunc || which untrunc-anthwlock
```

### 4b. If not installed, build from source

```bash
cd ~
brew install git ffmpeg
git clone https://github.com/anthwlock/untrunc.git
cd untrunc
make CXXFLAGS="-isystem$(brew --prefix ffmpeg)/include" LDFLAGS="-L$(brew --prefix ffmpeg)/lib -lavformat -lavcodec -lavutil"
```

If `make` fails with linker errors, find the exact ffmpeg version and path:
```bash
ls /opt/homebrew/Cellar/ffmpeg/
```
Then substitute the exact version into the paths above.

The binary will be at `~/untrunc/untrunc`.

### 4c. Run untrunc

```bash
~/untrunc/untrunc [REFERENCE_FILE_PATH] [BROKEN_FILE_PATH]
```

The output file is saved next to the broken file, named `[original]_fixed-dyn.MP4` or similar.

### 4d. If first attempt only recovers 1–2 seconds, try with `-s` flag

```bash
~/untrunc/untrunc -s [REFERENCE_FILE_PATH] [BROKEN_FILE_PATH]
```

---

## Step 5: Evaluate results and handle failures

**If recovery succeeds:** Tell the user the output file path and approximate duration recovered.

**If only 1–2 seconds recovered with 99%+ bytes unmatched:**
- The reference file likely has different codec parameters (different FPS, bitrate, color profile, or recording mode)
- Ask: "Do you have another clip from the same session, same exact settings?" Try a different reference file.
- If no other reference exists, the footage is likely unrecoverable at the file level.

**If the file was deleted from the SD card:**
- Inform the user that without the raw SD card data, recovery is not possible at this point.
- If the SD card still has the file, recommend stopping all writes to it and using a block-level tool like **PhotoRec** (`brew install testdisk`) as a last resort.

---

## Step 6: Cleanup (optional)

If untrunc was built from source and the user wants to remove it:
```bash
rm -rf ~/untrunc
```

---

## Notes

- The file size of a corrupted video is not indicative of how much footage was actually written. Cameras pre-allocate storage, so a 2.4GB file may only contain 1 second of actual footage.
- untrunc works by reading codec parameters from the healthy reference file and applying them to reconstruct the broken file's container. If the reference file has different settings, recovery will fail or be partial.
- This skill covers Mac (Apple Silicon and Intel) via Homebrew. Windows users should use WSL2 and follow the same steps inside it.
- Warnings during the untrunc build (C++17 extensions) are harmless and can be ignored.
