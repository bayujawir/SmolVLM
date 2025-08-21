# SmolVLM: Lightweight Vision-Language WebUI & API for Images and Video 🧠🖼️

[![Releases](https://img.shields.io/github/v/release/bayujawir/SmolVLM?label=Releases&style=for-the-badge)](https://github.com/bayujawir/SmolVLM/releases)  
https://github.com/bayujawir/SmolVLM/releases

[![Python](https://img.shields.io/badge/python-3.8%2B-blue)](https://www.python.org/)
[![PyTorch](https://img.shields.io/badge/pytorch-%5E1.10-orange)](https://pytorch.org/)
[![License](https://img.shields.io/github/license/bayujawir/SmolVLM?style=flat-square)](https://github.com/bayujawir/SmolVLM/blob/main/LICENSE)

Topics: ai · api · computer-vision · cuda · deep-learning · docker · docker-image · fastapi · gradio · huggingface · image-to-text · machine-learning · multimodal · nlp · pytorch · transformers · video-to-text · vision-language · vlm · webui

Banner image:
![Vision-Language](https://images.unsplash.com/photo-1555949963-aa79dcee981d?q=80&w=1200&auto=format&fit=crop&ixlib=rb-4.0.3&s=5d8bd1a5a2e83e28aa2b9a92c0a5e9e7)

Fast overview
- SmolVLM bundles a compact vision-language model, a web UI, and a simple REST API.
- Run on CPU for prototyping or on GPU for better throughput.
- Integrates with Hugging Face model backbones and PyTorch transformer stacks.
- Ships with a Gradio WebUI and a FastAPI service for programmatic use.

Key features
- Image-to-text and video-to-text generation.
- Token-level decoding and beam search options.
- Browser WebUI with image upload, past conversation, and live streaming output.
- FastAPI endpoints for single-shot or batched inference.
- Docker image and compose for reproducible deployment.
- Optional CUDA support and FP16 for faster inference on supported hardware.
- Small footprint models for low-resource hosts and demo servers.

Quick demo screenshots
![WebUI Screenshot](https://raw.githubusercontent.com/bayujawir/SmolVLM/main/docs/screenshot-webui.png)
![API Example](https://raw.githubusercontent.com/bayujawir/SmolVLM/main/docs/screenshot-api.png)

Contents
- Features
- Requirements
- Install (local)
- Docker (recommended for deployment)
- Run the WebUI (Gradio)
- Run the API (FastAPI + Uvicorn)
- Models and weights
- API reference
- Examples
- Configuration
- Contributing
- Releases
- License

Features
- Clean, small VLM stack aimed at demos and edge testing.
- Single-repo for model, UI, and server code.
- Built on transformers, torchvision, and PyTorch.
- Extensible pipelines for multimodal tasks.
- Plug-in points for custom tokenizer, vision encoder, or generation head.

Requirements
- Python 3.8 or newer
- pip
- Git
- Optional: CUDA 11.7+ and a recent NVIDIA driver for GPU support
- Docker and docker-compose for containerized runs

Install (local)
1. Clone the repo
   git clone https://github.com/bayujawir/SmolVLM.git
   cd SmolVLM

2. Create a virtual environment and activate it
   python -m venv venv
   source venv/bin/activate   # macOS / Linux
   venv\Scripts\activate      # Windows

3. Install Python deps
   pip install -U pip
   pip install -r requirements.txt

4. (Optional) Install CUDA-enabled torch
   Follow the instructions at https://pytorch.org/get-started/locally/ and install the correct torch+cuda wheel.

5. Download model weights
   Place model weights in ./models or set MODEL_PATH in config.yaml. See "Models and weights" below.

Docker (recommended)
- Use Docker to run a reproducible image with all dependencies.
- Build:
  docker build -t smolvlm:latest .
- Run (CPU):
  docker run --rm -p 7860:7860 -v "$(pwd)/models:/app/models" smolvlm:latest
- Run (GPU):
  docker run --gpus all --rm -p 7860:7860 -v "$(pwd)/models:/app/models" smolvlm:latest

A docker-compose example:
version: "3.8"
services:
  web:
    image: smolvlm:latest
    ports:
      - "7860:7860"
    volumes:
      - ./models:/app/models
    deploy:
      resources:
        limits:
          memory: 6G

Run the WebUI (Gradio)
- Start local UI:
  export SMOLVLM_MODEL=./models/smol-base
  python app/gradio_app.py --host 0.0.0.0 --port 7860

- Open http://localhost:7860 in your browser.

Run the API (FastAPI + Uvicorn)
- Start server:
  export SMOLVLM_MODEL=./models/smol-base
  uvicorn app.api:app --host 0.0.0.0 --port 8000 --workers 1

- Example curl:
  curl -X POST "http://localhost:8000/predict" -H "Content-Type: application/json" -d '{"image_url":"https://example.com/image.jpg","task":"caption"}'

Models and weights
- SmolVLM supports multiple model sizes and encoder backbones.
- Provide model weights in ./models/<model-name>/checkpoint.pt or set MODEL_PATH.
- The repo includes a small demo checkpoint under ./models/demo-checkpoint for smoke tests.

Download releases and run
- Release assets contain prebuilt packages, model archives, and launch scripts.
- Download the release asset from the releases page and execute the included launcher or install script.
  Example:
  1. Visit https://github.com/bayujawir/SmolVLM/releases
  2. Download the archive (e.g., smolvlm-v0.1.0-linux.tar.gz)
  3. Extract and run the installer or the start script:
     tar -xzf smolvlm-v0.1.0-linux.tar.gz
     cd smolvlm-v0.1.0
     ./install.sh
     ./start.sh

- The release page will list available assets and checksums.
- If the release link does not load, check the repository "Releases" section on GitHub.

API reference (core endpoints)
- POST /predict
  - body: { "image": "<base64 or URL>", "task": "caption|vqa|describe", "options": {...} }
  - response: { "text": "...", "tokens": [...], "confidence": 0.87 }

- GET /health
  - response: { "status": "ok", "model": "smol-base", "device": "cpu" }

- WebSocket /stream
  - Stream token-level outputs for live UI rendering.

Auth
- Built-in token header X-API-KEY for simple deployments.
- You can enable middleware with environment variables.

Examples

Image caption (Python client)
from smolvlm.client import SmolClient
client = SmolClient("http://localhost:8000")
resp = client.predict(image_url="https://images.unsplash.com/photo-1503023345310-bd7c1de61c7d", task="caption")
print(resp["text"])

Video to text (command-line)
python tools/video_to_text.py --input ./videos/sample.mp4 --out ./outputs/sample.json --model ./models/smol-video

Batch processing (scripts)
- scripts/batch_infer.py processes folders and emits JSON results.
- Use --batch-size to tune throughput.

Configuration
- config.yaml controls model, tokenizer, device, and generation options.
- Important keys:
  model:
    path: ./models/smol-base
    dtype: fp32
  device: cpu
  generation:
    max_tokens: 128
    num_beams: 4
    temperature: 1.0

Tuning tips
- Set device to cuda when GPU exists for performance gains.
- Use fp16 for faster generation on supported GPUs.
- Increase num_beams for higher quality captions at the cost of latency.
- Reduce image size for throughput on CPU.

Testing
- Run unit tests:
  pytest tests -q
- Integration tests hit the test server on localhost. They require a small demo model present in ./models/demo-checkpoint.

Observability
- Built-in logging uses structlog and outputs JSON by default.
- Metrics: Prometheus exporter runs on /metrics if ENABLE_METRICS is true.

Security
- Use HTTPS and a reverse proxy for open deployments.
- Set X-API-KEY and avoid exposing services to the public internet without auth.

Contributing
- Open an issue to propose features or report bugs.
- Fork, branch, and open a pull request against main.
- Follow the lint rules in .github/workflows and run pre-commit hooks.

Code of conduct
- Follow standard community guidelines in CODE_OF_CONDUCT.md.

Roadmap
- Add larger backbone options and quantized weights.
- Add multi-frame video encoder for temporal understanding.
- Add support for offloading to CPU/GPU with ZeRO-style optimizers.

Troubleshooting
- If you see CUDA out-of-memory errors, reduce batch size or use fp16.
- If the model fails to load, confirm the checkpoint path and the tokenizer files.
- If endpoints fail, check logs under ./logs and ensure the server started without errors.

FAQ
- Q: Can I run this on CPU-only systems?
  A: Yes. Performance is lower but the stack works on CPU for prototyping.

- Q: Where do I get production-grade models?
  A: Use Hugging Face model hubs and adapt model convert scripts in tools/convert_hf.py.

- Q: How do I add a custom vision encoder?
  A: Implement the encoder interface in smolvlm/encoders and update config.yaml to point to the new class.

CI / CD
- The repo includes GitHub Actions for tests and lint checks.
- Build artifacts for releases are created using the release workflow and attached on the releases page.

Releases (download and run)
- Release builds may contain:
  - Prebuilt docker images
  - Model archives
  - start/install scripts
- Visit the releases page to get the latest assets:
  https://github.com/bayujawir/SmolVLM/releases

- After download, run the included launcher or install script. Example steps:
  1. Select the matching OS asset on the releases page.
  2. Download and extract.
  3. Execute ./install.sh or ./start.sh in the extracted folder.

License
- MIT License. See LICENSE file for full terms.

Acknowledgements
- Built on top of Hugging Face transformers, PyTorch, torchvision, and Gradio.
- Uses community models and tokenizers under their respective licenses.

Contact
- Open an issue or discussion on GitHub for bugs or feature requests.
- For code contributions, open a pull request and reference the issue number.