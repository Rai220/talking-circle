# Frame Preparation Guide

The talking-circle skill requires 4 PNG images of your avatar character. Each image represents a different mouth/eye state used for lip-sync and blink animation.

## Required Frames

### 1. `neutral.png` — Mouth closed, eyes open
The default resting state. Shown when audio amplitude is low (silence or quiet).

### 2. `mouth-slight-open.png` — Mouth slightly open, eyes open
Shown during moderate speech volume. The mouth should be noticeably open but not at maximum.

### 3. `mouth-wide-open.png` — Mouth wide open, eyes open
Shown during loud speech. The mouth should be at its widest natural opening.

### 4. `eyes-closed.png` — Mouth closed, eyes closed
Used for periodic blink animation. Same as neutral but with eyes closed.

## Image Requirements

- **Format:** PNG
- **Resolution:** All 4 images must be the same resolution
- **Recommended size:** 2048x2048 pixels (will be downscaled for the video)
- **Aspect ratio:** Square (1:1)
- **Style consistency:** All frames should have the same art style, colors, and character positioning
- **Background:** Can be any color — the skill crops to a circle automatically

## Tips

- Keep the character's head position identical across all 4 frames — only the mouth and eyes should change
- Use clean, high-contrast artwork for best results at small circle sizes
- If using AI-generated art, generate all 4 frames from the same seed/style for consistency
- Test with a short audio clip first to verify the frames look natural in motion
