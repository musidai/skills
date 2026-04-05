---
name: musidai
description: Use when the user wants to generate a music video, video scene, character image, or music track via the Musid AI platform API. Triggers on /musidai, /musidai full, /musidai image, /musidai video, /musidai music commands, or any request to start an AI content generation pipeline on Musid.
---

# Musid AI

Reference guide for generating AI content on the Musid platform. Two modes: full pipeline (end-to-end music video) or single asset (image / video / music).

## Mode A: Full Pipeline (`/musidai` or `/musidai full`)

### Step 1: Check API key

Verify `MUSID_API_KEY` is in the environment. If missing, tell the user:

> Your Musid AI API key is not configured. Please visit https://musid.ai/settings/apikeys to generate an API key, then run `/update-config` and set `MUSID_API_KEY=<your key>`.

Stop here if no API key.

### Step 2: Collect parameters

**Required:**
- `prompt` — creative direction / theme / visual style

**Common optional (ask if not provided):**
- `audioUrl` — direct URL to mp3/wav (skip if none)
- `character` — main character description (AI auto-generates if omitted)
- `aspectRatio` — default `16:9`; commonly used alternatives: `9:16`, `1:1`, `3:2`, `2:3`
- `visibility` — `public` (default) or `private`

**Audio timing (when audioUrl is provided):**
- `audioDuration` — clip duration in seconds
- `audioStartTime` — start offset in seconds
- `audioEndTime` — end offset in seconds

**Pipeline behavior:**
- `autoMode` — `true` (default); set `false` to pause for manual storyboard review
- `multiShots` — optional boolean; request default is `true`
- `agentModel` — default `wan-2.7`; optional alternative `grok-imagine`

**Advanced (only if explicitly asked):**
- `resolution` — `720p` (default) or `1080p`
- `characterImageUrls` — array of reference image URLs (max 14)

If all params are provided upfront, proceed without asking.

### Step 3: Call the pipeline API

```typescript
const response = await fetch(`${process.env.NEXT_PUBLIC_APP_URL || 'https://musid.ai'}/api/agent/pipeline/start`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${process.env.MUSID_API_KEY}`,
  },
  body: JSON.stringify({
    prompt,
    audioUrl,           // optional
    audioDuration,      // optional, seconds
    audioStartTime,     // optional, seconds
    audioEndTime,       // optional, seconds
    agentModel,         // optional, default 'wan-2.7'
    character,          // optional
    aspectRatio,        // default '16:9'
    resolution,         // optional, '720p' or '1080p'
    visibility,         // optional, 'public' or 'private'
    autoMode,           // optional, default true
    multiShots,         // optional, default true when audio present
    characterImageUrls, // optional array
  }),
});
const data = await response.json();
```

Or via curl:
```bash
curl -X POST "${APP_URL}/api/agent/pipeline/start" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${MUSID_API_KEY}" \
  -d '{"prompt": "...", "audioUrl": "...", "aspectRatio": "16:9"}'
```

### Step 4: Show result

On success (`data.data.projectId` exists):
```
Music video generation started!

Project: {data.data.projectUrl}
Project ID: {data.data.projectId}

The pipeline is running in the background:
  - Character design & audio analysis (parallel)
  - Wait for "Audio analysis complete" when audio is present
  - Scene planning via the planScenes tool
  - Storyboard image generation
  - Scene video generation with the selected agent model

When complete, visit the project URL to preview scenes and click Merge to assemble the final video.
```

On error:
```
Failed to start pipeline: {data.message}
```

---

## Mode B: Single Asset (`/musidai image` | `/musidai video` | `/musidai music`)

> **Auth:** Accepts `Authorization: Bearer <MUSID_API_KEY>` (same key as the pipeline) or a web session cookie. `provider` and `model` are optional; the server selects defaults automatically.

Base URL: `process.env.MUSID_APP_URL || 'https://musid.ai'`

All single-asset requests go to `POST /api/ai/generate`.

---

### Image

**Scenes:** `text-to-image` (default) or `image-to-image`

**Parameters:**
| Field | Default | Notes |
|-------|---------|-------|
| `mediaType` | `"image"` | fixed |
| `scene` | `"text-to-image"` | or `"image-to-image"` |
| `prompt` | required | |
| `visibility` | `"public"` | or `"private"` |
| `options.aspect_ratio` | `"auto"` | e.g. `"16:9"`, `"9:16"`, `"1:1"` |
| `options.resolution` | `"1K"` | `"1K"` (4 cr) / `"2K"` (8 cr) / `"4K"` (12 cr) |
| `options.output_format` | `"jpg"` | or `"png"` |
| `options.google_search` | `false` | AI web grounding toggle |
| `options.image_input` | — | string[] of reference image URLs; `image-to-image` only |

```bash
# Text to image
curl -X POST "${APP_URL}/api/ai/generate" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${MUSID_API_KEY}" \
  -d '{
    "mediaType": "image",
    "scene": "text-to-image",
    "prompt": "...",
    "visibility": "public",
    "options": {
      "aspect_ratio": "16:9",
      "resolution": "1K",
      "output_format": "jpg"
    }
  }'

# Image to image
curl -X POST "${APP_URL}/api/ai/generate" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${MUSID_API_KEY}" \
  -d '{
    "mediaType": "image",
    "scene": "image-to-image",
    "prompt": "...",
    "options": {
      "image_input": ["https://example.com/ref.jpg"]
    }
  }'
```

---

### Video

**Scenes:** `text-to-video` (default) or `image-to-video`

**Model selection — always ask the user whether they need lip-sync:**

| Need | Recommended model | Resolution |
|------|------------------|------------|
| Lip-sync or full pipeline default | `wan-2.7` | `480p` / `720p` / `1080p` |
| No lip-sync / lowest cost | `grok-imagine` | typically `720p` |

Pass the model name in the `model` field. The API accepts `wan-2.7` and `grok-imagine` directly. Legacy aliases `musid-lite` and `musid-pro` still resolve to Wan 2.7, but they should not be recommended in new docs.
If `provider` and `model` are both omitted, the server picks the default provider first, then applies that provider's default model. This is config-dependent, so do not assume a specific video model unless you set `model` explicitly.

**Universal parameters:**
| Field | Default | Notes |
|-------|---------|-------|
| `mediaType` | `"video"` | fixed |
| `scene` | `"text-to-video"` | or `"image-to-video"` |
| `prompt` | required | |
| `visibility` | `"public"` | or `"private"` |
| `model` | provider default | recommend setting to `"wan-2.7"` or `"grok-imagine"` explicitly |
| `options.aspect_ratio` | `"9:16"` | Wan 2.7: `"16:9"` / `"9:16"` / `"1:1"` / `"4:3"` / `"3:4"` for T2V; Grok commonly uses `"2:3"` / `"3:2"` / `"1:1"` / `"9:16"` / `"16:9"` |
| `options.duration` | model-dependent | Wan 2.7 commonly defaults to `5`; Wan 2.7 supports `2-15` seconds (integer); Grok Imagine scenes are typically `6-30` seconds |

**Wan 2.7 fields:**
| Field | Default | Notes |
|-------|---------|-------|
| `options.resolution` | `"480p"` | `"480p"` / `"720p"` / `"1080p"` |
| `options.seed` | — | optional integer |
| `options.negative_prompt` | `""` | negative prompt |
| `options.enable_prompt_expansion` | `true` | AI prompt enhancement |
| `options.multi_shots` | — | boolean; requires `enable_prompt_expansion: true` |
| `options.audio` | — | URL to audio file |
| `options.image` / `options.first_frame` | — | reference image URL; image-to-video only. `options.image` is the compatibility-friendly field in this API; the provider layer maps it to `first_frame` for Wan 2.7 i2v |

**Grok Imagine fields:**
| Field | Default | Notes |
|-------|---------|-------|
| `options.mode` | `"normal"` | `"fun"` / `"normal"` / `"spicy"` |

**Wan 2.7 prompt rules:**
- Use structured multi-shot prompts with explicit time markers when needed, for example `[0-3s]`, `[3-8s]`.
- Put each timed segment on its own line.
- Do not plan scenes shorter than 5 seconds in the full music video agent flow unless the user explicitly needs single-asset clips outside the pipeline.
- If audio is supplied, Wan 2.7 is the recommended model because the agent pipeline is designed around its lip-sync support.

```bash
# Text to video with Wan 2.7
curl -X POST "${APP_URL}/api/ai/generate" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${MUSID_API_KEY}" \
  -d '{
    "mediaType": "video",
    "model": "wan-2.7",
    "scene": "text-to-video",
    "prompt": "Rain-soaked rooftop singer, cinematic blue backlight.\n\n[0-3s] Medium push-in as the singer turns toward camera and starts the chorus.\n\n[3-5s] Tight close-up with clear mouth movement, wind in hair, emotional delivery.",
    "options": {
      "aspect_ratio": "9:16",
      "duration": 5,
      "resolution": "720p",
      "enable_prompt_expansion": true
    }
  }'

# Image to video with Grok Imagine
curl -X POST "${APP_URL}/api/ai/generate" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${MUSID_API_KEY}" \
  -d '{
    "mediaType": "video",
    "model": "grok-imagine",
    "scene": "image-to-video",
    "prompt": "...",
    "options": {
      "image": "https://example.com/ref.jpg",
      "aspect_ratio": "9:16",
      "mode": "normal"
    }
  }'
```

---

### Music

**Parameters:**
| Field | Default | Notes |
|-------|---------|-------|
| `mediaType` | `"music"` | fixed |
| `prompt` | required | song description / direction |
| `visibility` | `"public"` | or `"private"` |
| `options.style` | `""` | genre / style tags |
| `options.title` | `""` | song title |
| `options.customMode` | `false` | enable custom lyrics mode |
| `options.instrumental` | `false` | no vocals |
| `options.vocalGender` | `""` | `"m"` or `"f"` |
| `options.negativeTags` | `""` | style tags to avoid |
| `options.lyrics` | `""` | custom lyrics; only when `customMode: true` and no audio uploaded |
| `options.scene` | — | `"add-vocals"` or `"add-instrumental"`; only when audio uploaded. For music requests, this switch lives inside `options`, not the top-level `scene` field |
| `options.uploadUrl` | — | uploaded audio file URL; only when audio uploaded |

```bash
# Generate music
curl -X POST "${APP_URL}/api/ai/generate" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${MUSID_API_KEY}" \
  -d '{
    "mediaType": "music",
    "prompt": "upbeat pop song about summer",
    "options": {
      "style": "pop, upbeat",
      "title": "Summer Vibes"
    }
  }'

# Add vocals to uploaded audio
curl -X POST "${APP_URL}/api/ai/generate" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${MUSID_API_KEY}" \
  -d '{
    "mediaType": "music",
    "prompt": "soulful R&B vocals",
    "options": {
      "scene": "add-vocals",
      "uploadUrl": "https://example.com/track.mp3"
    }
  }'
```

---

### Single-asset success response
```
{type} generation started! Task ID: {taskId}

View result at: {APP_URL}/my-creations

The webhook will update task status automatically when complete.
```

---

## Notes

- Full pipeline defaults to `wan-2.7` unless `agentModel` is provided explicitly.
- Pipeline scene planning is tool-driven: the agent must wait for `Audio analysis complete` before calling `planScenes`.
- Video merge (final assembly) is done **on the web** — click Merge on the project page.
- Pipeline projects show a CHARACTER step in the UI with the generated reference image.
- Credit costs: Image 1K=4cr, 2K=8cr, 4K=12cr · Music=4cr · Wan 2.7 video 6cr/sec (`480p`/`720p`) or 9cr/sec (`1080p`) · Grok Imagine video 1cr/sec · Audio transcription=1cr · Vocal remover=4cr
