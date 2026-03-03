---
name: talking-circle
description: >
  Create animated talking-circle videos (Telegram-style round video messages)
  from avatar frame images and audio. Supports audio-to-video and text-to-video
  via ElevenLabs TTS. Use when the user wants to generate lip-synced circular
  avatar animations, talking circles, or round video messages.
user-invocable: true
argument-hint: "[text or audio path]"
---

# Talking Circle

Create animated circular avatar videos with lip-sync and blink animations. Takes 4 avatar frame images (neutral, slight open, wide open, eyes closed) and produces a round video with audio-driven mouth movement.

## Prerequisites

- `python3` (3.9+)
- `ffmpeg` installed and on PATH
- Optional: `ELEVENLABS_API_KEY` environment variable (for text-to-video mode)

## Setup

Dependencies are auto-installed into a temporary venv on first run. To install manually:

```bash
pip install -r requirements.txt
```

## Mode 1: Audio to Video

Convert existing audio + frame images into an animated talking circle video.

```bash
python3 scripts/make_talking_circle_video.py \
  --neutral frames/neutral.png \
  --slight frames/mouth-slight-open.png \
  --wide frames/mouth-wide-open.png \
  --blink frames/eyes-closed.png \
  --audio speech.mp3 \
  --out /tmp/talking-circle.mp4
```

## Mode 2: Text to Video

Generate speech from text via ElevenLabs TTS, then create the animated video.

Requires `ELEVENLABS_API_KEY` set in environment or passed via `--api-key`.

```bash
python3 scripts/make_text_to_video.py \
  --text "Hello, this is a talking circle demo!" \
  --voice-id YOUR_VOICE_ID \
  --neutral frames/neutral.png \
  --slight frames/mouth-slight-open.png \
  --wide frames/mouth-wide-open.png \
  --blink frames/eyes-closed.png \
  --out /tmp/talking-circle.mp4
```

## Frame Image Requirements

You need 4 PNG images of your avatar, all the same resolution (recommended 2048x2048), square aspect ratio:

| Frame | Description |
|-------|-------------|
| `neutral` | Mouth closed, eyes open |
| `slight` | Mouth slightly open, eyes open |
| `wide` | Mouth wide open, eyes open |
| `blink` | Mouth closed, eyes closed |

See `examples/frames/README.md` for detailed preparation instructions.

## Parameters Reference

### Video output
| Parameter | Default | Description |
|-----------|---------|-------------|
| `--size` | 720 | Output video size in pixels |
| `--diameter` | 640 | Circle diameter within the video |
| `--fps` | 30 | Frames per second |

### Blink timing
| Parameter | Default | Description |
|-----------|---------|-------------|
| `--blink-start` | 1.1 | Seconds before first blink |
| `--blink-every` | 3.8 | Seconds between blinks |
| `--blink-duration-frames` | 4 | Number of frames per blink |

### Amplitude thresholds (audio-to-video)
| Parameter | Default | Description |
|-----------|---------|-------------|
| `--amp-low` | 1200 | RMS below this = neutral (closed mouth) |
| `--amp-high` | 2600 | RMS above this = wide open mouth |

### TTS voice settings (text-to-video)
| Parameter | Default | Description |
|-----------|---------|-------------|
| `--voice-id` | (required) | ElevenLabs voice ID |
| `--model-id` | eleven_multilingual_v2 | ElevenLabs model |
| `--stability` | 0.50 | Voice stability |
| `--similarity-boost` | 0.75 | Voice similarity boost |
| `--style` | 0.00 | Style exaggeration |
| `--speed` | 1.00 | Speech speed |
