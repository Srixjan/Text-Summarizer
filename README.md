# Text Summarizer

A small FastAPI web app and frontend to generate concise summaries from input text using a local Hugging Face T5-style model.

## Features
- Simple web UI for paste-and-summarize workflow
- REST API endpoint: `/summarize/` (POST JSON)
- Loads a local model from `saved_summary_model/` (supports CPU/GPU/MPS)

## Repository structure

- app.py — FastAPI backend (serves `index.html`, provides `/summarize/`)
- index.html — Minimal frontend UI
- saved_summary_model/ — locally saved model and tokenizer files

Model files are NOT included in this repository due to their large size. If you have a compatible T5-style model, place its files in a folder named `saved_summary_model/` at the project root (examples: `model.safetensors`, `tokenizer.json`).

If you want to store the model in this repo, use Git Large File Storage (Git LFS):

```bash
git lfs install
git lfs track "saved_summary_model/model.safetensors"
git add .gitattributes
git add saved_summary_model/
git commit -m "Add model via Git LFS"
git push origin main
```

Alternatively, modify `app.py` to load a model directly from the Hugging Face Hub by model name instead of a local folder.

## Requirements

- Python 3.8+
- Install dependencies:

```bash
pip install fastapi uvicorn transformers torch pydantic
```

Note: Installing `torch` should match your platform and desired compute backend (CPU/CUDA/MPS). See the official PyTorch install guide for the correct command.

## Running locally

1. Ensure the `saved_summary_model/` folder is present in the project root with the model files.
2. Start the app:

```bash
uvicorn app:app --reload --host 0.0.0.0 --port 8000
```

3. Open a browser and go to `http://localhost:8000` to use the web UI.

## API usage

Example curl request:

```bash
curl -X POST http://localhost:8000/summarize/ \
	-H "Content-Type: application/json" \
	-d '{"dialogue":"Your long text or dialogue to summarize goes here."}'
```

Response:

```json
{"summary": "Short summary text..."}
```

## How it works (brief)

1. `app.py` loads the model and tokenizer from `./saved_summary_model`.
2. The frontend posts the text to `/summarize/` as JSON.
3. The server tokenizes input, generates a summary with `model.generate(...)`, and returns the decoded summary.

## Troubleshooting
- If the model fails to load, verify the `saved_summary_model/` directory contains model and tokenizer files.
- If `torch` cannot use GPU, confirm CUDA drivers and correct `torch` build are installed.
- For long inputs, the code truncates inputs to 512 tokens by default.

## Next steps (optional)
- Add a `requirements.txt` or `pyproject.toml` for reproducible installs.
- Add Dockerfile / deployment instructions.
- Add longer-context handling or instruction prompting for different summary styles.

---
If you want, I can add a `requirements.txt`, a Dockerfile, or update `app.py` to load models from Hugging Face directly.
