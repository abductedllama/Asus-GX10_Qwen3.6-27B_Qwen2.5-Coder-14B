# ~~Asus-GX10_Qwen3.6-27B_Qwen2.5-Coder-14B~~  

I stopped running this setup and am now running qwen3.6 35b a3b + qwen3.8 28b dense on the asus gx10. will make a new repo soon with the config  

~~vLLM docker compose configs for Qwen Models on Asus GX10~~

~~Running two models on **ASUS GX10** (ARM64) via Docker Compose.~~

~~This may or may not be optimal. I was just trying to get two decently large models with a large context, running smoothly under any request.~~

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
Mem:           121Gi        88Gi       2.1Gi       191Mi        32Gi        32Gi
Swap:           15Gi       1.2Mi        15Gi

```

> **WIP** — initial setup, more to follow.

20260804 - Removed docker compose files from README, docker compose files uploaded are current, dont want to update in 2 places, Updated memory for new settings. <br>
20260803 - Updated compose files for both containers. Reason: use slightly less resources lowered gpu-memory-utilzation, Code: increase --max-num-seqs to 4 for better concurrent requests on QwenCode model
