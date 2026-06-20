# Demo Video Source

This directory contains the HyperFrames composition source used to render the PermitOps OS demo video.

## Files

- `index.html` — Main HTML composition (9 scenes, 1920×1080, 108s)
- `narration-script.md` — Full narration script used for voice generation
- `FRAME.md` — Frame specification (objective, audience, narrative, visual language)
- `storyboard.md` — Scene-by-scene storyboard
- `package.json` — HyperFrames project config
- `hyperframes.json` — Project metadata

## How to render

```bash
npm install -g hyperframes
npm install
npm run dev    # preview in browser
npm run render # render to MP4
```

## Production notes

- Narration generated via ElevenLabs voice cloning
- Background music synthesized with numpy (C-Am-F-G warm corporate pad with sidechain ducking)
- Voice processed with ffmpeg: highpass, FFT noise reduction, compression, broadcast loudness normalization
- Final mix: voice + ducked BGM, stereo, 44.1kHz, 192kbps AAC
