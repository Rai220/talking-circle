# talking-circle

An [OpenClaw](https://github.com/nicekrestnikov/openclaw) skill for creating animated talking-circle videos (Telegram-style round video messages) from avatar frame images and audio.

| Rizzi | Sbercat |
|-------|---------|
| [examples/rizzi/example.mp4](examples/rizzi/example.mp4) | [examples/sbercat/example.mp4](examples/sbercat/example.mp4) |

## Quick Start

Send this message to your OpenClaw assistant in Telegram:

> Установи скилл talking-circle для генерации видео-кружочков.
> Репозиторий: https://github.com/nicekrestnikov/talking-circle
> После установки сгенерируй опорные кадры для моего аватара и отправь мне тестовый кружочек со словами "Привет, это мой первый кружочек!"

The assistant will:
1. Clone the skill repository.
2. Generate 4 frame images for your character (neutral, slight open, wide open, blink) using image AI.
3. Synthesize speech via ElevenLabs TTS.
4. Render an animated talking-circle video and send it back.

### Manual installation

```bash
git clone https://github.com/nicekrestnikov/talking-circle.git ~/.claude/skills/talking-circle
```

### Prerequisites

- Python 3.9+
- ffmpeg installed and on PATH
- Optional: [ElevenLabs](https://elevenlabs.io) API key (for text-to-video mode)

Dependencies (`numpy`, `pillow`, `requests`) are auto-installed into a temporary venv on first run, or install manually:

```bash
pip install -r ~/.claude/skills/talking-circle/requirements.txt
```

## What is this?

Talking-circle takes 4 images of a character's face (mouth closed, slightly open, wide open, eyes closed) and an audio file, then produces an animated circular video where the character "speaks" in sync with the audio. The result looks like a Telegram video message ("kruzhochek").

The skill can be used by an AI assistant (via OpenClaw) to:
1. Generate a talking-circle video from existing audio and frame images.
2. Generate speech from text via ElevenLabs TTS and produce the video in one step.

## How It Works

1. Audio is converted to mono 16 kHz WAV.
2. For each video frame, RMS amplitude is computed over a 35 ms window centered on the frame's timestamp.
3. The RMS value selects which mouth frame to show:
   - Below `--amp-low` (default 1200): neutral (mouth closed)
   - Between `--amp-low` and `--amp-high` (default 2600): slightly open
   - Above `--amp-high`: wide open
4. Blink frames are overlaid at regular intervals (configurable).
5. If audio analysis produces a static result (e.g. silence), a fallback animation cycle is used.
6. Frames are composited into a circle with a subtle border, then encoded with ffmpeg.

## CLI Usage

### Audio to Video

```bash
python3 scripts/make_talking_circle_video.py \
  --neutral neutral.png --slight slight.png --wide wide.png --blink blink.png \
  --audio speech.mp3 --out /tmp/talking-circle.mp4
```

### Text to Video

Requires `ELEVENLABS_API_KEY` environment variable.

```bash
python3 scripts/make_text_to_video.py \
  --text "Hello world!" --voice-id YOUR_VOICE_ID \
  --neutral neutral.png --slight slight.png --wide wide.png --blink blink.png \
  --out /tmp/talking-circle.mp4
```

## Frame Preparation

You need 4 square PNG images of your character at the same resolution (recommended 2048x2048):

| Frame | Description |
|-------|-------------|
| Neutral | Mouth closed, eyes open — the default resting state |
| Slight open | Mouth slightly open, eyes open — moderate speech |
| Wide open | Mouth wide open, eyes open — loud speech |
| Blink | Mouth closed, eyes closed — periodic blink animation |

### Critical rules

- All 4 frames must have **identical** resolution, art style, colors, and character positioning.
- Only the mouth and eyes should differ between frames — head, body, background stay the same.
- Do not mix frames from different generation sessions.

### Generating frames with Image AI

If you don't have ready-made frames, generate them using an image generation API (DALL-E, Midjourney, Flux, etc.):

**Step 1.** Generate the neutral frame — a shoulder-up portrait with mouth closed, eyes open.

**Step 2.** Use image editing / inpainting on the neutral frame to produce the other 3 states:

| Frame | Edit region | Prompt hint |
|-------|-----------|-------------|
| Slight open | Mouth only | `"Mouth slightly open, teeth barely visible"` |
| Wide open | Mouth only | `"Mouth wide open as if saying 'ah'"` |
| Blink | Eyes only | `"Eyes gently closed, mouth closed"` |

**Step 3.** Verify all 4 images have the same resolution and that head/body position hasn't shifted.

See [`examples/frames/README.md`](examples/frames/README.md) for detailed instructions.

## Examples

### Rizzi

Anime-style AI assistant character.

**Reference and output:**

| Reference | Output |
|-----------|--------|
| <img src="examples/rizzi/reference.png" width="200"> | [example.mp4](examples/rizzi/example.mp4) |

**Frame set:**

| Neutral | Slight open | Wide open | Blink |
|---------|-------------|-----------|-------|
| <img src="examples/rizzi/neutral.png" width="150"> | <img src="examples/rizzi/slight.png" width="150"> | <img src="examples/rizzi/wide.png" width="150"> | <img src="examples/rizzi/blink.png" width="150"> |

### Sbercat

3D-rendered anthropomorphic cat character.

| Reference | Output |
|-----------|--------|
| <img src="examples/sbercat/reference.png" width="200"> | [example.mp4](examples/sbercat/example.mp4) |

## Parameters

### Video output
| Parameter | Default | Description |
|-----------|---------|-------------|
| `--size` | 720 | Output video size in pixels |
| `--diameter` | 640 | Circle diameter |
| `--fps` | 30 | Frames per second |

### Blink timing
| Parameter | Default | Description |
|-----------|---------|-------------|
| `--blink-start` | 1.1s | Delay before first blink |
| `--blink-every` | 3.8s | Interval between blinks |
| `--blink-duration-frames` | 4 | Frames per blink |

### Amplitude thresholds
| Parameter | Default | Description |
|-----------|---------|-------------|
| `--amp-low` | 1200 | RMS threshold for neutral vs slight |
| `--amp-high` | 2600 | RMS threshold for slight vs wide |

### TTS settings (text-to-video only)
| Parameter | Default | Description |
|-----------|---------|-------------|
| `--voice-id` | (required) | ElevenLabs voice ID |
| `--model-id` | eleven_multilingual_v2 | ElevenLabs model |
| `--stability` | 0.50 | Voice stability |
| `--similarity-boost` | 0.75 | Similarity boost |
| `--style` | 0.00 | Style exaggeration |
| `--speed` | 1.00 | Speech speed |

## License

MIT
