---
layout: post
title:  "A code agent with a local uncensored LLM"
lang: en
tags: [llm, agent, local]
category: tutorial
published: true
---

I believe that neutering LLMs with guardrails is detrimental to the model's performance, hurts the model usability in a broader sense and makes the model pretty much useless for certain specific tasks such as cybersecurity. The simplest thing to somewhat mitigate this is to run an uncensored model locally. Running locally also gives us more capabilities and more control over the LLM. Another aspect is that running inference locally keeps the data and metadata private. So I stringed together an instance of llama.cpp running an uncensored model and a coding harness. The harness and the inference engine run in docker containers, that allows for a certain level of isolation from the host system, and also flexibility of deployment. The harness is running in a rootless docker container with project directory mounted as a bind volume and can not access the filesystem outside of it. The image for inference container can be build on a dev machine and then transferred to and deployed on a machine without internet access. The inference engine can be used by other software as well, like open-webui. The following is my runbook. **The models in this example are too small and are not suitable for doing complex tasks, there are very few usecases for a setup like this.**

## todo

- [x] optimized llama.cpp docker deployment

## my setup

Ubuntu VM for the dev environment, [Debian LXC with an NVIDIA P40 and docker]({% post_url 2024-12-7-gpu-home-server %}) for running the inference. Also would need a Python virtual environment (through conda or venv) to download the model using hf cli.

## the model

I used [modelheretic.com](https://modelheretic.com/) to find an uncensored model that would fit my hardware, I decided to go with a MoE [Huihui-Qwen3.6-35B-A3B-Claude-4.7-Opus-abliterated-ggml-model-Q4_K](https://huggingface.co/huihui-ai/Huihui-Qwen3.6-35B-A3B-Claude-4.7-Opus-abliterated-MTP-GGUF) and dense [Qwen3.6-27B-Heretic-Uncensored-FINETUNE-NEO-CODE-Di-IMatrix-MAX-GGUF](https://huggingface.co/DavidAU/Qwen3.6-27B-Heretic-Uncensored-FINETUNE-NEO-CODE-Di-IMatrix-MAX-GGUF)  which are already in GGUF format. there are also regular models fine tuned for cyber, like the [formerly known as the whiterabbit](https://huggingface.co/DeepHat/DeepHat-V1-7B), 
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



`nano docker-compose.yml`:
```
services:
  llama-cpp:
    build: .
    container_name: llama-cpp
    volumes:
      - ./models:/models:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - 127.0.0.1:8081:8080
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
    ipc: host
    healthcheck:
      test: [CMD, curl, -f, http://localhost:8080/health]
      interval: 30s
      timeout: 10s
      retries: 20
      start_period: 60s
    restart: unless-stopped
```

`nano Dockerfile`, no need to edit anything except for the architecture:
```Dockerfile
FROM nvidia/cuda:12.6.3-devel-ubuntu24.04 AS build

RUN apt-get update && apt-get install -y --no-install-recommends \
      build-essential cmake libcurl4-openssl-dev \
 && rm -rf /var/lib/apt/lists/*

COPY llama.cpp /src
# DCMAKE_CUDA_ARCHITECTURES Tesla P40: 61; RTX 4090: 89; RTX 3090/3080: 86; A100: 80.
RUN cmake -S /src -B /src/build -DGGML_CUDA=ON -DBUILD_SHARED_LIBS=OFF -DCMAKE_CUDA_ARCHITECTURES=61 \
 && cmake --build /src/build -j"$(nproc)" --target llama-server

FROM nvidia/cuda:12.6.3-runtime-ubuntu24.04

RUN apt-get update && apt-get install -y --no-install-recommends curl libcurl4 libgomp1 \
 && rm -rf /var/lib/apt/lists/*

COPY --from=build /src/build/bin/llama-server /usr/local/bin/llama-server
COPY --chmod=755 entrypoint.sh /entrypoint.sh

EXPOSE 8080
ENTRYPOINT ["/entrypoint.sh"]
```

`nano entrypoint.sh`:
```bash
#!/bin/sh
exec llama-server \
  --models-dir /models \
  --host 0.0.0.0 \
  --port 8080 \
  --n-gpu-layers 999 \
  --n-cpu-moe 6 \
  --ctx-size 204800 \
  --flash-attn on \
  --cache-type-k q8_0 \
  --cache-type-v q8_0 \
  --parallel 1 \
  --batch-size 256 \
  --ubatch-size 128 \
  --no-mmap
```

start the stack
```bash
docker compose up -d --build
```

## the harness on the dev machine

I use the official docker image of https://github.com/anomalyco/opencode

```bash
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
            "context": 204800,
            "output": 32768
          }
        },
        "/models/Qwen3.6-27B-NEO-CODE-HERE-2T-OT-Q4_K_M.gguf": {
          "name": "Qwen3.6-27B via llama.cpp",
          "limit": {
            "context": 204800,
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





add rootless docker

```bash
dockerd-rootless-setuptool.sh install --force
systemctl --user enable --now docker
```

because I run the inference engine on another machine, i need to forward the remote port with llama.cpp service to my dev VM. for that wo work we need to solve the container network connectivity with the loopback. in:

```bash
systemctl --user edit docker
```

add:
```
[Service]
Environment=DOCKERD_ROOTLESS_ROOTLESSKIT_DISABLE_HOST_LOOPBACK=false
```

then
```bash
systemctl --user daemon-reload
systemctl --user restart docker
docker pull ghcr.io/anomalyco/opencode:1.17.18
```

**the keypart:**

forward the port to the local dev machine:
```bash
ssh -N inference -L 127.0.0.1:8081:localhost:8081
```

`cd` into a directory with a project we want to work on with an uncensored model, then run

```bash
docker --context rootless run -it --rm \
  --add-host=host.docker.internal:10.0.2.2 \
  -e HOME=/root \
  -e OPENCODE_DISABLE_MODELS_FETCH=true \
  -e OPENCODE_DISABLE_SHARE=true \
  -e OPENCODE_DISABLE_AUTOUPDATE=true \
  -v "$PWD:/workspace" \
  -w /workspace \
  -v "$HOME/.cache/opencode-home:/root" \
  ghcr.io/anomalyco/opencode:1.17.18
```


then inside the harness use command `/connect` and choose our provider `llama.cpp` and the model to be default for this session. then start with `/init`. when returning later, choose the latest `/session`. use tab to switch between plan and build modes

## performance

P40 gives me 130 tokens per second on prompt processing and 25 tokens per second on token generation (while offloading 5 eperts to CPU memory, and depending on the current amount of tokens present in the context, with this context window and model). this is fair for a PoC, and this setup is scalable to bigger models and any other more powerful hardware that is supported by llama.cpp. The performance is suitable for simple tasks. scaling nodes with llama.cpp is not viable because it would be relatively slow `pipeline parallelism`. the better option for multi-node is `tensor parallelism` with vLLM/SGLang/TRT-LLM.

## references

- https://github.com/VooDisss/opencode-privacy-fix
- https://github.com/anomalyco/opencode
- https://github.com/ggml-org/llama.cpp