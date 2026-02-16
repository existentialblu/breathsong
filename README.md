# BreathSong

### Your CPAP data, but make it a banger

BreathSong turns your PAP therapy data into audio so you can *hear* what your breathing did all night. Because looking at squiggly lines is so 2024.

## What Even Is This

You know how your CPAP/BiPAP/ASV machine records everything while you sleep? Flow rates, pressure changes, leaks, minute ventilation — hours and hours of data that normally lives in spreadsheets and charts?

BreathSong compresses an entire night into ~5 minutes of audio and maps each data channel to a different sound. Apneas become silence. Periodic breathing becomes tremolo. Your machine chasing your ventilation becomes dueling sine waves. It's like if your sleep study had a soundtrack.

## The Channels

| What | Sound | Why It Sounds Like That |
|------|-------|------------------------|
| **Flow** | 220Hz sine wave, amplitude-modulated | Breathe in = loud, breathe out = quiet, stop breathing = silence. Simple. |
| **Pressure** | 70-210Hz triangle wave | Low rumbling drone that rises when your machine pushes harder. It's the bass player. |
| **Leak** | Pink noise | Because leaks literally sound like hissing. Art imitates life. |
| **IPAP** | 350-700Hz sine, *anti-phase to MV* | This one's diabolical — see below. |
| **Minute Ventilation** | 350-700Hz sine | THE loop gain channel. Periodic breathing = pitch vibrato. |

### The IPAP/MV Trick

IPAP and MV are the same waveform, same frequency range, but 180° out of phase. When your ASV machine is perfectly tracking your ventilation (IPAP matches MV), they cancel out and you hear **silence**. When the servo is hunting and they diverge, you hear **beating**. The beat frequency literally *is* the servo error. It's unhinged and it works.

## How To Use It

1. Open `breathsong.html` in a browser. That's it. No install, no server, no Python, no npm, no Docker, no Kubernetes, no microservices architecture. It's a single HTML file and it's beautiful.
2. Drag your EDF files onto it:
   - **ResMed**: Drop `BRP.edf` + `PLD.edf` from the same session for all 5 channels
   - **Philips**: Drop `.005`/`.006` waveform files (DreamStation 2 encryption handled automatically)
3. Hit play
4. Adjust the mixer — each channel has volume, pan, and mute
5. Export as WAV if you want to keep your night's symphony

## ResMed vs Philips: A Love Letter to Data Formats

ResMed gives you 25Hz flow and pressure waveforms in standard EDF files, plus a separate PLD file with 0.5Hz leak, minute ventilation, IPAP, and more. It's a feast.

Philips gives you numbered binary files with model-dependent byte offsets, optional AES-256-GCM encryption, and summary data that amounts to one value per session. The waveform files (.005) have flow and pressure, which is genuinely useful, but the trend data situation is... *an architectural choice*.

**TL;DR**: Full 5-channel experience is ResMed only. Philips gets flow + pressure + derived MV. We work with what we're given.

## The Nerdy Bits

- **Compression**: A 7.5-hour session compresses to 5 minutes at ~90:1. A 60-second loop gain cycle becomes a 0.67-second audible wobble.
- **Audio**: Pre-rendered at 11025Hz into AudioBuffers. Playback via Web Audio API with per-channel GainNode + StereoPannerNode.
- **FM synthesis**: Pressure, IPAP, and MV channels use phase-continuous frequency modulation (stolen from a Python script that predates this project by a year).
- **Philips decryption**: Full AES-256-GCM + PBKDF2-SHA256 decryption chain runs in-browser via Web Crypto API. The keys are from OSCAR. Thanks, OSCAR.
- **WAV export**: Stereo mixdown with equal-power pan law.

## Requirements

- A browser (Chrome, Edge, Firefox — anything with Web Audio API)
- CPAP data files from your SD card
- Headphones recommended (the spatial panning is part of the experience)
- A willingness to listen to what you sound like when you're not conscious

## Credits

Built by a sleep apnea patient who got tired of *looking* at her data and wanted to *hear* it instead.

Made with help from Claude, who wrote most of the code while being gently bullied into adding more audio channels.

Philips decryption based on [OSCAR](https://www.sleepfiles.com/OSCAR/)'s reverse engineering work — absolute legends.

## License

Do whatever you want with it. If your sleep doctor asks why you're playing them a weird five-minute ambient track, tell them it's diagnostic.
