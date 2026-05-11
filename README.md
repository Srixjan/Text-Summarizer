# Text Summarizer

A lightweight FastAPI-based web application for automatic text summarization using Hugging Face Transformers and the T5 model architecture. Provides both a user-friendly web interface and a RESTful API endpoint.

## 🎯 Features

- **Web UI**: Clean, responsive browser interface for quick text summarization
- **REST API**: JSON-based API endpoint for programmatic access
- **Multi-device Support**: Automatic detection and usage of GPU/MPS/CPU
- **Local Model Loading**: Runs locally using a T5-style transformer model
- **Text Preprocessing**: Automatic cleaning and normalization of input text
- **Fast Inference**: Beam search decoding with configurable output length

## 📁 Repository Structure

```
Text-Summarizer/
├── app.py                       # FastAPI backend application
├── index.html                   # Web UI (served at root)
├── text_summarizer_minor_project.ipynb  # Development notebook
├── README.md                    # This file
├── .gitattributes              # Git LFS configuration
├── __pycache__/                # Python cache (auto-generated)
└── saved_summary_model/        # Model directory (NOT included - see setup)
    ├── config.json
    ├── generation_config.json
    ├── model.safetensors       # (230+ MB - stored via Git LFS)
    ├── tokenizer.json
    └── tokenizer_config.json
```

## ⚙️ Requirements

- **Python**: 3.8 or higher
- **Operating System**: Windows, macOS, or Linux
- **Disk Space**: ~500 MB (for model files)
- **RAM**: 4 GB minimum (8 GB recommended)
- **GPU** (optional): NVIDIA CUDA 11.8+ or Apple MPS for faster inference

## 🚀 Quick Start

### 1. Clone & Setup

```bash
git clone https://github.com/Srixjan/Text-Summarizer.git
cd Text-Summarizer
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

Or manually:

```bash
pip install fastapi uvicorn transformers torch pydantic
```

**Note on PyTorch**: The `torch` package should match your system. For GPU support, visit [PyTorch.org](https://pytorch.org/get-started/locally/) and select your platform/CUDA version.

### 3. Set Up the Model

Model files **are NOT included** in this repository because they exceed 100 MB. Choose one method below:

#### Option A: Use a Pre-downloaded Model

If you have a compatible T5-style model folder, place it at the project root:

```bash
# Copy your model folder
cp -r /path/to/your/t5-model ./saved_summary_model
```

Ensure the folder contains: `model.safetensors`, `tokenizer.json`, `config.json`, and `generation_config.json`.

#### Option B: Download from Hugging Face Hub (Recommended)

1. Create the model directory:
   ```bash
   mkdir saved_summary_model
   ```

2. Download a compatible model using Python:
   ```python
   from transformers import T5ForConditionalGeneration, T5Tokenizer
   
   model_name = "t5-small"  # or "t5-base", "t5-large", etc.
   
   model = T5ForConditionalGeneration.from_pretrained(model_name)
   tokenizer = T5Tokenizer.from_pretrained(model_name)
   
   model.save_pretrained("./saved_summary_model")
   tokenizer.save_pretrained("./saved_summary_model")
   
   print("Model downloaded and saved!")
   ```

Available T5 models: `t5-small`, `t5-base`, `t5-large`, `t5-3b`, `t5-11b` (larger = slower but more accurate).

#### Option C: Store Model via Git LFS (Advanced)

If you want to version control the model in this repo:

```bash
git lfs install
git lfs track "saved_summary_model/model.safetensors"
git add .gitattributes
git add saved_summary_model/
git commit -m "Add model via Git LFS"
git push origin main
```

### 4. Run the Application

```bash
uvicorn app:app --reload --host 0.0.0.0 --port 8000
```

Expected output:
```
INFO:     Uvicorn running on http://0.0.0.0:8000
Press CTRL+C to quit
```

### 5. Access the Application

- **Web UI**: Open browser → `http://localhost:8000`
- **API Docs**: `http://localhost:8000/docs` (Swagger UI)

## 🔌 API Documentation

### Endpoint: `/summarize/`

**Method**: `POST`

**Request Body**:
```json
{
  "dialogue": "Your long text or article to be summarized goes here. This can be multiple paragraphs or sentences."
}
```

**Response**:
```json
{
  "summary": "A concise summary of the provided text."
}
```

### Example Requests

**Using curl**:
```bash
curl -X POST http://localhost:8000/summarize/ \
  -H "Content-Type: application/json" \
  -d '{"dialogue":"Machine learning is a subset of artificial intelligence that enables systems to learn and improve from experience without being explicitly programmed. It focuses on developing algorithms and statistical models."}'
```

**Using Python**:
```python
import requests

url = "http://localhost:8000/summarize/"
data = {"dialogue": "Your text here..."}
response = requests.post(url, json=data)
print(response.json()["summary"])
```

**Using JavaScript/Fetch**:
```javascript
fetch("http://localhost:8000/summarize/", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ dialogue: "Your text here..." })
})
.then(res => res.json())
.then(data => console.log(data.summary));
```

## 🔍 How It Works

1. **Input Cleaning**: Text is normalized (lowercase, extra spaces removed, HTML tags stripped).
2. **Tokenization**: Text is converted to token IDs with a max length of 512 tokens.
3. **Model Inference**: The T5 model generates summary tokens using beam search (4 beams, early stopping).
4. **Decoding**: Generated tokens are converted back to readable text.
5. **Output**: Summary is returned via API or displayed in the web UI.

### Configuration Parameters (in `app.py`)

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `max_length` (tokenize) | 512 | Maximum input length in tokens |
| `max_length` (generate) | 150 | Maximum summary length in tokens |
| `num_beams` | 4 | Beam width for decoding |
| `early_stopping` | True | Stop early if best sequence found |

Modify these in `app.py` to adjust summarization behavior.

## 🛠️ Troubleshooting

### Model Files Not Found
```
FileNotFoundError: saved_summary_model/model.safetensors
```
**Solution**: Verify the model folder exists and contains required files. Use Option B above to download from Hugging Face.

### Out of Memory (OOM) Error
```
RuntimeError: CUDA out of memory
```
**Solutions**:
- Use a smaller model: `t5-small` instead of `t5-large`
- Reduce `max_length` in `app.py` (line ~50)
- Close other applications to free RAM
- Run on CPU by removing GPU detection in `app.py`

### GPU Not Detected
```
torch: CUDA device not available
```
**Solutions**:
- Install CUDA: Visit [NVIDIA CUDA Toolkit](https://developer.nvidia.com/cuda-toolkit)
- Reinstall PyTorch with CUDA support:
  ```bash
  pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
  ```
- Or use CPU (slower but always works)

### Port Already in Use
```
ERROR: Address already in use (port 8000 in use)
```
**Solution**: Use a different port:
```bash
uvicorn app:app --reload --host 0.0.0.0 --port 8001
```

### Very Slow Inference
- You may be using CPU; consider switching to GPU if available
- Try smaller model (`t5-small` vs `t5-large`)
- Reduce beam width from 4 to 2 in `app.py` line ~55

## 📊 Performance Tips

- **First run**: Slow (~10-30s) due to model loading
- **Subsequent runs**: Fast (~2-5s for 500 words on CPU, <1s on GPU)
- **GPU**: 5-10x faster than CPU
- **Input length**: Longer texts take proportionally longer

## 🔐 Security Notes

- The API currently has no authentication; for production, add:
  - API key validation
  - Rate limiting
  - Input validation
  - HTTPS/SSL

## 📚 Project Architecture

```
User Request
    ↓
[Web UI / REST API] (index.html / app.py:POST /summarize/)
    ↓
[Text Cleaning] (regex preprocessing)
    ↓
[Tokenization] (T5Tokenizer.encode)
    ↓
[Model Inference] (T5ForConditionalGeneration.generate)
    ↓
[Decoding] (tokenizer.decode)
    ↓
Response (JSON / HTML)
```

## 📝 License

This project is provided as-is for educational purposes.

## 🙏 Credits

- Built with [FastAPI](https://fastapi.tiangolo.com/) and [Hugging Face Transformers](https://huggingface.co/transformers/)
- UI styled with custom CSS
- Model weights from [Hugging Face Hub](https://huggingface.co/models)

---

**Questions or Issues?** Feel free to open an issue or fork the repository to contribute improvements!
