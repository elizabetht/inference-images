FROM ghcr.io/elizabetht/inference-images/vllm-serve:0.19.0

LABEL org.opencontainers.image.version="v2.0.0"
LABEL org.opencontainers.image.description="vLLM v0.19.0 (Gemma 4 support) + NIXL v1.0.0 (PD disagg KV transfer), GB10/sm_121 ARM64"
LABEL org.opencontainers.image.source="https://github.com/elizabetht/inference-images"

# Install NIXL for llm-d disaggregated prefill/decode KV transfer
RUN pip install "nixl==1.0.0" --no-cache-dir
