---
layout: post
title:  "Containerized codex-cli with a local uncensored LLM"
lang: en
tags: [llm, agent, local-ai]
category: tutorial
published: true
---

Another runbook on how to run coding harness with a local uncensored LLM. the previous was about the same [local LLM and opencode]({% post_url 2026-07-07-local-code %}) . this one differs in that opencode used chat completions, and this codex setup uses llama.cpp through API.

## my setup

Ubuntu VM for the dev environment, [Debian LXC with an NVIDIA P40 and docker]({% post_url 2024-12-7-gpu-home-server %}) for running the inference. Also would need a Python virtual environment (through conda or venv) to download the model using hf cli.

## the model

same models as in [opencode setup]({% post_url 2026-07-07-local-code %}).

the difference is that we also need the Jinja template for the model.

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


and put the file to the `models` directory

## the inference engine

same setup as in [opencode setup]({% post_url 2026-07-07-local-code %}) but need to add some flags to entrypoint.sh:
```
--jinja \
--chat-template-file /models/qwen3.6-codex.jinja \
```


## the harness on the dev machine

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
      "context_window": 102400,
      "max_context_window": 102400,
      "effective_context_window_percent": 90,
      "auto_compact_token_limit": 90000,
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
model_context_window = 102400
model_auto_compact_token_limit = 90000
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

## using the agent

add rootless docker the same way as in [the opencode setup]({% post_url 2026-07-07-local-code %}).

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


