# Artificial Studio Skills

Agent skills for integrating [Artificial Studio](https://artificialstudio.ai) — an AI media generation platform — into any AI coding agent.

Compatible with **34+ agents** including Claude Code, Cursor, GitHub Copilot, Gemini CLI, OpenAI Codex, Windsurf, Roo Code, and more.

## Installation

### Install all skills

```bash
npx skills add Artificial-Studio/skills
```

### Install specific skills

```bash
# Only image generation & editing
npx skills add Artificial-Studio/skills --skill artificial-studio-image

# Only video generation & editing
npx skills add Artificial-Studio/skills --skill artificial-studio-video

# Only audio generation & processing
npx skills add Artificial-Studio/skills --skill artificial-studio-audio

# Only 3D object generation
npx skills add Artificial-Studio/skills --skill artificial-studio-3d

# The general skill (covers everything)
npx skills add Artificial-Studio/skills --skill artificial-studio
```

### Install globally (available in all projects)

```bash
npx skills add Artificial-Studio/skills -g
```

### Install for a specific agent

```bash
npx skills add Artificial-Studio/skills --agent claude
npx skills add Artificial-Studio/skills --agent cursor
npx skills add Artificial-Studio/skills --agent copilot
```

## Available Skills

| Skill | Description |
|-------|-------------|
| **artificial-studio** | All capabilities — image, video, audio, 3D generation and editing |
| **artificial-studio-image** | Generate, edit, upscale, and transform images |
| **artificial-studio-video** | Generate, animate, edit, and transform videos |
| **artificial-studio-audio** | Generate speech, music, sound effects, and process audio |
| **artificial-studio-3d** | Generate 3D models and animations |

## Setup

1. Create an account at [artificialstudio.ai](https://artificialstudio.ai)
2. Get your API key at [Settings > API Keys](https://app.artificialstudio.ai/account/api-keys)
3. When your agent asks, provide your API key

## What can you do with these skills?

Once installed, just ask your AI agent naturally:

- *"Generate an image of a futuristic city at sunset"*
- *"Create a 5-second video of a drone flying over mountains"*
- *"Convert this text to speech using a female voice"*
- *"Create a 3D model of a medieval castle"*
- *"Remove the background from this image"*
- *"Upscale this image to higher resolution"*
- *"Animate this photo — make the water flow"*
- *"Split this song into separate instrument tracks"*

The agent will use the Artificial Studio API to fulfill the request and return the result.

## API Overview

All skills use the Artificial Studio REST API:

- **Base URL**: `https://api.artificialstudio.ai`
- **Auth**: API key in `Authorization` header
- **Flow**: Async — start with `POST /api/run`, poll with `GET /api/generations/:id`
- **Docs**: See individual SKILL.md files for detailed API usage

## License

MIT
