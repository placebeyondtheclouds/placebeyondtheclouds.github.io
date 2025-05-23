---
layout: post
title:  "How to convert Huggingface safetensors to Ollama model"
lang: en
tags: [en, ollama, huggingface, gguf, quantization, safetensors]
published: true
---

## the end result:

- LLM weights from Huggingface hub running locally with ollama backend

## prerequisites:

- an Ubuntu/Debian VM with NVIDIA GPU, conda, git etc. not sure if a GPU is needed for this particular task (it didn't use the GPU during the conversion), but the requirements for the converter include pytorch, which would install different versions depending on if a GPU is present in the system or not, and I believe it's better to go with CUDA version just in case. never tried it with the CPU-only pytorch

## steps:

create env and download the HF model. I took [the newest one](https://huggingface.co/WhiteRabbitNeo/WhiteRabbitNeo-2.5-Qwen-2.5-Coder-7B) out of the [WhiteRabbitNeo series](https://www.whiterabbitneo.com/) models as an example.

```shell
conda create --name convert_env python=3.10 -y
conda activate convert_env
pip install huggingface_hub
mkdir convert && cd convert
```

then

```shell
tee download.py <<EOF
from huggingface_hub import snapshot_download
# import os
# os.environ['HF_ENDPOINT'] = 'https://hf-mirror.com'
model_id="WhiteRabbitNeo/WhiteRabbitNeo-2.5-Qwen-2.5-Coder-7B"
snapshot_download(repo_id=model_id, local_dir="wrn-hf",
                  local_dir_use_symlinks=False, revision="main")
EOF

python download.py
```

or

```shell
#export HF_ENDPOINT=https://hf-mirror.com
huggingface-cli download --resume-download WhiteRabbitNeo/WhiteRabbitNeo-2.5-Qwen-2.5-Coder-7B --local-dir ./wrn-hf --cache-dir ./wrn-hf/.cache/
```



[quantization explained](https://github.com/ggerganov/llama.cpp/pull/1684)


as per [llama.cpp](https://github.com/ggerganov/llama.cpp/blob/5f6e0c0dff1e7a89331e6b25eca9a9fd71324069/examples/make-ggml.py):
```
New quant types (recommended):
- Q2_K: smallest, extreme quality loss - not recommended
- Q3_K: alias for Q3_K_M
- Q3_K_S: very small, very high quality loss
- Q3_K_M: very small, very high quality loss
- Q3_K_L: small, substantial quality loss
- Q4_K: alias for Q4_K_M
- Q4_K_S: small, significant quality loss
- Q4_K_M: medium, balanced quality - recommended
- Q5_K: alias for Q5_K_M
- Q5_K_S: large, low quality loss - recommended
- Q5_K_M: large, very low quality loss - recommended
- Q6_K: very large, extremely low quality loss
- Q8_0: very large, extremely low quality loss - not recommended
- F16: extremely large, virtually no quality loss - not recommended
- F32: absolutely huge, lossless - not recommended
```

use `q4_k_m` for large models and `q8_0` for small models

check if the model is listed in supported models:

```shell
python llama.cpp/convert_hf_to_gguf.py  --print-supported-models
```

converting Huggingface safetensors format to GGUF:

```shell
git clone https://github.com/ggerganov/llama.cpp.git
pip install -r llama.cpp/requirements.txt
python llama.cpp/convert_hf_to_gguf.py wrn-hf --outfile WhiteRabbitNeo-2.5-Qwen-2.5-Coder-7B-Q8_0.gguf --outtype q8_0
```

(optional) move the model to the docker container that runs ollama:

```shell
scp WhiteRabbitNeo-2.5-Qwen-2.5-Coder-7B-Q8_0.gguf $USER@192.168.3.200:~/WhiteRabbitNeo-2.5-Qwen-2.5-Coder-7B-Q8_0.gguf
rm WhiteRabbitNeo-2.5-Qwen-2.5-Coder-7B-Q8_0.gguf
ssh $USER@192.168.3.200 docker cp WhiteRabbitNeo-2.5-Qwen-2.5-Coder-7B-Q8_0.gguf ollama:/WhiteRabbitNeo-2.5-Qwen-2.5-Coder-7B-Q8_0.gguf
ssh $USER@192.168.3.200 rm WhiteRabbitNeo-2.5-Qwen-2.5-Coder-7B-Q8_0.gguf
DOCKER_HOST="ssh://$USER@192.168.3.200" docker exec -it ollama bash
```

creating Ollama model from GGUF file:

```shell
tee WhiteRabbitNeo-2.5-Qwen-2.5-Coder-7B-Q8_0.modelfile <<EOF
FROM "./WhiteRabbitNeo-2.5-Qwen-2.5-Coder-7B-Q8_0.gguf"
SYSTEM """You are an AI that can code. Please answer with code, and make sure to format with codeblocks using ``` and ```."""
PARAMETER temperature 0.75
PARAMETER top_k 50
PARAMETER top_p 1.0
PARAMETER num_ctx 16384
TEMPLATE """
<|im_start|>system
{{ .System }}<|im_end|>
<|im_start|>user
{{ .Prompt }}<|im_end|>
<|im_start|>assistant\n
Sure! Let me provide a complete and a thorough answer to your question, with functional and production ready code.
{{ .Response }}<|im_end|>
"""
EOF

ollama create "WhiteRabbitNeo-2.5-Qwen-2.5-Coder-7B-Q8_0" -f WhiteRabbitNeo-2.5-Qwen-2.5-Coder-7B-Q8_0.modelfile

```

test with:

```shell
ollama run WhiteRabbitNeo-2.5-Qwen-2.5-Coder-7B-Q8_0:latest
```

## references:

- https://www.substratus.ai/blog/converting-hf-model-gguf-model/
- https://github.com/ggerganov/llama.cpp/discussions/2948
- https://huggingface.co/WhiteRabbitNeo/WhiteRabbitNeo-2.5-Qwen-2.5-Coder-7B
- https://www.gpu-mart.com/blog/import-models-from-huggingface-to-ollama
- https://github.com/ggerganov/llama.cpp/pull/1684