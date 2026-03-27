# inference-images

Container image builds for the TokenLabs inference stack, targeting NVIDIA GB10 (DGX Spark, CUDA 13.0 / SM_121a).

## Images

| Image | Tag pattern | Registry |
|-------|-------------|----------|
| vLLM serve | `vllm-v<version>` | `ghcr.io/elizabetht/inference-images/vllm-serve` |
| SGLang serve | `sglang-v<version>` | `ghcr.io/elizabetht/inference-images/sglang-serve` |

## Building

Push a tag to trigger a build. Only the matching job runs.

```bash
# Build vLLM image
git tag vllm-v0.16.0 && git push origin vllm-v0.16.0

# Build SGLang image
git tag sglang-v0.4.0 && git push origin sglang-v0.4.0
```

## Structure

```
inference-images/
├── vllm/Dockerfile       # vLLM + cu130 + Ray + modelopt
├── sglang/Dockerfile     # SGLang (placeholder)
└── .github/workflows/
    └── build-and-push.yml
```

## Hardware

Builds run on a self-hosted `DGX-Spark` runner (NVIDIA GB10, aarch64, CUDA 13.0).
