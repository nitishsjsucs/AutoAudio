# AutoAudio

**Natural Offline Text-to-Speech Reader with Modern UI**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![HTML5](https://img.shields.io/badge/HTML5-E34F26?logo=html5&logoColor=white)](https://developer.mozilla.org/en-US/docs/Web/HTML)
[![CSS3](https://img.shields.io/badge/CSS3-1572B6?logo=css3&logoColor=white)](https://developer.mozilla.org/en-US/docs/Web/CSS)
[![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?logo=javascript&logoColor=black)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)

AutoAudio is a privacy-first, fully offline text-to-speech reader that transforms plain text files into an immersive listening experience. Featuring a Spotify-inspired glassmorphism UI, sentence-level highlighting, click-to-jump navigation, and advanced playback controls — all running entirely in the browser with zero external dependencies.

---

## Key Features

- **100% Offline & Private** — No internet connection required. Files never leave your device.
- **Natural Voice Selection** — Choose from all system-installed voices with real-time preview.
- **Sentence-Level Highlighting** — Active sentence is visually highlighted as audio plays.
- **Click-to-Jump Navigation** — Click any sentence to start playback from that point.
- **Spotify-Style Player Bar** — Progress bar, play/pause, volume, and track info.
- **Adjustable Playback** — Fine-tune speed (rate), pitch, and volume with slider controls.
- **Smart Auto-Scroll** — Keeps the active sentence centered in the viewport during playback.
- **Keyboard Shortcuts** — `Space` for play/pause, `Escape` to stop, arrow keys for scrolling.
- **Responsive Design** — Works seamlessly on desktop and mobile with touch/swipe support.
- **Modern Glassmorphism UI** — Dark theme with glass effects, smooth animations, and noise texture overlays.

---

## Screenshots

### Landing Page
The landing page introduces AutoAudio with a modern hero section and feature cards.

### Reader Interface
The main reader features a sticky player bar, sentence highlighting, voice/rate/pitch controls, and a progress scrubber.

---

## Architecture

```
AutoAudio/
├── index.html                          # Landing page with feature showcase
├── Frontend/
│   ├── asretnoieasrnttoieinao.html     # Main TTS reader application
│   └── reader-landing.html             # Alternative reader landing page
```

### Core Components

| Component | Description |
|-----------|-------------|
| **Text Parser** | Splits raw text into paragraphs and sentences using regex-based boundary detection with abbreviation handling |
| **Speech Engine** | Wraps the Web Speech API (`SpeechSynthesisUtterance`) with session tokens to prevent stale event handlers |
| **UI Controller** | Manages play/pause/stop state, progress bar, track info, and active sentence highlighting |
| **Auto-Scroll System** | Interval-based centering with drift detection and smooth scrolling |
| **Voice Manager** | Enumerates system voices, persists preferences in `localStorage`, and favors Google/Microsoft voices |

---

## How It Works

1. **Load a `.txt` file** via the file picker at the top of the reader.
2. **Text is parsed** into paragraphs and sentences using intelligent boundary detection (handles abbreviations like Mr., Dr., etc.).
3. **Each sentence becomes a clickable span** rendered in the document view.
4. **Press Play** or click any sentence to begin TTS playback from that point.
5. **The active sentence highlights** in real-time with auto-scroll keeping it centered.
6. **Adjust voice, rate, pitch, and volume** on the fly — changes apply immediately.

---

## Technology Stack

- **HTML5 / CSS3 / Vanilla JavaScript** — Zero dependencies, zero build step
- **Web Speech API** — Browser-native text-to-speech synthesis
- **CSS Glassmorphism** — Backdrop blur, glass borders, noise texture overlays
- **LocalStorage** — Persistent voice/rate/pitch/volume preferences
- **Google Fonts (Inter, Instrument Serif)** — Clean, modern typography

---

## Getting Started

### Prerequisites
- A modern web browser (Chrome, Edge, Firefox, Safari) with Web Speech API support

### Usage

1. Clone the repository:
   ```bash
   git clone https://github.com/nitishsjsucs/AutoAudio.git
   cd AutoAudio
   ```

2. Open `index.html` in your browser, or serve locally:
   ```bash
   # Python
   python -m http.server 8000

   # Node.js
   npx serve .
   ```

3. Click **Launch AutoAudio** to open the reader.
4. Load any `.txt` file and start listening.

---

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `Space` | Toggle Play / Pause |
| `Escape` | Stop playback |
| `Arrow Up/Down` | Scroll content |
| `Page Up/Down` | Fast scroll |

---

## My Contributions

- **Core TTS Engine** — Designed and implemented the speech synthesis pipeline with session token management, preventing stale event callbacks during rapid seek/skip operations.
- **Sentence Parsing System** — Built the regex-based text parser with intelligent abbreviation handling (Mr., Dr., e.g., etc.) and Unicode-aware sentence boundary detection.
- **Spotify-Style Player UI** — Designed and developed the glassmorphism player bar with progress scrubbing, volume control, and real-time track info display.
- **Auto-Scroll System** — Implemented the interval-based drift detection and smooth centering algorithm to keep the active sentence in view.
- **Landing Page Design** — Created the modern landing page with feature cards, gradient backgrounds, and noise texture overlays.

---

## License

MIT License — see [LICENSE](LICENSE) for details.
