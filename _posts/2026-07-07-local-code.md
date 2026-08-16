---
layout: post
title:  "A code agent with a local uncensored LLM"
lang: en
tags: [llm, agent, local-ai]
category: tutorial
published: true
---

I believe that neutering LLMs with guardrails is detrimental to the model's performance, hurts the model usability in a broader sense and makes the model pretty much useless for certain specific tasks such as cybersecurity. The simplest thing to somewhat mitigate this is to run an uncensored model locally. Running locally also gives us more capabilities and more control over the LLM. Another aspect is that running inference locally keeps the data and metadata private. So I stringed together an instance of llama.cpp running an uncensored model and a coding harness. The harness and the inference engine run in docker containers, that allows for a certain level of isolation from the host system, and also flexibility of deployment. The harness is running in a rootless docker container with project directory mounted as a bind volume and can not access the filesystem outside of it. The image for inference container can be build on a dev machine and then transferred to and deployed on a machine without internet access. The inference engine can be used by other software as well, like open-webui. The following is my runbook. **The models in this example are too small and are not suitable for doing complex tasks, there are very few usecases for a setup like this.** in this runbook opencode uses chat completions, codex setup uses llama.cpp through API.

## todo

- [x] optimize llama.cpp's docker deployment
- [x] combine opencode and codex-cli runbooks

## my setup

Ubuntu VM for the dev environment, [Debian LXC with an NVIDIA P40 and docker]({% post_url 2024-12-7-gpu-home-server %}) for running the inference. Also would need a Python virtual environment (through conda or venv) to download the model using hf cli.

## the model

I used [modelheretic.com](https://modelheretic.com/) to find an uncensored model that would fit my hardware, I decided to go with a MoE [Huihui-Qwen3.6-35B-A3B-Claude-4.7-Opus-abliterated-ggml-model-Q4_K](https://huggingface.co/huihui-ai/Huihui-Qwen3.6-35B-A3B-Claude-4.7-Opus-abliterated-MTP-GGUF) and dense [Qwen3.6-27B-Heretic-Uncensored-FINETUNE-NEO-CODE-Di-IMatrix-MAX-GGUF](https://huggingface.co/DavidAU/Qwen3.6-27B-Heretic-Uncensored-FINETUNE-NEO-CODE-Di-IMatrix-MAX-GGUF) and 
[Qwen3.6-27B-Fable-Fusion-711-Uncensored-Heretic-NM-DAU-NEO-MAX-MTP-GGUF](https://huggingface.co/DavidAU/Qwen3.6-27B-Fable-Fusion-711-Uncensored-Heretic-NM-DAU-NEO-MAX-MTP-GGUF) which are already in GGUF format. there are also regular models fine tuned for cyber, like the [formerly known as the whiterabbit](https://huggingface.co/DeepHat/DeepHat-V1-7B), 
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
#  HF_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxx hf download huihui-ai/Huihui-Qwen3.6-35B-A3B-Claude-4.7-Opus-abliterated-MTP-GGUF \
#   Huihui-Qwen3.6-35B-A3B-Claude-4.7-Opus-abliterated-ggml-model-Q4_K.gguf \
#   --include "*Q4_K*.gguf" --local-dir ./models --repo-type model --revision main

#  HF_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxx hf download DavidAU/Qwen3.6-27B-Heretic-Uncensored-FINETUNE-NEO-CODE-Di-IMatrix-MAX-GGUF \
#   Qwen3.6-27B-NEO-CODE-HERE-2T-OT-Q4_K_M.gguf \
#   --local-dir ./models --repo-type model --revision main

 HF_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxx hf download hf://DavidAU/Qwen3.6-27B-Fable-Fusion-711-Uncensored-Heretic-NM-DAU-NEO-MAX-MTP-GGUF/Qwen3.6-27B-Fable-Fus-711-UnHeretic-NM-DAU-NEO-MAX-NEO-MTP-Q4_K_M.gguf \
  --local-dir ./models
```

also need to download and modify the Jinja template for the model:

```bash
curl -fL \
  https://huggingface.co/DavidAU/Qwen3.6-27B-Fable-Fusion-711-Uncensored-Heretic-NM-DAU-MTP/raw/main/chat_template-instruct.jinja \
  -o qwen3.6-codex.jinja
```

replace 


{% raw %}
```
{%- if message.role == "system" or message.role == "developer" %}
    {%- if not loop.first %}
        {{- raise_exception('System message must be at the beginning.') }}
    {%- endif %}
```
{% endraw %}


with


{% raw %}
```
{%- if message.role == "system" or message.role == "developer" %}
    {# The initial system/developer message was rendered above. #}
```
{% endraw %}



## the inference engine

`git clone https://github.com/ggml-org/llama.cpp.git` . sure there are simpler methods to run llama.cpp, but I must compile it from the source code myself, i need convenient config files, more control, more flexibility and isolation, so I went with a setup like this. the complexity is compensated by the ease and flexibility of deployment. this setup can be modified for being completely self sustained, with the bind mounts removed and the weights baked into the image. this kind of image can be built locally and then transferred to a remote machine like `docker save llama-cpp:local | ssh $USER@inference docker load` without internet access and the container deployment controlled by the remote docker daemon with `DOCKER_HOST="ssh://$USER@inference" docker ...`


```
cat > docker-compose.yml <<'EOF'
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
EOF
```

no need to edit anything except for the architecture:

```Dockerfile
cat > Dockerfile <<'EOF'
FROM nvidia/cuda:12.6.3-devel-ubuntu24.04 AS build

RUN apt-get update && apt-get install -y --no-install-recommends \
      build-essential cmake libcurl4-openssl-dev \
 && rm -rf /var/lib/apt/lists/*

COPY llama.cpp /src
# DCMAKE_CUDA_ARCHITECTURES Tesla P40: 61; RTX 4090: 89; RTX 3090/3080: 86; A100: 80.
RUN cmake -S /src -B /src/build -DGGML_CUDA=ON -DBUILD_SHARED_LIBS=OFF -DCMAKE_CUDA_ARCHITECTURES=61 -DGGML_CUDA_FA_ALL_QUANTS=ON \
 && cmake --build /src/build -j"$(nproc)" --target llama-server

FROM nvidia/cuda:12.6.3-runtime-ubuntu24.04

RUN apt-get update && apt-get install -y --no-install-recommends curl libcurl4 libgomp1 \
 && rm -rf /var/lib/apt/lists/*

COPY --from=build /src/build/bin/llama-server /usr/local/bin/llama-server
COPY --chmod=755 entrypoint.sh /entrypoint.sh

EXPOSE 8080
ENTRYPOINT ["/entrypoint.sh"]
EOF
```

`spec-` flags are for MTP mode with MTP models. multi-user inference i.e. when `--parallel` > 1 is incompatible with MTP:
```bash
cat > entrypoint.sh <<'EOF'
#!/bin/sh
exec llama-server \
  --model /models/Qwen3.6-27B-Fable-Fus-711-UnHeretic-NM-DAU-NEO-MAX-NEO-MTP-Q4_K_M.gguf \
  --host 0.0.0.0 \
  --port 8080 \
  --n-gpu-layers 999 \
  --fit on \
  --fit-target 512 \
  --flash-attn on \
  --cache-type-k q8_0 \
  --cache-type-v q8_0 \
  --spec-draft-type-k q8_0 \
  --spec-draft-type-v q8_0 \
  --parallel 1 \
  --batch-size 256 \
  --ubatch-size 128 \
  --no-mmap \
  --spec-type draft-mtp \
  --spec-draft-n-max 2 \
  --jinja \
  --chat-template-file /models/qwen3.6-codex.jinja
EOF
```

start the stack
```bash
docker compose up -d --build
```

test with:

```bash
curl http://127.0.0.1:8081/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-no-key-required" \
  -d '{
    "model": "Qwen3.6-27B-Fable-Fus-711-UnHeretic-NM-DAU-NEO-MAX-NEO-MTP-Q4_K_M",
    "messages": [
      {"role": "user", "content": "What is 2+2?"}
    ]
  }' | python3 -m json.tool
```

## the harness on the dev machine, opencode

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
        "Qwen3.6-27B-Fable-Fus-711-UnHeretic-NM-DAU-NEO-MAX-NEO-MTP-Q4_K_M": {
          "name": "Qwen3.6-27B-MTP via llama.cpp",
          "limit": {
            "context": 66000,
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
  "model": "Qwen3.6-27B-Fable-Fus-711-UnHeretic-NM-DAU-NEO-MAX-NEO-MTP-Q4_K_M",
  "small_model": "Qwen3.6-27B-Fable-Fus-711-UnHeretic-NM-DAU-NEO-MAX-NEO-MTP-Q4_K_M"
}
```

add rootless docker

```bash
dockerd-rootless-setuptool.sh install --force
systemctl --user enable --now docker
```

because I run the inference engine on another machine, i need to forward the remote port with llama.cpp service to my dev VM. for that to work we need to solve the container network connectivity with the loopback. in:

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

forward the port to the local dev machine:
```bash
ssh -N inference -L 127.0.0.1:8081:localhost:8081
```

find the newest image
```bash
skopeo list-tags docker://ghcr.io/anomalyco/opencode
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
  ghcr.io/anomalyco/opencode:1.18.14
```

for a particular project remove `-rm` and add `--name projectname` for persistence, and then `docker --context rootless start -ai projectname`.

then inside the harness use command `/connect` and choose our provider `llama.cpp` and the model to be default for this session. then start with `/init`. when returning later, choose the latest `/session`. use tab to switch between plan and build modes


## the harness on the dev machine, codex-cli



prepare the files for the image

```bash
cat > models.json <<'EOF'
{
  "models": [
    {
      "slug": "Qwen3.6-27B-Fable-Fus-711-UnHeretic-NM-DAU-NEO-MAX-NEO-MTP-Q4_K_M",
      "display_name": "Qwen3.6 27B MTP via llama.cpp",
      "description": "Local Qwen3.6 coding model served by llama.cpp",
      "default_reasoning_level": "low",
      "supported_reasoning_levels": [
        {
          "effort": "low",
          "description": "Use the model's normal local reasoning behavior"
        }
      ],
      "shell_type": "shell_command",
      "apply_patch_tool_type": "freeform",
      "visibility": "list",
      "supported_in_api": true,
      "priority": 0,
      "base_instructions": "You are a coding agent working in the user's repository. Inspect relevant files before changing them, use the provided tools for shell commands and edits, preserve unrelated changes, and verify your work.",
      "supports_parallel_tool_calls": false,
      "context_window": 66000,
      "max_context_window": 66000,
      "effective_context_window_percent": 90,
      "auto_compact_token_limit": 50000,
      "reasoning_summary_format": "none",
      "default_reasoning_summary": "none",
      "supports_reasoning_summaries": false,
      "supports_reasoning_summary_parameter": false,
      "support_verbosity": false,
      "default_verbosity": null,
      "truncation_policy": {
        "mode": "tokens",
        "limit": 10000
      },
      "input_modalities": ["text"],
      "supports_image_detail_original": false,
      "prefer_websockets": false,
      "experimental_supported_tools": [],
      "supports_search_tool": false,
      "use_responses_lite": false,
      "include_skills_usage_instructions": false,
      "auto_review_model_override": null,
      "tool_mode": null,
      "multi_agent_version": null,
      "availability_nux": null,
      "minimal_client_version": "0.144.0",
      "upgrade": null
    }
  ]
}
EOF


cat > config.toml <<'EOF'
model = "Qwen3.6-27B-Fable-Fus-711-UnHeretic-NM-DAU-NEO-MAX-NEO-MTP-Q4_K_M"
model_provider = "llamacpp_qwen"
model_catalog_json = "/root/.codex/models.json"
model_context_window = 66000
model_auto_compact_token_limit = 50000
model_reasoning_summary = "none"
model_supports_reasoning_summaries = false

# The local llama.cpp service has no Codex hosted-search backend.
web_search = "disabled"
check_for_update_on_startup = false

[feedback]
enabled = false

[model_providers.llamacpp_qwen]
name = "Qwen3.6 via host llama.cpp"
base_url = "http://host.docker.internal:8081/v1"
wire_api = "responses"
requires_openai_auth = false
supports_websockets = false
supports_standalone_web_search = false
stream_idle_timeout_ms = 600000
request_max_retries = 1
stream_max_retries = 1

[projects."/workspace"]
trust_level = "trusted"
EOF


cat > Dockerfile <<'EOF'
FROM node:22-bookworm-slim

ARG CODEX_VERSION=0.147.0-alpha.6.5

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        bash \
        ca-certificates \
        curl \
        git \
        jq \
        less \
        openssh-client \
        python3 \
        ripgrep \
    && npm install --global "@openai/codex@${CODEX_VERSION}" \
    && npm cache clean --force \
    && rm -rf /var/lib/apt/lists/*

ENV HOME=/root \
    CODEX_HOME=/root/.codex \
    TERM=xterm-256color

RUN mkdir -p /root/.codex /workspace \
    && git config --system --add safe.directory /workspace

COPY --chmod=600 config.toml /root/.codex/config.toml
COPY --chmod=644 models.json /root/.codex/models.json

WORKDIR /workspace

# Docker is the outer sandbox. This lets Codex use all tools inside the
# container without trying to create a second sandbox inside it.
ENTRYPOINT ["codex", "--dangerously-bypass-approvals-and-sandbox"]
EOF

```

build the image
```bash
docker --context rootless build \
  --build-arg CODEX_VERSION=0.147.0-alpha.6.5 \
  -t localhost/codex-qwen-llamacpp:0.147.0-alpha.6.5 \
  .
```

verify
```bash
docker --context rootless run --rm \
  --entrypoint codex \
  localhost/codex-qwen-llamacpp:0.147.0-alpha.6.5 \
  --version
```

enable the host loopback the same way as above if not enabled yet.

forward the port to the local dev machine as before:
```bash
ssh -N inference -L 127.0.0.1:8081:localhost:8081
```

`cd` into a directory with a project we want to work on with an uncensored model, then run

```bash
docker --context rootless run --rm -it --init \
  --add-host=host.docker.internal:10.0.2.2 \
  --mount "type=bind,src=$(pwd -P),dst=/workspace" \
  --workdir /workspace \
  --env HOME=/root \
  --env CODEX_HOME=/root/.codex \
  localhost/codex-qwen-llamacpp:0.147.0-alpha.6.5
```

for a particular project remove `-rm` and add `--name projectname` for persistence, and then `docker --context rootless start -ai projectname`.





## performance

the speed is relatively slow, but this is fair for a PoC, and this setup is scalable with bigger models and any other more powerful hardware that is supported by llama.cpp. The current performance is suitable for simple tasks. scaling nodes with llama.cpp is not viable because it would be relatively slow `pipeline parallelism`. the better option for multi-node is `tensor parallelism` with vLLM/SGLang/TRT-LLM.

## references

- https://github.com/VooDisss/opencode-privacy-fix
- https://github.com/anomalyco/opencode
- https://github.com/ggml-org/llama.cpp