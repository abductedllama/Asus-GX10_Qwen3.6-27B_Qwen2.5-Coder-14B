# Asus-GX10_Qwen3.6-27B_Qwen2.5-Coder-14B
vLLM docker compose configs for Qwen Models on Asus GX10

Running two models on **ASUS GX10** (ARM64) via Docker Compose.

This may or may not be optimal. I was just trying to get two decently large models with a large context, running smoothly under any request.

## Models

| Service | Model | Purpose | Port |
|---|---|---|---|
| `qwen-reason` | `sakamakismile/Huihui-Qwen3.6-35B-A3B-abliterated-NVFP4` | Reasoning | 8000 |
| `qwen-coder` | `ilessio-aiflowlab/Qwen2.5-Coder-14B-Instruct-NVFP4-anima` | Coding | 8001 |

## Quick Start

```bash
# Start both containers
docker compose -f vllm/docker-compose.yml up -d
docker compose -f qwen-coder/docker-compose.yml up -d

# Test
curl http://localhost:8000/v1/models
curl http://localhost:8001/v1/models
```

## Hardware

- **ASUS GX10** (ARM64, GB10 (SM121), ~128 GB unified memory)
  - Should also work in theory on other Nvidia Spark machines
- Both containers use `nvidia` driver for GPU access

## Status

- `docker-compose.yml` files are live
- More details about tuning, concurrency, and optimization coming soon

Pushes memory pretty far:
```bash
$ free -h
               total        used        free      shared  buff/cache   available
Mem:           121Gi       114Gi       1.5Gi       188Mi       7.0Gi       7.2Gi
Swap:           15Gi       6.9Gi       9.1Gi
```

> **WIP** — initial setup, more to follow.

## "Qwen-reason" (Qwen3.6-35B-A3B-abliterated-NVFP4) /qwen-reason/docker-compose.yml

```bash
services:
  vllm:
    image: vllm/vllm-openai:v0.26.0  # Was: vllm/vllm-openai:latest
    container_name: vllm

    ports:
      - "8000:8000"

    environment:
      HF_TOKEN: ${HF_TOKEN}

    volumes:
      - ~/.cache/huggingface:/root/.cache/huggingface
      - ~/.cache/vllm:/root/.cache/vllm


    command:
      - sakamakismile/Huihui-Qwen3.6-35B-A3B-abliterated-NVFP4
      - --host
      - 0.0.0.0
      - --port
      - "8000"
      - --tensor-parallel-size
      - "1"
      - --kv-cache-dtype
      - fp8
      - --attention-backend
      - flashinfer
      - --gpu-memory-utilization
      - "0.55"
      - --max-model-len
      - "262144"
      - --max-num-seqs
      - "4"
      - --max-num-batched-tokens
      - "16384"
      - --enable-chunked-prefill
      - --async-scheduling
      - --enable-prefix-caching
      - --speculative-config
      - '{"method":"mtp","num_speculative_tokens":3}'
      - --load-format
      - safetensors
      - --reasoning-parser
      - qwen3
      - --tool-call-parser
      - qwen3_xml
      - --enable-auto-tool-choice

    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]

    restart: unless-stopped

```

## "Qwen-Coder" (Qwen2.5-Coder-14B-Instruct-NVFP4-anima) /qwen-coder/docker-compose.yml


```bash
services:
  qwen-coder:
    image: vllm/vllm-openai:v0.26.0
    container_name: qwen-coder-nvfp4
    restart: unless-stopped

    ports:
      - "8001:8000"

    environment:
      HF_TOKEN: ${HF_TOKEN}
      VLLM_ALLOW_LONG_MAX_MODEL_LEN: "1"

    volumes:
      - ~/.cache/huggingface:/root/.cache/huggingface

    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]

    ipc: host

    command:
      - --model
      - ilessio-aiflowlab/Qwen2.5-Coder-14B-Instruct-NVFP4-anima
      - --served-model-name
      - qwen-coder
      - --trust-remote-code
      - --enable-auto-tool-choice
      - --tool-call-parser
      - hermes
      - --max-model-len
      - "65536"
      - --hf-overrides
      - '{"rope_type":"yarn","factor":2.0,"original_max_position_embeddings":32768}'
      - --gpu-memory-utilization
      - "0.30"
      - --max-num-seqs
      - "2"

```
