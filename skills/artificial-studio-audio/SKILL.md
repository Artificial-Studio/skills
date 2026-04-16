---
name: artificial-studio-audio
description: Generate speech, music, sound effects, and process audio using the Artificial Studio API. Use when the user wants to create or manipulate audio with AI.
license: MIT
metadata:
  author: artificialstudio
  version: "1.0"
---

# Artificial Studio — Audio Tools

Generate speech, music, sound effects, and process audio using the Artificial Studio API.

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
4. When done, `output` contains the audio URL(s)

**Always search first** to discover the right tool/model and get the exact input schema.

## Audio Generation Tools

### text-to-speech — Convert text to speech

```bash
curl -X POST https://api.artificialstudio.ai/api/run \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "text-to-speech",
    "input": {
      "prompt": "Hello! Welcome to Artificial Studio.",
      "voice": "alloy"
    }
  }'
```

**Models**: OpenAI TTS, ElevenLabs TTS.

**Input fields**:
- `prompt` (required): Text to convert to speech (max 3000 characters)
- `model` (optional): `"openai-tts"` or `"elevenlabs-tts"`
- `voice` (optional): Voice selection
  - OpenAI voices: `"alloy"`, `"echo"`, `"fable"`, `"onyx"`, `"nova"`, `"shimmer"`
  - ElevenLabs voices: `"Adam"`, `"Alice"`, `"Antoni"`, `"Aria"`, `"Arnold"`, `"Bill"`, `"Brian"`, `"Callum"`, `"Charlie"`, `"Charlotte"`, `"Chris"`, `"Daniel"`, `"Dave"`, `"Emily"`, `"Fin"`, `"George"`, `"Lily"`, `"Liam"`, `"Sarah"`, `"Will"`

### create-music — Generate music from text

```bash
curl -X POST https://api.artificialstudio.ai/api/run \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "create-music",
    "input": {
      "prompt": "An upbeat electronic track with a catchy melody"
    }
  }'
```

**Models**: ElevenLabs Music, MusicGen.

**Input fields**:
- `prompt` (required): Describe the music you want
- `model` (optional): Model slug

### sound-effects — Create sound effects

```bash
curl -X POST https://api.artificialstudio.ai/api/run \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "sound-effects",
    "input": {
      "prompt": "Thunder rumbling in the distance with light rain"
    }
  }'
```

### drum-generator — Generate drum beats

```bash
curl -X POST https://api.artificialstudio.ai/api/run \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "drum-generator",
    "input": {}
  }'
```

## Audio Processing Tools

### song-splitter — Separate instruments from songs

```bash
curl -X POST https://api.artificialstudio.ai/api/run \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "song-splitter",
    "input": {
      "audio_url": "https://example.com/song.mp3"
    }
  }'
```

### voice-isolator — Remove background noise

```bash
curl -X POST https://api.artificialstudio.ai/api/run \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "voice-isolator",
    "input": {
      "audio_url": "https://example.com/recording.mp3"
    }
  }'
```

### translate-audio — Translate audio to other languages

```bash
curl -X POST https://api.artificialstudio.ai/api/run \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "translate-audio",
    "input": {
      "audio_url": "https://example.com/speech.mp3"
    }
  }'
```

### Transcription tools

| Tool Slug | Description | Input |
|-----------|-------------|-------|
| `audio-to-text` | Transcribe audio to text | `audio_url` |
| `audio-to-subtitles` | Generate SRT/WebVTT subtitles | `audio_url` |

## Uploading Local Files

Upload audio files before processing:

```bash
curl -X POST https://api.artificialstudio.ai/files \
  -H "Authorization: YOUR_API_KEY" \
  -F "file=@/path/to/audio.mp3"
```

Use the returned `url` in the generation input.

## Polling for Results

```bash
curl https://api.artificialstudio.ai/api/generations/GENERATION_ID \
  -H "Authorization: YOUR_API_KEY"
```

Audio generations typically complete in 5-60 seconds. Poll every 3-5 seconds.

## Discovering Tools and Models

### Search by text (recommended)

```bash
curl "https://api.artificialstudio.ai/api/search?q=text+to+speech" \
  -H "Authorization: YOUR_API_KEY"
```

Returns matching tools with their models and full `inputSchema` — tells you exactly what fields each model accepts (voices, max length, etc.).

### Get a specific tool's models

```bash
curl https://api.artificialstudio.ai/api/tools/text-to-speech \
  -H "Authorization: YOUR_API_KEY"
```

Returns all available models with their slugs and input schemas.
