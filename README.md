# Violin Practice Room

**Live site:** [violin.daiyip.com](https://violin.daiyip.com)

A personal practice companion for learning the violin, in a single web page. It needs no build step and no server.

## Tools

| Tab | What it does |
| --- | --- |
| **Today** | Builds a timed practice plan (15–60 min) from your log, your pieces and your ear-training results. Each step opens the tool it needs. |
| **Tuner** | Reference tones for G, D, A and E, plus a live cents meter from the microphone. |
| **Metronome** | Tap each beat to make it accented, normal or silent; 1–7 beats, subdivisions, tap tempo, silent bars for checking your pulse, and a speed trainer (steady climb or two up, one back) from a start tempo to a goal. |
| **Drone** | A sustained tonic (with optional fifth) in any key, for intonation practice. |
| **Fingerboard** | A first-position map with finger numbers, scale highlighting and high/low 2nd-finger patterns per key, plus a note-naming quiz. |
| **Scales** | Major and minor scales and arpeggios in one or two octaves. It listens as you play, scores each note in cents, and says which finger to move. Recent runs are kept so you can see progress. |
| **Sheet music** | Photograph a page of music and Claude or Gemini Flash (your choice) transcribes it to ABC notation. The app draws the notes so you can check them against the photo and fix misreads, then plays them (violin or piano sound) at any tempo with a metronome count-in or click track, or clicks only. It can also hand the tempo and time signature to the metronome, save pieces, and export a MIDI file. |
| **Ear training** | Interval recognition (with a tune to remember each interval) and a sharp/flat/in-tune drill that narrows to the smallest pitch difference you can hear. |
| **Listen back** | Open a recording: pitch trace and per-note intonation (cents sharp/flat), slow-down playback with A–B loop, and a rhythm check (tempo you played, rushing/dragging, evenness). |
| **Posture** | Open a video of yourself playing (or use the camera): checks shoulder, head tilt, left wrist, bow path and bow elbow using on-device pose tracking. |
| **Practice log** | Session timer, log with focus tags and notes, weekly chart, streak, and a repertoire tracker (current vs goal tempo). |

All audio analysis is plain signal processing (YIN pitch detection, onset detection). The posture tab uses Google's MediaPipe pose model, which runs entirely in the browser; no audio or video leaves your device.

The sheet-music tab is the one exception: reading a photo sends it either to the Claude API (model `claude-opus-5-5`) or to the Gemini API (model `gemini-3.7-flash`), using your own API key for that service, which is stored only in your browser. On Gemini's free tier, Google may use what you send to improve its products. The Anthropic SDK is bundled in `anthropic/` rather than loaded from a CDN, so no third-party script runs on the page that holds the key; Gemini is called with a plain `fetch`, with no SDK. Drawing, playback and MIDI export run locally with [abcjs](https://www.abcjs.net/); the instrument samples download from the abcjs soundfont site the first time you press Play.

## Running it

Serve the folder over HTTP so the bundled pose-tracking files load:

```sh
python3 -m http.server 8000
# then open http://localhost:8000
```

Opening `index.html` directly from disk also works. In that case the posture tab loads MediaPipe from the jsDelivr and Google CDNs instead of `pose/`. The microphone and camera need `localhost` or HTTPS, so GitHub Pages works well too.

Your log, repertoire and settings are stored in the browser's `localStorage`. Saved sheet-music pieces (notes, tempo and the original photos) are kept in the browser's IndexedDB. When the page runs as a Claude artifact, the log and repertoire sync to your account instead.

## Google Drive sync for sheet music

The sheet-music tab can sync your saved pieces to a "Violin Practice Room" folder in Google Drive, so they survive a reinstall and appear on every device. Sign-in is a plain OAuth redirect to Google and back, so no Google script runs on the page. The `drive.file` scope lets the app see only the files it created.

One-time setup by the app's owner in [Google Cloud Console](https://console.cloud.google.com/):

1. Use a project (new or existing) and enable the **Google Drive API**.
2. On the **OAuth consent screen**, choose External, add the `.../auth/drive.file` scope, and add your Google account as a test user.
3. Create an **OAuth client ID** of type Web application:
   - Authorized JavaScript origins: `https://violin.daiyip.com`
   - Authorized redirect URIs: `https://violin.daiyip.com/`
4. Put the client ID in `GOOGLE_CLIENT_ID` in `index.html` (violin.daiyip.com's is already there), or paste it under "Google client ID" in the app. It is not a secret.

While the consent screen is in Testing, only the listed test users can sign in. Sync is last-writer-wins per piece; deleting a piece on one device deletes it everywhere after the next sync.

## Files

- `index.html` contains the whole app: HTML, CSS and JavaScript.
- `pose/` holds MediaPipe Tasks Vision 0.10.14 (Apache 2.0) and the `pose_landmarker_lite` model, stored base64-encoded as `.b64.txt`.
- `abcjs/` holds abcjs 6.7.1 (MIT), which draws and plays the sheet-music tab's notation.
- `anthropic/sdk.min.mjs` is the Anthropic TypeScript SDK 0.131.0 (MIT) built as a single browser ES module for the sheet-music tab. To rebuild it for a newer version:

  ```sh
  npm i @anthropic-ai/sdk@<version> esbuild
  npx esbuild node_modules/@anthropic-ai/sdk/index.mjs --bundle --format=esm --platform=browser --minify --legal-comments=eof --outfile=anthropic/sdk.min.mjs
  ```
- `docs/feature-brainstorm.md` is the original feature brainstorm.
