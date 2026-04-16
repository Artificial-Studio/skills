---
name: artificial-studio
description: >
  Use this skill when the user wants to generate images, videos, audio, 3D models, or text using AI via the Artificial Studio API.
  TRIGGER when: the user asks to generate media with Artificial Studio, integrate the Artificial Studio API, or use any Artificial Studio endpoint.
license: MIT
metadata:
  author: artificialstudio
  version: "1.0"
---

# Artificial Studio API

Artificial Studio provides a single REST API to generate images, videos, audio, 3D models, and text using state-of-the-art AI models.

**Base URL:** `https://api.artificialstudio.ai`

## Authentication

All requests require an API key in the `Authorization` header. **Do not** use the `Bearer` prefix.

```
Authorization: YOUR_API_KEY
```

**How to get the API key:**

1. First, check the environment variable `ARTIFICIAL_STUDIO_API_KEY`. If it's set, use it automatically.
2. If the environment variable is not set, ask the user for their API key before making any request.
3. Tell the user they can get a key at https://artificialstudio.ai/settings/api-keys
4. Suggest they set it permanently with: `export ARTIFICIAL_STUDIO_API_KEY=your_key_here` (add to `.bashrc`/`.zshrc` for persistence).

Never hardcode the key in source code.

## How It Works

Every generation follows four steps:

1. **Discover** — `GET /api/search?q=...` to find the right tool and model. The response includes the full input schema so you know exactly what fields to send.
2. **Create** — `POST /api/run` with the tool, model, and input. Returns a generation ID immediately.
3. **Poll or Webhook** — Either poll `GET /api/generations/:id` until `status` is `success`, or pass a `webhook` URL to get notified.
4. **Get output** — The completed generation includes an `output` URL pointing to the generated media.

## Endpoints

### POST /api/run — Create a Generation

```json
{
  "tool": "create-image",
  "input": {
    "prompt": "A serene mountain landscape at sunset"
  },
  "webhook": "https://your-server.com/webhook"  // optional
}
```

**Headers:**
- `Authorization` (required) — API key
- `Content-Type` (required) — `application/json`

**Body:**
- `tool` (string, required) — Tool identifier. Use `GET /api/tools` to list available tools.
- `input` (object, required) — Tool-specific parameters.
- `input.model` (string, optional) — Model to use. Defaults to the tool's primary model. Use `GET /api/tools/:slug` to see available models.
- `input.prompt` (string, varies) — Text prompt.
- `webhook` (string, optional) — HTTPS URL to receive a POST when the generation completes.

The body is validated strictly — unknown fields return `400`.

**Response (202):**
```json
{
  "id": "507f1f77bcf86cd799439011",
  "status": "processing",
  "tool": "create-image",
  "createdAt": "2024-01-15T10:30:00.000Z"
}
```

### GET /api/generations/:id — Get a Generation

**Response (200):**
```json
{
  "id": "507f1f77bcf86cd799439011",
  "status": "success",
  "tool": "create-image",
  "output": "https://files.artificialstudio.ai/generations/abc123.png",
  "thumbnail": "https://files.artificialstudio.ai/thumbnails/abc123.jpg",
  "error": null,
  "type": "image",
  "payload": { "model": "nano-banana", "prompt": "..." },
  "createdAt": "2024-01-15T10:30:00.000Z"
}
```

**Status values:** `pending` → `processing` → `uploading` → `success` | `error`

### GET /api/generations — List Generations

**Query params:** `limit` (default 20, max 100), `offset` (default 0), `status` (optional filter).

**Response:** `{ data: [...], pagination: { total, limit, offset, hasMore } }`

### GET /api/search?q=... — Search Tools and Models

Search for the right tool and model by describing what you want to do. Returns matching tools with their models and full input schemas.

```bash
curl "https://api.artificialstudio.ai/api/search?q=upscale+video" \
  -H "Authorization: YOUR_API_KEY"
```

**Response:**
```json
{
  "data": [
    {
      "slug": "upscale-video",
      "name": "Upscale Video",
      "description": "Increase video resolution with AI upscaling",
      "type": "video",
      "outputType": "video",
      "models": [
        {
          "slug": "bytedance-upscaler",
          "name": "Bytedance Upscaler",
          "inputSchema": {
            "video_url": { "type": "string", "required": true, "label": "Video", "description": "URL of the video to upscale" }
          }
        }
      ]
    }
  ]
}
```

**IMPORTANT: Always use this endpoint first** to discover the right tool and model for the user's request. The response includes the full `inputSchema` for each model, telling you exactly which fields to send.

### GET /api/tools — List Available Tools

Returns all available tools with `slug`, `name`, `description`, `type`, `outputType`.

### GET /api/tools/:slug — Get Tool Details

Returns tool info including available `models` with their `slug`, `name`, and `inputSchema` (full field definitions with types, required/optional, defaults, and allowed values).

### GET /api/account — Get Account Info

Returns `id`, `email`, `credits`, `plan` (`free` or `pro`), `createdAt`.

### POST /files — Upload a File

Upload a local file to get a URL you can use in generation inputs.

```bash
curl -X POST https://api.artificialstudio.ai/files \
  -H "Authorization: YOUR_API_KEY" \
  -F "file=@/path/to/image.jpg"
```

**Response:**
```json
{
  "id": "file123",
  "url": "https://cdn.artificialstudio.ai/...",
  "type": "image",
  "createdAt": "2025-01-01T00:00:00.000Z"
}
```

## Available Tools

### Image Generation & Editing

| Tool Slug | Description | Requires Image |
|-----------|-------------|----------------|
| `create-image` | Generate images from text prompts | No |
| `edit-image` | Edit images using text prompts | Yes |
| `extend-image` | Extend/reframe images to different aspect ratios | Yes |
| `upscale-image` | Increase image resolution | Yes |
| `image-background-remover` | Remove background from images | Yes |
| `image-depth-map-generator` | Create depth map from image | Yes |
| `image-to-svg` | Convert images to SVG | Yes |
| `svg-creator` | Create SVG graphics from text | No |
| `style-transfer` | Transfer style between images | Yes |
| `try-clothes` | Virtual try-on with garment + person photos | Yes |
| `image-canvas` | Transform sketches into images | Yes |

### Video Generation & Editing

| Tool Slug | Description | Requires Media |
|-----------|-------------|----------------|
| `create-video` | Generate videos from text prompts | No |
| `animate-image` | Create video from a static image | Yes (image) |
| `edit-video` | Edit videos using text prompts | Yes (video) |
| `extend-video` | Extend video duration | Yes (video) |
| `upscale-video` | Increase video resolution | Yes (video) |
| `video-background-remover` | Remove video background | Yes (video) |
| `translate-video` | Translate video to other languages | Yes (video) |
| `video-sync-audio` | Generate synchronized audio for video | Yes (video) |
| `motion-transfer` | Transfer motion from reference video to image | Yes (both) |
| `character-replace` | Replace characters in videos | Yes (both) |

### Audio Generation & Processing

| Tool Slug | Description | Requires Audio |
|-----------|-------------|----------------|
| `text-to-speech` | Convert text to speech | No |
| `create-music` | Generate music from text | No |
| `sound-effects` | Create sound effects from text | No |
| `drum-generator` | Generate drum beats | No |
| `song-splitter` | Separate instruments from songs | Yes |
| `translate-audio` | Translate audio to other languages | Yes |
| `voice-isolator` | Remove background noise, isolate voice | Yes |

### 3D Object Generation

| Tool Slug | Description | Requires Image |
|-----------|-------------|----------------|
| `text-to-3d` | Create 3D model from text | No |
| `image-to-3d-object` | Create 3D model from image | Yes |
| `motion-3d` | Generate 3D human motions from text | No |

### Text Extraction

| Tool Slug | Description | Requires File |
|-----------|-------------|---------------|
| `audio-to-text` | Transcribe audio to text | Yes |
| `audio-to-subtitles` | Generate subtitles from audio | Yes |
| `image-to-text` | Describe images with AI | Yes |
| `text-extractor` | OCR - extract text from images/docs | Yes |

Use `GET /api/tools` to see the full up-to-date list with all models.

## Input Schema Patterns

### Text-only tools (create-image, create-video, text-to-speech, etc.)

```json
{
  "tool": "create-image",
  "input": {
    "model": "seedream-4",
    "prompt": "A futuristic city at sunset",
    "image_size": "landscape_16_9"
  }
}
```

Common optional fields:
- `image_size`: `"square_hd"`, `"square"`, `"portrait_4_3"`, `"portrait_16_9"`, `"landscape_4_3"`, `"landscape_16_9"`
- `duration`: `"5"`, `"10"` (for video tools)
- `voice`: model-specific voice options (for TTS)

### Image/file input tools (edit-image, animate-image, upscale-image, etc.)

```json
{
  "tool": "edit-image",
  "input": {
    "model": "flux-kontext",
    "prompt": "Change the background to a beach",
    "image_urls": ["https://example.com/photo.jpg"]
  }
}
```

Files must be provided as publicly accessible URLs in `image_urls`, `video_url`, or `audio_url`. Upload local files first with `POST /files`.

## Webhooks

Add a `webhook` URL to `POST /api/run` to receive results via POST instead of polling.

Requirements: must be HTTPS, publicly accessible, return 200 within 5 seconds.

**Webhook payload:**
```json
{
  "id": "507f1f77bcf86cd799439011",
  "status": "success",
  "tool": "create-image",
  "type": "image",
  "output": "https://files.artificialstudio.ai/generations/abc123.png",
  "thumbnail": "https://files.artificialstudio.ai/thumbnails/abc123.jpg",
  "createdAt": "2024-01-15T10:30:00.000Z"
}
```

**Headers:** `X-Webhook-Source: artificial-studio`, `X-Media-Id: <generation-id>`

Retries on failure with exponential backoff (5s, 15s, 1m, 5m, 15m, 1h, 6h, 12h, 24h).

## Errors

All errors return `{ "message": "..." }`. Validation errors also include `errors` array.

| Status | Meaning |
|--------|---------|
| 400 | Invalid parameters or unknown fields |
| 401 | Invalid or missing API key |
| 402 | Not enough credits |
| 404 | Tool, model, or generation not found |
| 429 | Rate limit exceeded |
| 5xx | Server error — retry after a few seconds |

## Rate Limits

| Endpoint | Free | Pro |
|----------|------|-----|
| `POST /api/run` | 20 req/min | 60 req/min |
| `GET` endpoints | 60 req/min | 200 req/min |

## Code Examples

### JavaScript — Generate Image with Polling

```javascript
const API_KEY = process.env.ARTIFICIAL_STUDIO_API_KEY;
const BASE = 'https://api.artificialstudio.ai';

async function generate(tool, input) {
  const res = await fetch(`${BASE}/api/run`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json', 'Authorization': API_KEY },
    body: JSON.stringify({ tool, input })
  });
  if (!res.ok) throw new Error((await res.json()).message);
  return res.json();
}

async function poll(id) {
  while (true) {
    const res = await fetch(`${BASE}/api/generations/${id}`, {
      headers: { 'Authorization': API_KEY }
    });
    const gen = await res.json();
    if (gen.status === 'success') return gen;
    if (gen.status === 'error') throw new Error(gen.error);
    await new Promise(r => setTimeout(r, 2000));
  }
}

const { id } = await generate('create-image', {
  prompt: 'A futuristic city at sunset, cyberpunk style'
});
const result = await poll(id);
console.log(result.output);
```

### Python — Generate Image with Polling

```python
import requests, time, os

API_KEY = os.environ['ARTIFICIAL_STUDIO_API_KEY']
BASE = 'https://api.artificialstudio.ai'

def generate(tool, input):
    res = requests.post(f'{BASE}/api/run',
        headers={'Content-Type': 'application/json', 'Authorization': API_KEY},
        json={'tool': tool, 'input': input})
    res.raise_for_status()
    return res.json()

def poll(generation_id):
    while True:
        res = requests.get(f'{BASE}/api/generations/{generation_id}',
            headers={'Authorization': API_KEY})
        gen = res.json()
        if gen['status'] == 'success': return gen
        if gen['status'] == 'error': raise Exception(gen['error'])
        time.sleep(2)

data = generate('create-image', {'prompt': 'A futuristic city at sunset'})
result = poll(data['id'])
print(result['output'])
```

## Best Practices

1. **Always search first** — call `GET /api/search?q=...` to find the right tool and model, and get the exact input schema
2. **Always poll** for results — generations take seconds to minutes depending on the tool
3. **Check credits** before starting expensive generations (video/3D cost more)
4. **Use the default model** (omit `model` field) when the user doesn't specify a preference — it's always the best available
5. **Show the user** the output URL when the generation completes
6. **Handle errors** — if status is `"error"`, the `error` field contains the reason
