# talking-circle

A Claude Code skill for creating animated talking-circle videos (Telegram-style round video messages) from avatar frame images and audio.

## Installation

```bash
git clone https://github.com/nicekrestnikov/talking-circle.git ~/.claude/skills/talking-circle
```

## Quick Start

### Audio to Video

```bash
python3 ~/.claude/skills/talking-circle/scripts/make_talking_circle_video.py \
  --neutral neutral.png --slight slight.png --wide wide.png --blink blink.png \
  --audio speech.mp3 --out /tmp/talking-circle.mp4
```

### Text to Video

Requires `ELEVENLABS_API_KEY` environment variable.

```bash
python3 ~/.claude/skills/talking-circle/scripts/make_text_to_video.py \
  --text "Hello world!" --voice-id YOUR_VOICE_ID \
  --neutral neutral.png --slight slight.png --wide wide.png --blink blink.png \
  --out /tmp/talking-circle.mp4
```

## Prerequisites

- Python 3.9+
- ffmpeg installed and on PATH
- Optional: [ElevenLabs](https://elevenlabs.io) API key (for text-to-video mode)

Dependencies (`numpy`, `pillow`, `requests`) are auto-installed into a temporary venv on first run, or install manually:

```bash
pip install -r ~/.claude/skills/talking-circle/requirements.txt
```

## Frame Preparation

You need 4 square PNG images of your avatar at the same resolution (recommended 2048x2048):

| Frame | File | Description |
|-------|------|-------------|
| Neutral | `neutral.png` | Mouth closed, eyes open |
| Slight open | `slight.png` | Mouth slightly open, eyes open |
| Wide open | `wide.png` | Mouth wide open, eyes open |
| Blink | `blink.png` | Mouth closed, eyes closed |

See [`examples/frames/README.md`](examples/frames/README.md) for detailed instructions.

## How It Works

1. Audio is converted to mono 16kHz WAV
2. For each video frame, RMS amplitude is computed over a 35ms window centered on the frame's timestamp
3. The RMS value determines which mouth frame to show:
   - Below `--amp-low` (1200): neutral (closed)
   - Between `--amp-low` and `--amp-high` (2600): slight open
   - Above `--amp-high`: wide open
4. Blink frames are overlaid at regular intervals
5. If audio analysis produces a static result (all frames the same), a fallback animation cycle is used
6. Frames are composited into a circle with a subtle border, then encoded with ffmpeg

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
