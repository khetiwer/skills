---
name: transcribe-audio
description: Transcribe (or translate) a local audio or video file with OpenAI Whisper. Use when the user wants to convert audio or video to text, says "transcribe this," "run Whisper on," "convert this recording," or shares a file path to an audio/video file.
---

# Transcribe Audio

Transcribes a local audio or video file using OpenAI Whisper on CPU. Handles setup checks, model selection, task type (transcribe or translate), output format selection, and output location.

---

## Step 1: Check dependencies

Run both checks before doing anything else.

**Check Whisper:**
```
python -m whisper --help
```

If this fails, stop and provide the Whisper install guidance at the bottom of this file. Do not proceed until the user confirms it is working.

**Check FFmpeg:**
```
ffmpeg -version
```

If this fails, provide the FFmpeg install guidance at the bottom of this file. FFmpeg is required for most audio formats.

---

## Step 2: Collect inputs

Ask for any missing inputs. You can ask multiple questions at once if several are missing.

**Input file** — if not provided, ask for the full path to the file.

**Task type** — ask whether the user wants to:
- `transcribe` — convert speech to text in its original language (default)
- `translate` — convert speech to English text regardless of the source language

**Model** — if not provided, ask which model to use:

| Model | Speed | Quality | Notes |
|---|---|---|---|
| `tiny` / `tiny.en` | Fastest | Roughest | Quick drafts only |
| `base` / `base.en` | Fast | Decent | Good balance |
| `small` / `small.en` | Moderate | Good | Recommended default |
| `medium` | Slow | Very good | When quality matters |
| `large` | Slowest | Best | High-resource |

For English audio, prefer `.en` variants. Steer away from `tiny` if the user mentions distant speakers, background noise, conference rooms, or interference.

Before running: warn the user that if this is the first time using the selected model, Whisper will download it automatically. Downloads range from ~75 MB (tiny) to ~1.5 GB (large) and the process may appear to hang during the download — it has not. Tell them to wait.

**Output directory** — if not provided, ask where to save the output files.

**Output format** — if not provided, ask which format(s) they want:

| Format | Use case |
|---|---|
| `txt` | Plain transcript, easiest to read |
| `srt` | Subtitle file for video players/editors |
| `vtt` | Web caption format |
| `tsv` | Timestamp rows, useful for spreadsheet analysis |
| `json` | Structured output for downstream processing |

If the user selects more than one format, use `--output_format all`. If they want only one, use that format name.

---

## Step 3: Confirm before running

Summarize the resolved inputs before running:

- Input file
- Task (transcribe or translate)
- Model
- Output directory
- Output format(s)

Wait for confirmation unless the user has already confirmed implicitly by answering all questions in a single message.

---

## Step 4: Run Whisper

**Single format:**
```
python -m whisper "<INPUT_FILE>" --model <MODEL> --task <TASK> --language English --fp16 False --output_dir "<OUTPUT_DIR>" --output_format <FORMAT> --verbose False
```

**Multiple formats:**
```
python -m whisper "<INPUT_FILE>" --model <MODEL> --task <TASK> --language English --fp16 False --output_dir "<OUTPUT_DIR>" --output_format all --verbose False
```

Rules:
- Quote all Windows paths.
- Always include `--fp16 False` — this forces CPU mode and prevents errors on machines without a GPU.
- Always include `--verbose False` — suppresses the live inline transcript during processing.
- Use `--task translate` when the user selected translate, `--task transcribe` otherwise.

---

## Step 5: Report output

After the run completes:
- Tell the user which files were created and their full paths
- Confirm the run completed successfully

Do not display the transcript inline unless the user explicitly asks.

---

## Install guidance

### Whisper missing

1. Verify Python and pip are installed:
   - `python --version`
   - `pip --version`
2. Install Whisper:
   - `python -m pip install -U openai-whisper`
3. Verify the install:
   - `python -m whisper --help`

### FFmpeg missing

1. Install via winget:
   - `winget install -e --id Gyan.FFmpeg`
2. Reopen your terminal after install
3. Verify:
   - `ffmpeg -version`

---

## Configuration notes

These defaults are intentional. Change them by editing this file directly.

- **Language**: hardcoded to `English` via `--language English`. To transcribe a different language, replace `English` with the target language (e.g., `Spanish`, `French`), or remove the flag entirely to let Whisper auto-detect.
- **CPU mode**: `--fp16 False` forces CPU processing. If you have a CUDA-capable GPU, remove this flag for significantly faster transcription.
- **Inline output**: `--verbose False` suppresses the live transcript during processing. Change to `--verbose True` to see output line by line as Whisper processes the file — useful for confirming the run is progressing on long files.
