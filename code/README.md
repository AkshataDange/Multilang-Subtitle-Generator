# Multilang Subtitle Generator

A command-line pipeline that takes a video, transcribes the speech, translates the subtitles into a target language, and burns the translated subtitles back into the video — with correct font rendering for non-Latin scripts (Devanagari, Bengali, Tamil, Malayalam, Telugu, Kannada, and more).

## How it works

1. **Transcribe** — [OpenAI Whisper](https://github.com/openai/whisper) extracts the audio and transcribes it to timestamped segments.
2. **Translate** — depending on the source/target language pair, translation runs through one of two model families:
   - [AI4Bharat IndicTrans2](https://github.com/AI4Bharat/IndicTrans2) for English ↔ Indic-language pairs (Hindi, Marathi, Bengali, Tamil, Malayalam, Telugu, Kannada) and Indic ↔ Indic
   - [Facebook M2M100](https://huggingface.co/facebook/m2m100_418M) for English ↔ Malay
   - For Indic ↔ Malay, translation pivots through English (two hops) since no direct model pairing exists
3. **Render** — translated subtitles are converted to `.ass` format and hard-burned into the output video via `ffmpeg`, using the correct Noto Sans variant for the target script.

## Tech stack

Python, OpenAI Whisper, Hugging Face Transformers, AI4Bharat IndicTrans2, Facebook M2M100, ffmpeg-python.

## Setup

```bash
pip install -r requirements.txt

# IndicTransToolkit is required for IndicTrans2 pre/post-processing
git clone https://github.com/VarunGumma/IndicTransToolkit
cd IndicTransToolkit
pip install --editable .

sudo apt-get install ffmpeg
```

Python 3.10 is required.

### Fonts

Subtitle rendering needs the right font installed per script (e.g. Noto Sans Devanagari for Hindi/Marathi, Noto Sans Tamil for Tamil, etc.) so translated text renders correctly instead of showing tofu boxes.

1. Download the needed [Noto Sans](https://fonts.google.com/noto) variants for your target languages.
2. Place them in a `fonts/` folder next to `main.py` (this is the default `FONTS_DIR`), or set the `FONTS_DIR` environment variable to point wherever you keep them.

## Usage

```bash
python main.py
```

You'll be prompted for:
- The input video path
- Source language code (e.g. `en`, `hi`, `ms`)
- Target language code (e.g. `hi`, `en`, `ms`)

Supported language codes: `en` (English), `hi` (Hindi), `mr` (Marathi), `bn` (Bengali), `ta` (Tamil), `ml` (Malayalam), `te` (Telugu), `kn` (Kannada), `ms` (Malay).

Output: `output_subtitled.mp4` in the working directory, along with the intermediate `original.srt` / `translated.srt` / `translated.ass` files.

## Notes

- Translation runs on GPU automatically if `torch.cuda.is_available()`, otherwise falls back to CPU.
- This is a command-line proof of concept, not a production service — there's no batching UI, job queue, or error recovery beyond what's in the script.
