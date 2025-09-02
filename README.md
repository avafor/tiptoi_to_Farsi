# TipToi to Farsi
scripts to change your tiptoi files to Farsi

# Tiptoi Farsi Builder (end-to-end)

This project converts a Tiptoi `.gme` book into a Farsi-speaking version by:
1) Extracting original audio,
2) (Optionally) transcribing with Whisper,
3) (Optionally) translating to Farsi (via OpenAI),
4) Synthesizing Farsi audio,
5) Building a YAML referencing relative media,
6) Assembling a new `.gme`.

## Dependencies (macOS / Linux)

- `tttool` (>= 1.11) in your PATH  
- `ffmpeg`
- `espeak` or `espeak-ng` (we’ll detect at runtime)
- `whisper-cli` (from `whisper-cpp`) + a model (e.g. `models/ggml-base.bin`)
- Python 3.10+ with: `pip install pandas pyyaml`

> **Important:** When assembling, the YAML and the `.ogg` files must be in the **same folder** and the YAML must use **relative** paths like `media/7901.ogg`.

## Quick start

```bash
git clone <your-repo> tiptoi-farsi
cd tiptoi-farsi

# Put your original book here (or pass via ARG):
# ./Suchen_und_Entdecken-Meine_Welt.gme  (example)

# (Optional) copy .env settings then adjust if needed
cp .env.example .env

# End-to-end run (safe defaults; see variables below)
./e2e.sh \
  --gme ./Suchen_und_Entdecken-Meine_Welt.gme \
  --product-id 219 \
  --whisper-model ./models/ggml-base.bin \
  --skip-transcribe false \
  --skip-translate true  # set to false if you want OpenAI translation


SHELL := /bin/bash

GME ?= ./Suchen_und_Entdecken-Meine_Welt.gme
PRODUCT_ID ?= 219
WHISPER_MODEL ?= ./models/ggml-base.bin

.PHONY: deps extract transcribe csv translate tts yaml assemble sanity

deps:
	@scripts/check_deps.sh

extract:
	@bash scripts/extract_media.sh "$(GME)"

transcribe:
	@bash scripts/whisper_all.sh "$(WHISPER_MODEL)"

csv:
	@python3 scripts/make_transcript_csv.py

translate:
	@python3 scripts/translate_csv_openai.py

tts:
	@python3 scripts/synth_fa_audio.py

yaml:
	@python3 scripts/build_yaml.py --product-id $(PRODUCT_ID)

assemble:
	@pushd build >/dev/null && tttool assemble book_fa.yaml && mv *.gme book_fa.gme && popd >/dev/null
	@echo "✅ GME ready: build/book_fa.gme"

sanity:
	@python3 scripts/sanity.py
