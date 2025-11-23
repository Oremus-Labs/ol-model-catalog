# Oremus Labs Model Catalog

This repository contains model configuration files for the Oremus Labs AI platform.

## Structure

- `models/` - Model configuration JSON files
- `schema/` - JSON schemas for model configurations (optional)

## Model Configuration Format

Each model is defined in a JSON file with the following structure:

```json
{
  "id": "unique-model-id",
  "displayName": "Human Readable Name",
  "hfModelId": "HuggingFace/Model-ID",
  "runtime": "kserve-runtime-name",
  "nodeSelector": {
    "kubernetes.io/hostname": "target-node"
  },
  "tolerations": [...],
  "resources": {
    "requests": {...},
    "limits": {...}
  },
  "vllm": {
    "dtype": "float16",
    "gpuMemoryUtilization": 0.9,
    "maxModelLen": 4096,
    "tensorParallelSize": 1
  }
}
```

## Usage

The model-manager service reads these configurations and dynamically creates/updates KServe InferenceServices based on API requests.

### Local weight storage conventions

- Set `storageUri` to the PVC (`pvc://venus-model-storage`) that the model-manager uses for cached HuggingFace downloads.
- Install weights via the `/weights/install` endpoint; it will return the directory name (e.g. `qwen2.5-0.5b-instruct`).
- Point the `MODEL_ID` env var (or equivalent) to `/mnt/models/<directory-name>` so the runtime loads the cached copy rather than redownloading from HuggingFace.
