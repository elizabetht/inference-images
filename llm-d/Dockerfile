FROM ghcr.io/llm-d/llm-d-cuda:v0.6.0

LABEL org.opencontainers.image.version="v1.0.0"
LABEL org.opencontainers.image.description="llm-d v0.6.0 + transformers>=5.5.0 for Gemma 4 support, GB10/sm_121 ARM64"
LABEL org.opencontainers.image.source="https://github.com/elizabetht/inference-images"

RUN python3 -m ensurepip --upgrade && python3 -m pip install --upgrade "transformers>=5.5.0" --no-cache-dir
