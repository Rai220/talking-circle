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

### Critical rules

- All 4 frames must have **identical** resolution, art style, colors, and character positioning.
- Only the mouth and eyes should change between frames — head, body, background must stay the same.
- Do not mix frames from different generation sessions or different styles.

## Generating Frames with Image AI

If the user does not have ready-made frames, generate them using an image generation API (DALL-E, Midjourney, Flux, etc.). Follow this workflow:

### Step 1: Generate the neutral frame

Generate a shoulder-up portrait of the character. This is the base frame — all other frames must match it exactly.

Example prompt:
```
Shoulder-up portrait of [CHARACTER DESCRIPTION]. Square composition, clean background,
mouth closed, eyes open, looking at camera. High detail, consistent lighting.
```

### Step 2: Generate remaining 3 frames as edits of neutral

Use image editing / inpainting on the neutral frame to produce the other states.
Only modify the mouth and eyes region — everything else must remain pixel-identical.

| Frame | What to change | Edit prompt example |
|-------|---------------|-------------------|
| `slight` | Mouth slightly open | `"Mouth slightly open, teeth barely visible, same expression"` |
| `wide` | Mouth wide open | `"Mouth wide open as if saying 'ah', same expression"` |
| `blink` | Eyes closed | `"Eyes gently closed, mouth closed, same expression"` |

### Step 3: Verify consistency

Before using the frames:
1. Check that all 4 images have the same resolution.
2. Overlay them to verify the head/body position hasn't shifted.
3. If any frame drifts, regenerate it from the neutral base.

### Examples

See `examples/` for reference images and ready-to-use frame sets:

- **Rizzi** — anime-style girl, complete frame set included:
  - `examples/rizzi/reference.png` — character reference
  - `examples/rizzi/neutral.png` — mouth closed, eyes open
  - `examples/rizzi/slight.png` — mouth slightly open
  - `examples/rizzi/wide.png` — mouth wide open
  - `examples/rizzi/blink.png` — eyes closed
  - `examples/rizzi/example.mp4` — finished talking-circle video
- **Sbercat** — 3D-rendered anthropomorphic cat, lavender-blue fur, green eyes, pink nose, green hoodie:
  - `examples/sbercat/reference.png` — character reference
  - `examples/sbercat/example.mp4` — finished talking-circle video
  - Frame set not included — generate from reference using the workflow above

To test with the included Rizzi frames:

```bash
python3 scripts/make_talking_circle_video.py \
  --neutral examples/rizzi/neutral.png \
  --slight examples/rizzi/slight.png \
  --wide examples/rizzi/wide.png \
  --blink examples/rizzi/blink.png \
  --audio your-audio.mp3 \
  --out /tmp/talking-circle.mp4
```

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
