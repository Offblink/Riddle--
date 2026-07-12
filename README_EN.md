# Riddle--

A browser-based reimagining of [MaximeRivest/riddle](https://github.com/MaximeRivest/riddle) — the magical diary of Tom Riddle. No e-ink hardware required. Just a browser, a pen (or mouse), and an API key.

## Riddle vs. Riddle--

| | **riddle** | **riddle--** |
|---|---|---|
| **Platform** | reMarkable Paper Pro (e-ink, aarch64) | Any browser (Windows / macOS / Linux / phone) |
| **Language** | Rust + C/C++ | Single HTML file + vanilla JS |
| **Display backend** | Direct e-ink engine (quill) or X11 (qtfb) | Canvas |
| **Pen input** | evdev, 4096-level pressure | Pointer events (mouse + touch) |
| **Handwriting recognition** | Vision LLM (GPT-4o reads a PNG screenshot) | **Google IME stroke API** (sends coordinate arrays, not images) |
| **LLM for replies** | Same vision model (reads image, writes text) | **Pure text LLM** (DeepSeek / OpenAI-compatible) |
| **Handwriting synthesis** | Font → Zhang-Suen skeletonization → stroke trace → animated replay | Canvas `fillText` + clip-reveal animation, centered layout |
| **Customization** | Fixed font | Selectable font (presets + system fonts), size, speed |
| **Setup** | Developer mode + SSH + AppLoad | Open index.html |
| **Cost per interaction** | ~500–1000 vision tokens | ~50 text tokens (*~10–20× cheaper*) |
| **Offline** | No (requires API) | No (requires API) |
## Why not Vision LLM?

The original riddle sends a PNG screenshot of the entire page to GPT-4o for both handwriting reading AND reply generation. That's 500+ KB of image data for what could be a few bytes of stroke coordinates.

Riddle-- does what a handwriting recognizer should do:

```
Canvas strokes → Google IME API (stroke→text) → Pure-text LLM → Canvas animation
```

- The recognizer is a dedicated, free, stroke-based service — not a $20/Mtok generalist
- The LLM only sees text — you pay for words, not pixels
- The architecture is swappable: replace the recognizer, replace the LLM, independently

## Quick start

```bash
# 1. Copy config
cp config.example.js config.js

# 2. Edit config.js — add your API key
#    (DeepSeek, OpenAI, or any compatible provider)
# 3. Open index.html directly
```

Settings can also be changed in-app (⚙ gear icon) and persist to localStorage.

## Project structure

```
riddle--/
├── index.html          # The entire app (no build, no dependencies beyond config.js)
├── config.js           # Your API key (gitignored)
├── config.example.js   # Template (safe to commit)
├── Monocraft.ttf       # UI font (Minecraft-style monospace)
├── .gitignore
└── README.md
```

## Actions

| Action | Result |
|---|---|
| Write, then rest 2.8s | User ink + old reply both fade out, diary replies |
| Ctrl+Z or ↩ button | Undo last stroke |
| ⚙ button | Open settings panel |
| ▷ button (bottom-right) | Toggle debug log |

## Settings

Click ⚙ to adjust (auto-saved to localStorage):

| Setting | Description |
|---|---|
| Handwriting language | 30+ languages (Chinese, English, Japanese, Korean, etc.) |
| Reply font | 9 presets (Ma Shan Zheng, Caveat, KaiTi, etc.) + auto-load system fonts |
| Font size | 20–48px (auto-scaled to fit canvas) |
| Speed | `−` / `+` adjust per-character animation speed (50–500ms), live preview on main canvas |
| API Key / Base / Model | LLM config (config.js values take priority) |
| System Prompt | Role / personality |


## Handwriting recognition backend

Default: **Google IME handwriting API** (adapted from [handwriting.js](https://github.com/ChenYuHo/handwriting.js), MIT).

The API accepts stroke coordinate arrays and returns recognized text. Supported languages: Chinese (Simplified/Traditional), English, Japanese, Korean, and 30+ others.

> ⚠️ Google's endpoint may be inaccessible inside mainland China. The recognizer is designed as a swappable module — replace `recognizeHandwriting()` with any stroke-based or OCR API.

### Alternative backends

- **Baidu/Tencent handwriting OCR** — render strokes to a tiny clean image, send to OCR API. Not stroke-native, but still 20× cheaper than Vision LLM.
- **Local ONNX model** — train/convert a small stroke-sequence classifier, run in-browser with ONNX Runtime Web. Zero API calls. (Future work.)

## LLM backend

Default: **DeepSeek** (`deepseek-v4-flash`). Any OpenAI-compatible API works — just change `config.js`.


## Credits

### [handwriting.js](https://github.com/ChenYuHo/handwriting.js) — Handwriting Recognition

By ChenYuHo (MIT License, 185 ★). Wraps Google Input Tools' handwriting recognition
API — sends stroke coordinate arrays `[[x1,x2,…], [y1,y2,…], []]` to Google's IME
endpoint and receives a ranked list of text candidates. Supports 40+ languages
including Chinese (Simplified & Traditional), Japanese, Korean, and English.

Riddle-- inlines its core recognition logic (~40 lines) directly into `index.html`,
stripping away the original Canvas wrapper in favor of a clean
`recognizeHandwriting(trace, language)` interface.

### [Monocraft](https://github.com/IdreesInc/Monocraft) — UI Font

By IdreesInc (SIL Open Font License 1.1, 11K ★). A monospaced programming font
inspired by the Minecraft typeface — 1500+ glyphs, each carefully redesigned from
the original pixel art for readability and spacing in a monospaced grid. Includes
programming ligatures and enchantment-table alphabet support.

Riddle-- uses the prebuilt `.ttf` from the project's dist directory as its UI
font — the status bar, settings panel, and log panel all render in Monocraft, a
cold pixel contrast to the flowing handwriting on the Canvas.

## License

MIT
