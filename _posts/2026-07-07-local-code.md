---
layout: post
title:  "A code agent with a local uncensored LLM"
lang: en
tags: [llm, agent, local]
category: tutorial
published: true
---

I believe that neutering LLMs with guardrails is detrimental to the model's performance, hurts the model usability in a broader sense and makes the model pretty much useless for certain specific tasks such as cybersecurity. The simplest thing to somewhat mitigate this is to run an uncensored model locally. Running locally also gives us more capabilities and more control over the LLM. Another aspect is that running inference locally keeps the data and metadata private. So I stringed together an instance of llama.cpp running an uncensored model and a coding harness. The harness and the inference engine run in docker containers, that allows for a certain level of isolation from the host system, and also flexibility of deployment. The harness is running in a project directory and can not access anything outside of it. The image for inference container can be build on a dev machine and then transferred to and deployed on a machine without internet access. The inference engine can be used by other software as well, like Continue in VSCode or open-webui. The following is my runbook.

## my setup

Ubuntu VM for the dev environment, [Debian LXC with an NVIDIA P40 and docker]({% post_url 2024-12-7-gpu-home-server %}) for running the inference. Also would need a Python virtual environment (through conda or venv) to download the model using hf cli.

## the model

I used [modelheretic.com](https://modelheretic.com/) to find an uncensored model that would fit my hardware, I decided to go with [Huihui-Qwen3.6-35B-A3B-Claude-4.7-Opus-abliterated-ggml-model-Q4_K](https://huggingface.co/huihui-ai/Huihui-Qwen3.6-35B-A3B-Claude-4.7-Opus-abliterated-MTP-GGUF) and [Qwen3.6-27B-Heretic-Uncensored-FINETUNE-NEO-CODE-Di-IMatrix-MAX-GGUF](https://huggingface.co/DavidAU/Qwen3.6-27B-Heretic-Uncensored-FINETUNE-NEO-CODE-Di-IMatrix-MAX-GGUF)  which are already in GGUF format. there are also regular models fine tuned for cyber, like the [formerly known as the whiterabbit](https://huggingface.co/DeepHat/DeepHat-V1-7B), 
[CyberSecQwen-4B](https://huggingface.co/lablab-ai-amd-developer-hackathon/CyberSecQwen-4B), [Foundation-Sec-8B-Reasoning](https://huggingface.co/fdtn-ai/Foundation-Sec-8B-Reasoning), [VulnLLM-R-7B](https://huggingface.co/Virtue-AI-HUB/VulnLLM-R-7B) etc. there is [a model fine tuned on CVEs](https://huggingface.co/build-small-hackathon/OpenMythos), but it needs more VRAM. and [there are other uncensored models fine tuned for cyber](https://huggingface.co/models?search=abliterated+cyber). also, safetensors must be [converted to GGUF]({% post_url 2025-02-14-hf-to-ollama %}) with appropriate quantization.

llama.cpp can download models from Huggingface, but I would like to separate these processes and copy the model weights manually.

also need to [generate new token on Huggingface](https://huggingface.co/settings/tokens/new?tokenType=read) to bypass the download speed limits.

on the inference machine (or any other machine, but move the `models` directory to the inference machine afterwards) create a python virtual environment to download the model weights. with conda it would be like
```bash
conda create --name hf_env python=3.10 -y
conda activate hf_env
```

and with venv it would be like this:
```bash
python -m venv .venv
source .venv/bin/activate
```

download the model
```bash
mkdir -p ./models
pip install "huggingface_hub[cli]"
pip install "httpx[socks]"
 HF_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxx hf download huihui-ai/Huihui-Qwen3.6-35B-A3B-Claude-4.7-Opus-abliterated-MTP-GGUF \
  Huihui-Qwen3.6-35B-A3B-Claude-4.7-Opus-abliterated-ggml-model-Q4_K.gguf \
  --include "*Q4_K*.gguf" --local-dir ./models --repo-type model --revision main

 HF_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxx hf download DavidAU/Qwen3.6-27B-Heretic-Uncensored-FINETUNE-NEO-CODE-Di-IMatrix-MAX-GGUF \
  Qwen3.6-27B-NEO-CODE-HERE-2T-OT-Q4_K_M.gguf \
  --local-dir ./models --repo-type model --revision main
```



## the inference engine

`git clone https://github.com/ggml-org/llama.cpp.git` . sure there are simpler methods to run llama.cpp, but I must compile it from the source code myself, i need convenient config files, more control, more flexibility and isolation, so I went with a setup like this. the complexity is compensated by the ease and flexibility of deployment. this setup can be modified for being completely self sustained, with the bind mounts removed and the weights baked into the image. this kind of image can be built locally and then transferred to a remote machine like `docker save llama-cpp:local | ssh $USER@inference docker load` without internet access and the container deployment controlled by the remote docker daemon with `DOCKER_HOST="ssh://$USER@inference" docker ...`


`nano .env` (modify LLAMA_PORT, LLAMA_MODEL_PATH, CMAKE_CUDA_ARCHITECTURES and LLAMA_CTX_SIZE):

```
# Host port to expose llama-server on.
LLAMA_PORT=8081

# Host directory containing GGUF model weights. This is mounted as /models:ro.
LLAMA_MODELS_DIR=./models

# Path inside the container, not the host path.
LLAMA_MODEL_PATH=/models/Huihui-Qwen3.6-35B-A3B-Claude-4.7-Opus-abliterated-ggml-model-Q4_K.gguf

# Build settings.
CUDA_VERSION=12.6.3
UBUNTU_VERSION=24.04
LLAMA_CPP_REF=master
# Tesla P40: 61; RTX 4090: 89; RTX 3090/3080: 86; A100: 80.
CMAKE_CUDA_ARCHITECTURES=61

# Runtime settings. These defaults favor stability on a Tesla P40 with 24 GB
# VRAM and 32 GB system RAM.
LLAMA_N_GPU_LAYERS=auto
LLAMA_CTX_SIZE=102400
LLAMA_FLASH_ATTN=off
LLAMA_N_PARALLEL=1
LLAMA_BATCH_SIZE=256
LLAMA_UBATCH_SIZE=128
LLAMA_MMAP=false
#LLAMA_SPEC_TYPE=draft-mtp
#LLAMA_SPEC_DRAFT_N_MAX=6

# Refresh persisted /llama-bin from the image on each start.
LLAMA_SYNC_BINARIES=1
LLAMA_SHM_SIZE=32g

```

`nano docker-compose.yml`, no need to edit anything here:
```
services:
  llama-cpp:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        CUDA_VERSION: "${CUDA_VERSION}"
        UBUNTU_VERSION: "${UBUNTU_VERSION}"
        LLAMA_CPP_REPO: "https://github.com/ggml-org/llama.cpp.git"
        LLAMA_CPP_REF: "${LLAMA_CPP_REF}"
        # NVIDIA Tesla P40 is Pascal / sm_61. Override for other GPUs.
        CMAKE_CUDA_ARCHITECTURES: "${CMAKE_CUDA_ARCHITECTURES}"
    image: local/llama.cpp:cuda${CUDA_VERSION}
    container_name: llama-cpp
    volumes:
      - llama-bin:/llama-bin
      - ${LLAMA_MODELS_DIR}:/models:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "127.0.0.1:${LLAMA_PORT}:8080"
    environment:
      LLAMA_SYNC_BINARIES: "${LLAMA_SYNC_BINARIES}"

      # Model and server binding. LLAMA_MODEL_PATH is the path inside the container.
      LLAMA_ARG_MODEL: "${LLAMA_MODEL_PATH}"
      LLAMA_ARG_HOST: "0.0.0.0"
      LLAMA_ARG_PORT: "8080"

      # Conservative Tesla P40 / 24 GB VRAM defaults for large CUDA models.
      # Increase these only after a short OpenAI-compatible completion is stable.
      LLAMA_ARG_N_GPU_LAYERS: "${LLAMA_N_GPU_LAYERS}"
      LLAMA_ARG_CTX_SIZE: "${LLAMA_CTX_SIZE}"
      LLAMA_ARG_FLASH_ATTN: "${LLAMA_FLASH_ATTN}"
      LLAMA_ARG_N_PARALLEL: "${LLAMA_N_PARALLEL}"
      LLAMA_ARG_BATCH: "${LLAMA_BATCH_SIZE}"
      LLAMA_ARG_UBATCH: "${LLAMA_UBATCH_SIZE}"
      LLAMA_ARG_MMAP: "${LLAMA_MMAP}"

      # Speculative decoding can trigger CUDA backend crashes on some
      # model/build/driver combinations. Set LLAMA_SPEC_TYPE=draft-mtp to opt in.
      #LLAMA_ARG_SPEC_TYPE: "${LLAMA_SPEC_TYPE}"
      #LLAMA_ARG_SPEC_DRAFT_N_MAX: "${LLAMA_SPEC_DRAFT_N_MAX}"

      # Optional diagnostics.
      NVIDIA_VISIBLE_DEVICES: "${NVIDIA_VISIBLE_DEVICES:-all}"
      NVIDIA_DRIVER_CAPABILITIES: "${NVIDIA_DRIVER_CAPABILITIES:-compute,utility}"
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
    ipc: host
    shm_size: "${LLAMA_SHM_SIZE}"
    extra_hosts:
      - host.docker.internal:host-gateway
    healthcheck:
      test:
        [
          "CMD-SHELL",
          "curl -fsS http://127.0.0.1:8080/health >/dev/null || exit 1",
        ]
      interval: 30s
      timeout: 10s
      retries: 20
      start_period: 60s
    restart: unless-stopped


volumes:
  llama-bin:

```

`nano Dockerfile`, no need to edit anything as well:
```Dockerfile
FROM nvidia/cuda:12.6.3-devel-ubuntu24.04 AS builder

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y --no-install-recommends \
    pciutils \
    build-essential \
    cmake \
    curl \
    libcurl4-openssl-dev \
    ca-certificates \
    git \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /src

COPY llama.cpp /src/llama.cpp

RUN cmake /src/llama.cpp -B /src/llama.cpp/build \
      -DBUILD_SHARED_LIBS=OFF \
      -DGGML_CUDA=ON \
 && cmake --build /src/llama.cpp/build \
      --config Release \
      -j"$(nproc)" \
      --clean-first \
      --target llama-cli llama-mtmd-cli llama-server llama-gguf-split \
 && mkdir -p /opt/llama.cpp/bin \
 && cp /src/llama.cpp/build/bin/llama-* /opt/llama.cpp/bin/


FROM nvidia/cuda:12.6.3-runtime-ubuntu24.04

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y --no-install-recommends \
    pciutils \
    curl \
    libcurl4 \
    libgomp1 \
    ca-certificates \
    bash \
 && rm -rf /var/lib/apt/lists/*

COPY --from=builder /opt/llama.cpp/bin /opt/llama.cpp/bin
COPY entrypoint.sh /entrypoint.sh

RUN chmod +x /entrypoint.sh \
 && mkdir -p /llama.cpp /models

ENV PATH="/llama.cpp:${PATH}" \
    LLAMA_BIN_DIR="/llama.cpp" \
    LLAMA_SERVER_BIN="/llama.cpp/llama-server" \
    LLAMA_HOST="0.0.0.0" \
    LLAMA_PORT="8080"

VOLUME ["/llama.cpp"]

EXPOSE 8080

ENTRYPOINT ["/entrypoint.sh"]

```

`nano entrypoint.sh`, can be optimized, for sure:
```bash
#!/usr/bin/env bash
set -euo pipefail

mkdir -p /llama-bin

# The named volume mounted at /llama-bin persists the built llama.cpp binaries.
# Default behavior: refresh it from the image on every container start so a
# rebuilt image updates the persisted volume too. Set LLAMA_SYNC_BINARIES=0 to
# copy only when the volume is empty.
if [[ "${LLAMA_SYNC_BINARIES:-1}" == "1" ]] || [[ ! -x /llama-bin/llama-server ]]; then
  cp -a /opt/llama.cpp/bin/. /llama-bin/
  cp -a /opt/llama.cpp/LLAMA_CPP_COMMIT /llama-bin/ 2>/dev/null || true
  chmod +x /llama-bin/llama-* 2>/dev/null || true
fi

export PATH="/llama-bin:${PATH}"
export LD_LIBRARY_PATH="/llama-bin:${LD_LIBRARY_PATH:-}"

if [[ "${LLAMA_ARG_N_GPU_LAYERS:-}" == "auto" ]] || [[ "${LLAMA_ARG_N_GPU_LAYERS:-}" == "" ]]; then
  unset LLAMA_ARG_N_GPU_LAYERS
fi

case "${LLAMA_ARG_SPEC_TYPE:-}" in
  ""|0|off|false|none|disabled)
    unset LLAMA_ARG_SPEC_TYPE
    unset LLAMA_ARG_SPEC_DRAFT_N_MAX
    ;;
esac

if [[ "$#" -eq 0 ]]; then
  set -- /llama-bin/llama-server
fi

# Allow docker compose command: ["--model", "/models/model.gguf", ...]
if [[ "${1#-}" != "$1" ]]; then
  set -- /llama-bin/llama-server "$@"
fi

exec "$@"
```

start the stack
```bash
docker compose up -d --build
```

## the harness on the dev machine

I use the official docker image of https://github.com/anomalyco/opencode

```bash
mkdir -p "$HOME/.local/share/opencode" "$HOME/.config/opencode"
mkdir -p \
  "$HOME/.cache/opencode-home/.config/opencode" \
  "$HOME/.cache/opencode-home/.local/share/opencode" \
  "$HOME/.cache/opencode-home/.local/state" \
  "$HOME/.cache/opencode-home/.cache"
```


then `nano $HOME/.cache/opencode-home/.config/opencode/opencode.json` with the following content:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "default_agent": "plan",
  "provider": {
    "llamacpp": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "llama.cpp",
      "options": {
        "baseURL": "http://host.docker.internal:8081/v1",
        "apiKey": "sk-no-key-required"
      },
      "models": {
        "/models/Huihui-Qwen3.6-35B-A3B-Claude-4.7-Opus-abliterated-ggml-model-Q4_K.gguf": {
          "name": "Qwen3.6-35B via llama.cpp",
          "limit": {
            "context": 102400,
            "output": 32768
          }
        },
        "/models/Qwen3.6-27B-NEO-CODE-HERE-2T-OT-Q4_K_M.gguf": {
          "name": "Qwen3.6-27B via llama.cpp",
          "limit": {
            "context": 102400,
            "output": 32768
          },
          "options": {
            "temperature": 0.6,
            "topP": 0.95,
            "topK": 20,
            "minP": 0.0,
            "presencePenalty": 0.0,
            "repetitionPenalty": 1.0
          }
        },
      }
    }
  },
  "model": "/models/Huihui-Qwen3.6-35B-A3B-Claude-4.7-Opus-abliterated-ggml-model-Q4_K.gguf",
  "small_model": "/models/Huihui-Qwen3.6-35B-A3B-Claude-4.7-Opus-abliterated-ggml-model-Q4_K.gguf"
}
```

## using the agent

because I run the inference engine on another machine, i need to forward the remote port with llama.cpp service to my dev VM with
```bash
ip -4 addr show docker0
# my first docker bridge ip is 172.17.0.1 so:
sudo ufw allow in on docker0 from 172.17.0.0/16 to 172.17.0.1 port 8081 proto tcp
ssh -N inference -L 172.17.0.1:8081:localhost:8081
```

a container started with `docker run` without the network parameter will connect to the first bridge network, which is `docker0`

**the keypart:**

`cd` into a directory with a project we want to work on with an uncensored model, then run

```bash
docker run -it --rm \
  --add-host=host.docker.internal:host-gateway \
  --user "$(id -u):$(id -g)" \
  -e HOME=/home/opencode \
  -e XDG_CONFIG_HOME=/home/opencode/.config \
  -e XDG_DATA_HOME=/home/opencode/.local/share \
  -e XDG_STATE_HOME=/home/opencode/.local/state \
  -e XDG_CACHE_HOME=/home/opencode/.cache \
  -e OPENCODE_DISABLE_MODELS_FETCH=true \
  -e OPENCODE_DISABLE_SHARE=true \
  -e OPENCODE_DISABLE_AUTOUPDATE=true \
  -v "$PWD:/workspace" \
  -w /workspace \
  -v "$HOME/.cache/opencode-home:/home/opencode" \
  ghcr.io/anomalyco/opencode
```

then inside the harness use command `/connect` and choose our provider `llama.cpp` and the model to be default for this session. then start with `/init`. when returning later, choose the latest `/session`. use tab to switch between plan and build modes

## performance

P40 gives me 130 tokens per second on prompt processing and 25 tokens per second on token generation (depending on the current amount of tokens present in the context, with this context window and model). this is fair for a PoC, and this setup is scalable to bigger models and any other more powerful hardware that is supported by llama.cpp. The performance is suitable for simple tasks. scaling nodes with llama.cpp is not viable because it would be relatively slow `pipeline parallelism`. the better option for multi-node is `tensor parallelism` with vLLM/SGLang/TRT-LLM.

## references

- https://github.com/VooDisss/opencode-privacy-fix
- https://github.com/anomalyco/opencode
- https://github.com/ggml-org/llama.cpp