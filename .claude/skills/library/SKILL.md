---
name: library
description: Works with Libraries, the primary abstraction. Sets up a new library or works with an existing library. Creates directory structure (/transcripts, /roughcuts) and initializes library.yaml for footage analysis. Use anytime a user mentions a library or wants to work with a library. Used in virtually every context.
---

# Skill: Setup Library

## Purpose

This skill guides you through setting up a new library or working on an existing library.

A library is conceptually similar to a Final Cut Pro library - it's a container that can hold multiple rough cuts (equivalent to FCP projects). However, it uses a simple file structure (markdown, YAML) optimized for AI analysis and transcription, rather than FCP's proprietary `.fcpbundle` format.

A library is focused on footage analysis and transcription, not editorial planning. Editorial decisions (purpose, tone, audience, etc.) are made during rough cut creation.

## When to Use This Skill

- When the user provides video files for a new project
- When the user mentions starting a new video series
- Before beginning transcription or rough cut work
- When you need to understand project requirements

## Step 1: Check for Existing Library

**ALWAYS** check if a library already exists before asking setup questions:

```bash
# Check if library directory and library.yaml exist
ls libraries/[library-name]/library.yaml
```

**If library.yaml exists:**
- Skip setup entirely - the library is already configured
- Read the existing library.yaml to understand project status
- User is returning to existing work

**If library directory exists but library.yaml is missing:**
- Check what files are present (/transcripts/, /roughcuts/, etc.)
- If files exist, inform user of the current state
- Proceed with creating/recreating library.yaml (Step 2) to restore consistency

**If no library directory exists:**
- Proceed to Step 2 to gather project information and create new library

## Step 2: Gather Project Information

Ask the user these questions:

1. **What is the library name for this project?**
   - Examples: "bike-locking-video-series", "raiders-2025-highlights", "yo-yo-techniques"
   - Take whatever the user provides and normalize it:
     - Replace spaces with dashes
     - Convert to lowercase
     - Remove special characters (keep alphanumeric and dashes)

2. **Where are the video files located?**
   - Accept either:
     - A directory path (will recursively find all video files including subdirectories)
     - Individual file paths
   - Verify all files exist before proceeding
   - Inform user of what was found: "Found 5 video files totaling 2.3GB"

3. **What language is spoken in these videos?**
   - Common options: `en` (English), `es` (Spanish), `fr` (French), `de` (German), `ja` (Japanese)
   - Or `auto` for auto-detect (adds ~30 seconds per video during transcription)
   - This applies to all videos in this library
   - Save response to library.yaml for use during transcription

## Step 3: Create Directory Structure

Create the library directory structure:

```bash
mkdir -p libraries/[library-name]
mkdir -p libraries/[library-name]/transcripts
mkdir -p libraries/[library-name]/roughcuts
```

Note: A single `/tmp/` directory at the root of video-agent is used for all temporary files. You can create new directories inside there when you need to create temp files. Those directories or files can be deleted after they've been used.

## Step 4: Create Library File

Duplicate the template from `templates/library_template.yaml` to create `library.yaml`:

For each video file given from the user or located inside the directory:
1. Use `ffprobe` to get duration and file size
2. Add entry to library.yaml with empty transcript_path and visual_transcript_path
3. Paths remain empty until transcription is complete - empty means todo, a valid path means done

The `language` field stores the language code for all videos in this library.

Progressively update the `footage_summary` field after each video is transcribed with 1-3 sentences covering subjects, locations, activities, visual style, etc.

## Step 5: Start Footage Analysis

After library setup completes, **automatically start analyzing all footage**:

1. Inform the user: "Library setup complete. Found [N] videos ([total size]). Starting footage analysis..."
2. Read library.yaml to get the language code and find videos needing transcription
3. Launch `transcribe-audio` agents (can run in parallel for multiple videos)
4. As each transcribe-audio agent completes, update library.yaml with `transcript_path`
5. After all audio transcripts complete, launch `analyze-video` agents (can run in parallel)
6. As each analyze-video agent completes, update library.yaml with `visual_transcript_path`
7. Analyze ALL videos before offering to create rough cuts

**Parallel execution pattern:**
- Launch multiple agents in parallel (one per video) to maximize throughput
- Each agent returns: video_path and transcript_path (or visual_transcript_path)
- Parent thread updates YAML sequentially as agents complete
- This prevents race conditions while maximizing parallelism

**Important terminology:**
- User-facing: Call it "footage analysis" or "analyzing footage"
- Internal/file names: Use "transcription" (library.yaml, transcript_path, etc.)

**If user requests rough cut before analysis completes:**
- Warn: "I can create a rough cut now, but I'll do a better job after analyzing all the footage. Continue anyway?"
- If user confirms, proceed with rough cut creation
- Otherwise, wait for analysis to complete

## Key Reminders

- Always check for existing library before asking setup questions
- Normalize library names: replace spaces with dashes, remove special characters
- Each library is self-contained with all its files and outputs
- The library.yaml is your single source of truth - it contains all metadata, footage descriptions, and transcription status
- Update library.yaml with footage_summary as you work
- Libraries are containers for footage analysis/transcription, editorial planning should take place in roughcuts
- Automatically start footage analysis after setup - analyze ALL videos before offering rough cuts
