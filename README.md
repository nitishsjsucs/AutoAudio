# AutoAudio

Turning text into listenable audio, three different ways. The repo holds three
independent pieces that share a goal rather than any code:

| Directory | What it is | Runs on |
|---|---|---|
| `index.html`, `Frontend/` | Browser TTS reader — loads a `.txt`, speaks it with the Web Speech API | Any modern browser, no build, no server |
| `epubToWav/` | EPUB → chunked CSV → Deepgram Aura → WAV segments | Python 3.12 |
| `Tranium/AutoAudioNitish/` | Fine-tuning a Piper TTS voice on AWS Trainium | trn1 instance |

They are not wired together, but they do chain by hand: `epubToWav` produces
exactly the `audio_file|text` pairs that the Trainium scripts train on.

---

## 1. Browser reader

A zero-dependency, fully offline `.txt` reader. Load a text file, and the
browser's own speech synthesis reads it aloud with the active sentence
highlighted and scrolled into view.

- **Offline and private.** The file never leaves the page; there is no network
  call and no backend.
- **Sentence parsing** with abbreviation handling (Mr., Dr., e.g.) so it does
  not stop mid-title.
- **Click any sentence** to jump playback there.
- **Session tokens** around `SpeechSynthesisUtterance` so a rapid seek does not
  get clobbered by a stale `onend` from the previous utterance.
- **Voice, rate, pitch, volume** controls, persisted in `localStorage`.
- **Auto-scroll** with drift detection to keep the active sentence centred.
- **Shortcuts:** `Space` play/pause, `Escape` stop, arrows/PgUp/PgDn to scroll.

### Files, and a wrinkle

There are **two near-identical copies** of the reader:

```
index.html                          landing page  -> links to the root reader
asretnoieasrnttoieinao.html         the reader    -> links back to index.html
Frontend/
  reader-landing.html               landing page  -> links to the Frontend reader
  asretnoieasrnttoieinao.html       the reader    -> links back to reader-landing.html
```

The **root pair is what ships** — `.nojekyll` plus `index.html` is what GitHub
Pages serves. `Frontend/` is an earlier, differently branded copy of the same
two pages ("Reader — Offline TTS TXT Player" rather than "AutoAudio"); the two
readers differ by under 100 bytes. Each pair links only within itself, so both
work, but `Frontend/` is redundant and is a good candidate for deletion once
you have confirmed nothing links to it externally.

The filename `asretnoieasrnttoieinao.html` is a keyboard mash. It is kept
because it is the published Pages URL, and renaming it breaks any existing
link.

### Running it

Open `index.html`, or serve the directory:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

Voice quality is whatever the OS provides. Chrome and Edge expose the best
voices; Firefox's list is thinner.

---

## 2. `epubToWav/` — EPUB to narrated audio

Two scripts, run in order, that turn an EPUB into a directory of WAV segments
narrated by Deepgram's Aura model.

**`epubToCsv.py`** extracts text from every `.epub` in `epubInput/`, normalises
whitespace, splits on sentence boundaries, then repacks sentences into chunks
of roughly ten seconds of speech (assuming 150 wpm, clamped to 8–12s). A short
trailing chunk is merged back into its predecessor rather than emitted alone.
Output is `csvOutput/<book>.csv` with `segment_index,text,word_count,
estimated_duration_seconds`.

**`csvToWav.py`** prompts you to pick a CSV, then POSTs each row's text to
Deepgram `/v1/speak` (model `aura-2-neptune-en`, linear16 WAV) and writes
`wavOutput/<book>_0001.wav` and so on. It is **resumable**: completed segment
indices are checkpointed to `wavOutput/.progress/<book>.json` after every
segment, and already-present WAV files are skipped, so a `Ctrl-C` or an API
error costs you nothing but the segment in flight.

### Setup

```bash
cd epubToWav
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

cp .env.example .env     # then put your Deepgram key in it
```

`.env` needs one variable:

```
DEEPGRAM_API_KEY=...     # https://console.deepgram.com/
```

Then:

```bash
python epubToCsv.py      # epubInput/*.epub -> csvOutput/*.csv
python csvToWav.py       # csvOutput/*.csv  -> wavOutput/*.wav
```

Both scripts use paths relative to the working directory, so run them from
inside `epubToWav/`.

`csvOutput/`, `wavOutput/` and `venv/` are gitignored — they are output and
environment, and synthesising a full book costs real Deepgram credits.

---

## 3. `Tranium/AutoAudioNitish/` — Piper voice fine-tuning on AWS Trainium

Scripts for training a [Piper](https://github.com/OHF-voice/piper1-gpl) TTS
voice on an AWS Trainium (`trn1`) instance, using PyTorch Neuron / XLA instead
of CUDA.

| File | Purpose |
|---|---|
| `prepare_dataset.py` | Converts a `segment_index,text` CSV plus an audio dir into Piper's `audio_file\|text` format; validates each WAV with librosa and resamples to 22.05 kHz |
| `test_dataset_format.py` | Dry run over `test.csv` — prints the filenames it would expect and the Piper lines it would emit |
| `train_piper_trainium.py` | The training entry point: moves the model to an XLA device, casts to BF16, wraps Piper's `PiperDataModule`/`PiperModel` in a Lightning `Trainer` |
| `train_piper.sh` | Two-pass driver — a one-epoch `COMPILE=1` pre-compilation pass, then real training |
| `train_piper_multi.sh` | Multi-core variant |
| `prepared_dataset.csv`, `test.csv`, `audio/` | Small sample dataset |
| `piper-docs/`, `TRAINING.md`, `*.rst` | Vendored upstream Piper and AWS Neuron reference docs |

### Setup

This only runs on Trainium hardware. On a `trn1` instance with the Neuron SDK:

```bash
pip config set global.extra-index-url https://pip.repos.neuron.amazonaws.com
pip install -r Tranium/AutoAudioNitish/requirements.txt

# Piper's training extra needs a source install and a Cython build:
git clone https://github.com/OHF-voice/piper1-gpl.git
cd piper1-gpl && pip install -e '.[train]' && ./build_monotonic_align.sh
```

Then edit the paths at the top of `train_piper.sh` and run it. Versions in
`requirements.txt` are deliberately unpinned: torch and torch-xla are dictated
by the Neuron SDK release on the instance.

`prepare_dataset.py` and `test_dataset_format.py` need only `pandas` and
`librosa` and run fine on a laptop.

---

## Security

`epubToWav/.env`, holding a **Deepgram API key**, was committed to this public
repository. It has been removed from the current tree and gitignored, but it
**remains in git history** and is retrievable by anyone who can clone the repo.

**Treat that key as compromised: revoke it in the Deepgram console and issue a
new one.** Deepgram bills per character synthesised, so also check the
account's usage for charges you did not make. Removing the file from HEAD does
not undo the exposure.

## Content note

`epubToWav/epubInput/` contains commercially published EPUBs, one of them
plainly a Library Genesis / Z-Library rip. They are test fixtures, but they are
copyrighted works in a public repository. They have been left in place because
removing them is the owner's call, not a cleanup decision — but they should
probably go, replaced by `test.epub` or another public-domain text.

## Licence

MIT is claimed but no LICENSE file is present; add one or drop the claim.
Note that Piper (`piper1-gpl`) is GPL-licensed, which constrains how any
derived voice-training work can be distributed.
