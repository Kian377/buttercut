---
name: installation-check
description: Verifies ffmpeg and WhisperX are installed. Use when setting up the video-agent environment or when encountering errors related to missing tools.
---

# Skill: Installation Check

## Purpose

Verifies that ffmpeg and WhisperX are installed before starting video editing work.

## When to Use This Skill

- When setting up the video-agent environment for the first time
- When encountering errors related to missing tools
- When the user reports issues with transcription or video processing

## Required Tools

1. **ffmpeg** - Video processing and frame extraction
2. **WhisperX** - Audio transcription with accurate timestamps

## Installation

### macOS (using Homebrew):

```bash
brew install ffmpeg
pip install whisperx
```

### Linux (using apt):

```bash
sudo apt-get update && sudo apt-get install ffmpeg
pip install whisperx
```

## Verification

Check both tools are installed:

```bash
# Check ffmpeg
ffmpeg -version

# Check WhisperX
whisperx --help
```

## Troubleshooting

**ffmpeg not found:**
- macOS: `brew install ffmpeg`
- Linux: `sudo apt-get install ffmpeg`

**WhisperX not found:**
- Install: `pip install whisperx`
- May require PyTorch: `pip install torch`
- On Apple Silicon, WhisperX can use MPS (Metal Performance Shaders) for GPU acceleration

## Notes

- Installation only needs to be done once per environment
- WhisperX has its own dependencies (PyTorch, etc.) that are installed automatically via pip
