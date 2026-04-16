---
name: artificial-studio-image
description: Generate, edit, upscale, and transform images using the Artificial Studio API. Use when the user wants to create or manipulate images with AI.
license: MIT
metadata:
  author: artificialstudio
  version: "1.0"
---

# Artificial Studio — Image Tools

Generate, edit, upscale, and transform images using the Artificial Studio API.

## Authentication

All requests require an API key in the `Authorization` header (no `Bearer` prefix).

1. First, check the environment variable `ARTIFICIAL_STUDIO_API_KEY`. If it's set, use it automatically.
2. If not set, ask the user for their API key before making any request.
3. Tell them they can get a key at https://app.artificialstudio.ai/account/api-keys
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
4. When done, `output` contains the image URL(s)

**Always search first** to discover the right tool/model and get the exact input schema.

## Image Tools

### create-image — Generate images from text

```bash
curl -X POST https://api.artificialstudio.ai/api/run \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "create-image",
    "input": {
      "prompt": "A futuristic city at sunset",
      "image_size": "landscape_16_9"
    }
  }'
```

**Models**: Seedream5-Lite, Seedream4.5, Seedream4, Ideogram v3, Flux 2 Max, Flux 2, Flux Schnell, Recraft v4, Recraft v3, GPT Image 1, Google Image 4, Grok Imagine, Qwen Image 2, and more.

**Input fields**:
- `prompt` (required): Text description of the image
- `model` (optional): Model slug. Omit to use the default (recommended)
- `image_size` (optional): `"square_hd"`, `"square"`, `"portrait_4_3"`, `"portrait_16_9"`, `"landscape_4_3"`, `"landscape_16_9"`

### edit-image — Edit images with text prompts

```bash
curl -X POST https://api.artificialstudio.ai/api/run \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "edit-image",
    "input": {
      "prompt": "Change the background to a beach at sunset",
      "image_urls": ["https://example.com/photo.jpg"]
    }
  }'
```

**Models**: NanoBanana 2 Edit, Seedream4.5 Edit, Flux Kontext, GPT Image 1 Edit, Grok Imagine Edit, Qwen Image 2 Edit, and more.

**Input fields**:
- `prompt` (required): Describe the changes you want
- `image_urls` (required): Array of image URLs (1-5 depending on model)
- `model` (optional): Model slug
- `aspect_ratio` (optional): `"auto"`, `"1:1"`, `"16:9"`, `"9:16"`, `"4:3"`, `"3:4"`, `"3:2"`, `"2:3"`, `"21:9"`

### extend-image — Extend/reframe images

```bash
curl -X POST https://api.artificialstudio.ai/api/run \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "extend-image",
    "input": {
      "prompt": "Extend to show more of the landscape",
      "image_urls": ["https://example.com/photo.jpg"]
    }
  }'
```

### upscale-image — Increase image resolution

```bash
curl -X POST https://api.artificialstudio.ai/api/run \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "upscale-image",
    "input": {
      "image_urls": ["https://example.com/photo.jpg"]
    }
  }'
```

### image-background-remover — Remove background

```bash
curl -X POST https://api.artificialstudio.ai/api/run \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "image-background-remover",
    "input": {
      "image_urls": ["https://example.com/photo.jpg"]
    }
  }'
```

### style-transfer — Transfer style between images

```bash
curl -X POST https://api.artificialstudio.ai/api/run \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "style-transfer",
    "input": {
      "prompt": "Apply the style to the content image",
      "image_urls": ["https://example.com/content.jpg", "https://example.com/style.jpg"]
    }
  }'
```

### Other image tools

| Tool Slug | Description | Input |
|-----------|-------------|-------|
| `image-depth-map-generator` | Create depth map from image | `image_urls` |
| `image-to-svg` | Convert image to SVG | `image_urls` |
| `svg-creator` | Create SVG from text | `prompt` |
| `try-clothes` | Virtual try-on | `image_urls` (person + garment) |
| `image-canvas` | Transform sketches to images | `prompt` + `image_urls` |

## Uploading Local Files

If you need to provide a local image, upload it first:

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

Poll every 3-5 seconds. Image generations typically complete in 5-30 seconds.

## Discovering Tools and Models

### Search by text (recommended)

```bash
curl "https://api.artificialstudio.ai/api/search?q=edit+image" \
  -H "Authorization: YOUR_API_KEY"
```

Returns matching tools with their models and full `inputSchema` — tells you exactly what fields each model accepts.

### Get a specific tool's models

```bash
curl https://api.artificialstudio.ai/api/tools/create-image \
  -H "Authorization: YOUR_API_KEY"
```

Returns all available models with their slugs and input schemas. Use the default model (omit `model` field) when the user has no preference.
