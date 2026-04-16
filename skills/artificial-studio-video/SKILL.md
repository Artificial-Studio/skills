---
name: artificial-studio-video
description: Generate, edit, animate, upscale, and transform videos using the Artificial Studio API. Use when the user wants to create or manipulate videos with AI.
license: MIT
metadata:
  author: artificialstudio
  version: "1.0"
---

# Artificial Studio — Video Tools

Generate, edit, animate, upscale, and transform videos using the Artificial Studio API.

## Authentication

All requests require an API key in the `Authorization` header (no `Bearer` prefix).

1. First, check the environment variable `ARTIFICIAL_STUDIO_API_KEY`. If it's set, use it automatically.
2. If not set, ask the user for their API key before making any request.
3. Tell them they can get a key at https://artificialstudio.ai/settings/api-keys
4. Suggest: `export ARTIFICIAL_STUDIO_API_KEY=your_key_here`

## Base URL

```
https://api.artificialstudio.ai
```

## Workflow

All generations are asynchronous:

1. `GET /api/search?q=...` — search for the right tool and model (returns tools with full input schemas)
2. `POST /api/run` — start a generation (returns `id` with status `pending`)
3. `GET /api/generations/:id` — poll until status is `success` or `error`
4. When done, `output` contains the video URL(s)

**Always search first** to discover the right tool/model and get the exact input schema.

Video generations take longer than images — poll every 5-10 seconds, and expect 30 seconds to several minutes.

## Video Tools

### create-video — Generate video from text

```bash
curl -X POST https://api.artificialstudio.ai/api/run \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "create-video",
    "input": {
      "prompt": "A drone flying over a tropical island at golden hour",
      "duration": "5"
    }
  }'
```

**Models**: Seedance 2, Seedance 2 Fast, Sora 2, Sora 2 Pro, Kling 3.0 Pro, Kling 3.0 Standard, Veo 3.1, Veo 3, Luma Ray 2, Minimax, Wan 2.2, and more.

**Input fields**:
- `prompt` (required): Text description of the video
- `model` (optional): Model slug. Omit to use the default
- `duration` (optional): `"5"` or `"10"` seconds (model-dependent)

### animate-image — Create video from a static image

```bash
curl -X POST https://api.artificialstudio.ai/api/run \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "animate-image",
    "input": {
      "prompt": "The camera slowly zooms in as the wind blows through the trees",
      "image_urls": ["https://example.com/photo.jpg"]
    }
  }'
```

**Models**: Seedance 2, Wan 2.2, Kling O3, Veo 3.1, Luma Ray 2, Minimax, and more.

**Input fields**:
- `prompt` (required): Describe the motion/animation
- `image_urls` (required): Array with one image URL
- `model` (optional): Model slug
- `duration` (optional): `"5"` or `"10"` (model-dependent)

### edit-video — Edit video with text prompts

```bash
curl -X POST https://api.artificialstudio.ai/api/run \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "edit-video",
    "input": {
      "prompt": "Change the sky to a dramatic sunset",
      "video_url": "https://example.com/video.mp4"
    }
  }'
```

### extend-video — Extend video duration

```bash
curl -X POST https://api.artificialstudio.ai/api/run \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "extend-video",
    "input": {
      "prompt": "Continue the scene naturally",
      "video_url": "https://example.com/video.mp4"
    }
  }'
```

### motion-transfer — Transfer motion to a character

```bash
curl -X POST https://api.artificialstudio.ai/api/run \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "motion-transfer",
    "input": {
      "image_urls": ["https://example.com/character.jpg"],
      "video_url": "https://example.com/reference-motion.mp4"
    }
  }'
```

**Models**: Kling Motion Control v3 Pro, v3 Standard, v2.6 Pro, v2.6 Standard.

### Other video tools

| Tool Slug | Description | Input |
|-----------|-------------|-------|
| `upscale-video` | Increase video resolution | `video_url` |
| `video-background-remover` | Remove video background | `video_url` |
| `translate-video` | Translate video to other languages | `video_url` |
| `video-sync-audio` | Generate synchronized audio | `video_url` |
| `character-replace` | Replace characters in video | `image_urls` + `video_url` |

## Uploading Local Files

Upload videos or images before using them:

```bash
curl -X POST https://api.artificialstudio.ai/files \
  -H "Authorization: YOUR_API_KEY" \
  -F "file=@/path/to/video.mp4"
```

Use the returned `url` in the generation input.

## Polling for Results

```bash
curl https://api.artificialstudio.ai/api/generations/GENERATION_ID \
  -H "Authorization: YOUR_API_KEY"
```

Video generations can take 30 seconds to several minutes. Poll every 5-10 seconds.

## Discovering Tools and Models

### Search by text (recommended)

```bash
curl "https://api.artificialstudio.ai/api/search?q=animate+image" \
  -H "Authorization: YOUR_API_KEY"
```

Returns matching tools with their models and full `inputSchema`.

### Get a specific tool's models

```bash
curl https://api.artificialstudio.ai/api/tools/create-video \
  -H "Authorization: YOUR_API_KEY"
```

Returns all available models with their slugs and input schemas.
