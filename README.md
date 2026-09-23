# Local AI Stack for macOS

A fully local, private AI setup running entirely on-device — no cloud APIs, no external dependencies for inference. Chat with LLMs and generate images offline using your own hardware.

## Stack

- **[Ollama](https://ollama.ai)** — runs LLMs locally (Llama 3.1, Qwen2.5-Coder, etc.)
- **[Open WebUI](https://github.com/open-webui/open-webui)** — ChatGPT-style web interface, connected to Ollama via Docker
- **[Stable Diffusion (Automatic1111)](https://github.com/AUTOMATIC1111/stable-diffusion-webui)** — local image generation, integrated directly into chat responses

## Features

- Chat with multiple local LLMs through a clean web interface
- Generate images from chat prompts, powered by a locally-run Stable Diffusion instance
- Fully offline after setup — no data leaves your machine
- Swap models freely (general chat, coding-focused, etc.) via Ollama

## Requirements

- macOS (Apple Silicon or Intel)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [Homebrew](https://brew.sh)
- ~15GB+ free disk space (more if you add additional models)

## Setup

### 1. Install Ollama

Download from [ollama.ai](https://ollama.ai) and install the macOS app, or via terminal:

```bash
ollama pull llama3.1
```

### 2. Install Docker Desktop

Download from [docker.com](https://www.docker.com/products/docker-desktop/) and launch it once before continuing.

### 3. Deploy Open WebUI

```bash
docker run -d -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

Open `http://localhost:3000` and create your admin account (first account created is automatically admin).

### 4. Install Stable Diffusion (Automatic1111)

```bash
brew install pyenv git
pyenv install 3.10
pyenv global 3.10

mkdir ~/stablediff && cd ~/stablediff
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git .
chmod +x webui.sh

# Note: as of writing, Stability-AI's original repo dependency is unavailable.
# Use this override before launching:
export STABLE_DIFFUSION_REPO="https://github.com/w-e-w/stablediffusion.git"

./webui.sh --listen --api
```

Wait for it to finish installing and downloading the base model (~4GB on first run). Once you see `Running on local URL: http://0.0.0.0:7860`, it's ready.

### 5. Connect Stable Diffusion to Open WebUI

In Open WebUI: **Settings → Images**
- Set **Image Generation Engine** to `Automatic1111`
- Set **Base URL** to `http://host.docker.internal:7860`
- Enable **Image Generation** and save

## Usage

- Open `http://localhost:3000` in your browser
- Pick a model from the dropdown (e.g. `llama3.1:latest` for general chat, `qwen2.5-coder:latest` for coding)
- Click the image icon on any response to generate a matching image via Stable Diffusion

## Notes

- Older Ollama models (e.g. `llama2`, `codegemma`) predate Ollama's tool-calling support and may error with newer versions of Open WebUI. Use current models like `llama3.1` or `qwen2.5-coder` instead.
- Without a dedicated GPU, Stable Diffusion runs on CPU and will be slower than a desktop rig with dedicated graphics cards — expect generation times of 20–60+ seconds per image depending on your Mac.

## Credits

Built by following and adapting [NetworkChuck's local AI server tutorial](https://www.youtube.com/@NetworkChuck) for macOS, with additional fixes for current dependency and environment issues.
