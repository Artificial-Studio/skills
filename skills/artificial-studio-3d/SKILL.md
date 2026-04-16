---
name: artificial-studio-3d
description: Generate 3D models and animations from text or images using the Artificial Studio API. Use when the user wants to create 3D objects or motions with AI.
license: MIT
metadata:
  author: artificialstudio
  version: "1.0"
---

# Artificial Studio — 3D Tools

Generate 3D models and animations from text or images using the Artificial Studio API.

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
4. When done, `output` contains the 3D model URL(s) (GLB/OBJ format)

**Always search first** to discover the right tool/model and get the exact input schema.

3D generations can take 1-5 minutes. Poll every 10 seconds.

## 3D Tools

### text-to-3d — Create 3D model from text

```bash
curl -X POST https://api.artificialstudio.ai/api/run \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "text-to-3d",
    "input": {
      "prompt": "A medieval castle with towers and a drawbridge"
    }
  }'
```

**Models**: Meshy V6, Hunyuan3D V3.

**Input fields**:
- `prompt` (required): Describe the 3D object (max 600 characters for Meshy)
- `model` (optional): `"meshy-v6"` or `"hunyuan3d-v3"`

### image-to-3d-object — Create 3D model from image

```bash
curl -X POST https://api.artificialstudio.ai/api/run \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "image-to-3d-object",
    "input": {
      "image_urls": ["https://example.com/object-photo.jpg"]
    }
  }'
```

**Models**: Trellis 2, TripoSR, Hunyuan3D V2.1.

**Input fields**:
- `image_urls` (required): Array with one image URL of the object
- `model` (optional): Model slug

### motion-3d — Generate 3D human motions from text

```bash
curl -X POST https://api.artificialstudio.ai/api/run \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "motion-3d",
    "input": {
      "prompt": "A person doing a backflip"
    }
  }'
```

**Model**: Hunyuan Motion.

**Input fields**:
- `prompt` (required): Describe the motion

## Uploading Local Files

Upload images before using them in image-to-3d:

```bash
curl -X POST https://api.artificialstudio.ai/files \
  -H "Authorization: YOUR_API_KEY" \
  -F "file=@/path/to/image.jpg"
```

Use the returned `url` in `image_urls`.

## Polling for Results

```bash
curl https://api.artificialstudio.ai/api/generations/GENERATION_ID \
  -H "Authorization: YOUR_API_KEY"
```

3D generations typically take 1-5 minutes. Poll every 10 seconds.

## Discovering Tools and Models

### Search by text (recommended)

```bash
curl "https://api.artificialstudio.ai/api/search?q=3d+model" \
  -H "Authorization: YOUR_API_KEY"
```

Returns matching tools with their models and full `inputSchema`.

### Get a specific tool's models

```bash
curl https://api.artificialstudio.ai/api/tools/text-to-3d \
  -H "Authorization: YOUR_API_KEY"
```

Returns all available models with their slugs and input schemas.
