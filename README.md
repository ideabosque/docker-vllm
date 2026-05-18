# docker-vllm

Run [vLLM](https://github.com/vllm-project/vllm) in Docker with HuggingFace Hub model loading and a host-mounted model cache.

## Layout

- `docker-compose.yml` — vLLM OpenAI-compatible service with NVIDIA GPU support
- `.env` — runtime configuration (model id, token, ports, etc.)
- `.env.example` — template you can copy
- `./models/` — host directory mounted as the HF cache (created on first run)

## Prerequisites

- Docker + Docker Compose v2
- NVIDIA GPU + recent driver + [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)

## Quick start

1. Copy and edit the env file:

   ```bash
   cp .env.example .env
   # edit MODEL_ID and (if gated) HF_TOKEN
   ```

2. Launch:

   ```bash
   docker compose up -d
   docker compose logs -f vllm
   ```

3. Test the OpenAI-compatible endpoint:

   ```bash
   curl http://localhost:8000/v1/models
   curl http://localhost:8000/v1/chat/completions \
     -H "Content-Type: application/json" \
     -d '{
       "model": "Qwen/Qwen3-4B",
       "messages": [{"role": "user", "content": "Hello"}]
     }'
   ```

   Note: the `model` field in the request must match `MODEL_ID` from `.env`
   (vLLM serves the model under its HuggingFace repo id by default).

## Switching models

Edit `MODEL_ID` in `.env` and restart:

```bash
docker compose up -d --force-recreate
```

Models are cached in `MODELS_DIR` (default `./models`) so subsequent launches skip the download.

## Common knobs (`.env`)

| Variable | Purpose |
|---|---|
| `MODEL_ID` | HuggingFace repo id (e.g. `Qwen/Qwen2.5-7B-Instruct`) |
| `HF_TOKEN` | Token for gated/private models |
| `MODELS_DIR` | Host path mounted into `/root/.cache/huggingface` |
| `HOST_PORT` | Host port mapped to container port 8000 |
| `TP_SIZE` | Tensor-parallel size (number of GPUs) |
| `DTYPE` | `auto`, `bfloat16`, `float16`, etc. |
| `GPU_MEMORY_UTILIZATION` | Fraction of GPU memory vLLM may use (0.0–1.0) |
| `MAX_MODEL_LEN` | Max sequence length (prompt + generated) |
| `TOOL_CALL_PARSER` | e.g. `hermes` for Qwen, `mistral`, `llama3_json` |
| `REASONING_PARSER` | e.g. `qwen3`, `deepseek_r1` |
| `IMAGE_TAG` | Override `vllm/vllm-openai` tag |
