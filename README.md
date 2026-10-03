# Violin Practice Room

**Live site:** [violin.daiyip.com](https://violin.daiyip.com)

A personal practice companion for learning the violin, in a single web page. It needs no build step and no server.

## Tools

| Tab | What it does |
| --- | --- |
| **Today** | Builds a timed practice plan (15–60 min) from your log, your pieces and your ear-training results. Each step opens the tool it needs. |
| **Tuner** | Reference tones for G, D, A and E, plus a live cents meter from the microphone. |
| **Metronome** | Accents, subdivisions, tap tempo, Italian tempo markings and a speed trainer that adds a few bpm every few bars. |
| **Drone** | A sustained tonic (with optional fifth) in any key, for intonation practice. |
| **Fingerboard** | A first-position map with finger numbers, scale highlighting and high/low 2nd-finger patterns per key, plus a note-naming quiz. |
| **Ear training** | Interval recognition (with a tune to remember each interval) and a sharp/flat/in-tune drill that narrows to the smallest pitch difference you can hear. |
| **Listen back** | Open a recording: pitch trace and per-note intonation (cents sharp/flat), slow-down playback with A–B loop, and a rhythm check (tempo you played, rushing/dragging, evenness). |
| **Posture** | Open a video of yourself playing (or use the camera): checks shoulder, head tilt, left wrist, bow path and bow elbow using on-device pose tracking. |
| **Practice log** | Session timer, log with focus tags and notes, weekly chart, streak, and a repertoire tracker (current vs goal tempo). |

All audio analysis is plain signal processing (YIN pitch detection, onset detection). The posture tab uses Google's MediaPipe pose model, which runs entirely in the browser; no audio or video leaves your device.

## Running it

Serve the folder over HTTP so the bundled pose-tracking files load:

```sh
python3 -m http.server 8000
# then open http://localhost:8000
```

Opening `index.html` directly from disk also works. In that case the posture tab loads MediaPipe from the jsDelivr and Google CDNs instead of `pose/`. The microphone and camera need `localhost` or HTTPS, so GitHub Pages works well too.

Your log, repertoire and settings are stored in the browser's `localStorage`. When the page runs as a Claude artifact, the log and repertoire sync to your account instead.

## Files

- `index.html` contains the whole app: HTML, CSS and JavaScript.
- `pose/` holds MediaPipe Tasks Vision 0.10.14 (Apache 2.0) and the `pose_landmarker_lite` model, stored base64-encoded as `.b64.txt`.
- `docs/feature-brainstorm.md` is the original feature brainstorm.
