# OCR Model Performance Tuning Log

Tracking iterative configuration changes and throughput measurements on the Venus node (32 vCPU / 32 GiB RAM / 96 GiB AMD GPU).

## DeepSeek OCR (`deepseek-ai/DeepSeek-OCR`)

| Trial | Max Model Len | GPU Mem Util | Swap (GiB) | CPU Req/Lim | Mem Req/Lim | Extra Args | Notes | Tokens/s |
|-------|---------------|--------------|-----------|-------------|-------------|------------|-------|----------|
| D1 | 2048 | 0.90 | 8 | 6 / 10 | 16 GiB / 24 GiB | `--swap-space 8` | Baseline; avg of 3 runs (max_tokens 512) | ~108 tok/s |

## PaddleOCR-VL (`PaddlePaddle/PaddleOCR-VL`)

| Trial | Max Model Len | GPU Mem Util | Swap (GiB) | CPU Req/Lim | Mem Req/Lim | Extra Args | Notes | Tokens/s |
|-------|---------------|--------------|-----------|-------------|-------------|------------|-------|----------|

## Tuning Inputs to Consider

Reference from current vLLM ROCm builds (0.11.x). All can be supplied via `vllm` block or `extraArgs`.

### Core Execution
- `tensorParallelSize`
- `pipelineParallelSize` (not exposed yet but via `extraArgs` `--pipeline-parallel-size`)
- `dtype` (`auto`, `float16`, `bfloat16`)
- `--quantization` + `--quantization-param-path`
- `--enforce-eager`, `--disable-custom-all-reduce`
- `--max-context-len-to-capture`, `--max-num-seqs`, `--max-num-batched-tokens`
- `--max-log-len`, `--max-seq-seed`, `--kv-cache-dtype`
- `--device` (`auto`, `cuda`, `hip`, `cpu`)

### Memory / Cache
- `gpuMemoryUtilization`
- `maxModelLen`
- `--swap-space`
- `--max-cpu-memory`
- `--enable-pinned-memory`
- `--enable-chunked-prefill`, `--chunked-prefill-size`
- `--enable-prefix-caching`
- `--disable-log-stats` (reduces CPU overhead)
- `--use-v2-block-manager`, `--paged-kv-cache`, `--gpu-memory-utilization-decay`

### Throughput / SpecDec
- `--speculative-mode` (e.g., EAGLE), `--speculative-model`, `--speculative-multi-step`
- `--best-of`, `--sampler` settings via `extraArgs`
- `--gpu-memory-utilization-decay`, `--kv-cache-freeze-fraction`
- `--max-num-logprobs` if log probs requested frequently

### Deployment
- CPU / Memory requests & limits (stay ≤32 vCPU / 32 GiB host)
- Probe timings (now 120s start)
- Additional env vars (e.g., `VLLM_WORKER_MULTIPROC_METHOD`, HIP tuning flags)

Record every trial row with **all** relevant knobs so we can reproduce when context resets.

Document every change in the tables above (one row per trial) with measured tokens/sec from `https://ai.oremuslabs.app/v1/completions` so we can roll back to the best-known combination quickly.
