---
title: Quick Start
description: Install the Musid AI skill for your AI coding agent and start generating music videos, images, and music with natural language.
---

Generate music videos, images, and music tracks directly from your AI coding agent — no code required.

**Skill:** [github.com/musidai/skills](https://github.com/musidai/skills)

---

## 1. Get your API key

Sign in at [musid.ai](https://musid.ai), then go to [Settings → API Keys](https://musid.ai/settings/apikeys) and create a key.

---

## 2. Install the skill

Pick your agent tool:

### Claude Code

```bash
mkdir -p ~/.claude/skills/musidai
curl -fsSL https://raw.githubusercontent.com/musidai/skills/main/SKILL.md \
  -o ~/.claude/skills/musidai/SKILL.md
```

Then set your API key:

```bash
claude config set MUSID_API_KEY your-key-here
```

Use it: type `/musidai` in any Claude Code session.

---

### Cursor

Create `.cursor/rules/musidai.mdc` in your project:

```bash
curl -fsSL https://raw.githubusercontent.com/musidai/skills/main/SKILL.md \
  -o .cursor/rules/musidai.mdc
```

Then add to your `.env` or Cursor environment settings:

```
MUSID_API_KEY=your-key-here
```

Cursor will automatically apply the rule when you ask about music video generation.

---

### Codex

**Global** (applies to all projects):

```bash
mkdir -p ~/.codex
curl -fsSL https://raw.githubusercontent.com/musidai/skills/main/SKILL.md \
  >> ~/.codex/instructions.md
```

**Project-level** (add to your repo's `AGENTS.md`):

```bash
curl -fsSL https://raw.githubusercontent.com/musidai/skills/main/SKILL.md >> AGENTS.md
```

Set the key in your environment:

```bash
export MUSID_API_KEY=your-key-here
```

---

### Gemini CLI

```bash
curl -fsSL https://raw.githubusercontent.com/musidai/skills/main/SKILL.md >> GEMINI.md
```

```bash
export MUSID_API_KEY=your-key-here
```

---

### OpenCode

```bash
mkdir -p ~/.opencode/skills/musidai
curl -fsSL https://raw.githubusercontent.com/musidai/skills/main/SKILL.md \
  -o ~/.opencode/skills/musidai/SKILL.md
```

---

### Cline / OpenClaw (VS Code)

Open the Cline settings panel → **Custom Instructions**, paste the skill content:

```bash
curl -fsSL https://raw.githubusercontent.com/musidai/skills/main/SKILL.md
```

Then add `MUSID_API_KEY=your-key-here` to your VS Code environment or `.env`.

---

## 3. Use it

Just describe what you want in natural language. Your agent handles the API calls.

The full music video pipeline now defaults to **Wan 2.7**. If you provide audio, the agent will first wait for audio analysis to finish, then plan scenes, generate storyboard frames, and render scene videos.

```
Generate a music video: neon cyberpunk city at night, vertical format
```

```
Create a music video with my track: https://example.com/song.mp3
```

```
Create a lip-synced music video in Wan 2.7: moody rain-soaked rooftop performance, 16:9, cinematic lighting
```

```
Create a lower-cost non-lipsync video with Grok Imagine: surreal desert dance sequence, vertical format
```

```
Generate an image: album cover, bold geometric shapes, 1:1 square
```

```
Generate a music track: upbeat pop song about summer, no vocals
```

---

## What it generates

| Command | Output |
|---------|--------|
| Music video (full) | Audio analysis + character design → scene planning → storyboard images → Wan 2.7 or Grok scene videos |
| Image | Single AI-generated image |
| Video clip | Single video scene with Wan 2.7 or Grok Imagine |
| Music track | Original AI-generated song |

## Current model behavior

- **Wan 2.7** is the default full-pipeline model and the recommended choice when you need lip-sync.
- **Wan 2.7** supports `2-15s` scene durations and can use multi-shot prompts with time markers like `[0-3s]`, `[3-8s]`.
- **Grok Imagine** is the lower-cost alternative for non-lip-sync scenes.
- Final merge still happens on the project page after all scene clips finish rendering.

View all results at [musid.ai/my-creations](https://musid.ai/my-creations).
