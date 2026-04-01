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

```
Generate a music video: neon cyberpunk city at night, vertical format
```

```
Create a music video with my track: https://example.com/song.mp3
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
| Music video (full) | Character design → scene planning → storyboard → video clips |
| Image | Single AI-generated image |
| Video clip | Single video scene |
| Music track | Original AI-generated song |

View all results at [musid.ai/my-creations](https://musid.ai/my-creations).
