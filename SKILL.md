---
name: skeptical-no-stutter-video-motion
description: "Kill stuttering Ken Burns/zoom motion in ffmpeg builds; static-stills rule and source linting. Use when slow zooms or pans on stills look steppy, when QA-ing a video build pipeline for motion defects, or when choosing motion versus static treatment for text cards."
when-to-use: "Slow zoom/pan (Ken Burns) on stills looks steppy or jittery in the final render, you are QA-ing a video build pipeline for motion defects, or you are choosing between motion and static treatment for text cards."
---

# No-stutter motion in ffmpeg video builds

## What this does (plain English)

If a slow zoom on a still image looks jittery in your ffmpeg video, it is not your settings. ffmpeg rounds crop and zoompan positions to whole pixels, so slow motion of half a pixel per frame renders as steps (0,0,1,0,1) that you can see. This skill gives your AI assistant the production-proven fix hierarchy: text-bearing stills get no motion at all, hook cards stay static, and smooth motion is saved for plain b-roll. It also teaches a better way to catch the problem: lint the build script for banned patterns instead of trying to detect stutter in pixels.

## Install what you need

You need `ffmpeg` and `ffprobe`.

macOS:

```
brew install ffmpeg
```

Linux (Debian/Ubuntu shown; on Fedora use `sudo dnf install ffmpeg`, which may need the RPM Fusion repo enabled):

```
sudo apt install ffmpeg
```

Windows (PowerShell or Command Prompt; open a NEW terminal window after installing so the tools show up on PATH):

```
winget install Gyan.FFmpeg
```

Check it worked: `ffmpeg -version` and `ffprobe -version` should both print a version.

## Worked example

A decision walk-through using only the rules below:

1. You have a still with a baked headline that you were going to push in slowly. Per the fix hierarchy, it gets no motion at all: render it as a pure static frame with `-loop 1 -t <dur>` and zero push-in, pan or zoom.
2. The still is a hook or thumbnail card. It must be static anyway, so every possible thumbnail frame is composed.
3. You want movement. Use it only on pure b-roll or source footage with no baked text, or cut to a new shot. Visual density comes from cuts, not drift.
4. QA the build by linting the source. Grep the build script for the banned patterns: any `zoompan`; time-driven crop or overlay expressions (crop x/y or overlay x/y containing `t`, `mod(` or `if(gt(t`); per-frame PIL zoom piped as rawvideo into ffmpeg; motion filters applied to `-loop` still inputs. Exit nonzero on any match, with no overrides.
5. Check the video stream duration, not the format duration, when a still lacks `-loop 1`.

---

The full production recipe follows.


## When to Use
- Slow zoom/pan (Ken Burns) on stills looks steppy/jittery in the final render
- You are QA-ing a video build pipeline for motion defects
- Choosing between motion and static treatment for text cards

## Root cause
ffmpeg crop/zoompan positions are rounded to WHOLE PIXELS. Slow motion
(0.5px/frame) renders as steps: 0,0,1,0,1... visible stutter. Naive
scale-1.06x + float crop expressions AND zoompan at 1x scale both stutter.

## The fix hierarchy (production-proven)

1. TEXT-BEARING STILLS GET NO MOTION AT ALL (the real fix). Any still with
   baked headline text (cards, press images with overlay text) renders as a
   pure static frame: `-loop 1 -t <dur>`, zero push-in/pan/zoom. Motion belongs
   ONLY on pure B-roll/source footage with no baked text. Moving text behind a
   zoom reads as cheap and annoys viewers.
2. HOOK/THUMBNAIL CARDS MUST BE STATIC anyway: platforms grab arbitrary frames
   for thumbnails; a push-in means the grab lands off-center. Static = every
   possible thumbnail frame is perfectly composed.
3. If you truly need smooth motion on footage: a 4x-supersample zoompan
   (render at 4x size, scale down) smooths the rounding steps, but it is
   expensive and usually unnecessary given rules 1-2. Prefer cutting to a new
   shot over moving a still: visual density comes from CUTS, not drift.

## QA: lint the SOURCE, not the pixels
Pixel-level stutter detection is UNRELIABLE: frame-diff thresholds false-fail
approved smooth push-ins AND frozen-then-jump stutters both read as "frozen"
at any downscale that fits in memory. No threshold separated approved builds
from real stutter in production testing.

Instead, GREP THE BUILD SCRIPT for banned patterns (deterministic, zero false
positives, catches the bug at the line that causes it):
- any zoompan usage (ban outright)
- time-driven crop/overlay expressions (crop x/y or overlay x/y containing 't'
  or 'mod(' or 'if(gt(t')
- per-frame PIL zoom piped as rawvideo into ffmpeg
- motion filters applied to -loop still inputs
Exit nonzero on any match; no overrides.

## Related traps
- zoompan on a LOOPED image input multiplies frames; feed a single still and
  let zoompan's d= produce the frame count (`-i img.jpg without -loop`; output
  length = d/fps).
- A still WITHOUT -loop 1 emits exactly ONE frame regardless of -t; the concat
  comes out silently short (ffprobe FORMAT duration still reads the audio
  length - check the VIDEO STREAM duration).
- First-10-seconds rule for retention: nothing holds >2s in the opening window;
  pack 4+ distinct shots into the first 8s and assert it in code.
