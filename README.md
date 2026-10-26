# skeptical-no-stutter-video-motion (Grok skill)

Why ffmpeg zooms stutter (whole-pixel rounding), the static-stills rule for text cards, and linting the build script instead of pixel-diffing.

## Install

This repo is the skill folder: `SKILL.md` sits at the top level. Clone it into a skills directory.

User scope (all your projects), macOS or Linux:

    mkdir -p ~/.grok/skills
    git clone https://github.com/skeptical-agency/skeptical-no-stutter-video-motion.git ~/.grok/skills/skeptical-no-stutter-video-motion

User scope, Windows PowerShell (assumes Grok reads skills from `$HOME\.grok\skills`):

    New-Item -ItemType Directory -Force "$HOME\.grok\skills"
    git clone https://github.com/skeptical-agency/skeptical-no-stutter-video-motion.git "$HOME\.grok\skills\skeptical-no-stutter-video-motion"

Project scope (shared via git):

    mkdir -p .grok/skills
    git clone https://github.com/skeptical-agency/skeptical-no-stutter-video-motion.git .grok/skills/skeptical-no-stutter-video-motion

No git? Download the repo as a zip from GitHub and unzip it into a folder named `skeptical-no-stutter-video-motion` inside the skills directory.

More skills: [skeptical-agency/grok-skills](https://github.com/skeptical-agency/grok-skills).

## Requirements

- `ffmpeg` and `ffprobe` on your PATH (macOS: `brew install ffmpeg`; Linux: `sudo apt install ffmpeg`; Windows: `winget install Gyan.FFmpeg`)

## Use

Ask Grok things like:

> my Ken Burns zoom on stills looks steppy in ffmpeg, fix it
> lint this ffmpeg build script for motion that will stutter: <script>

SKILL.md has a plain-English intro, a worked example, the exact commands and the known pitfalls.

## License

MIT, see LICENSE.
