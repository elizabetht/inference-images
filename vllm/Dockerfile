FROM nvidia/cuda:13.0.2-cudnn-devel-ubuntu24.04

# Install build essentials and runtime dependencies
RUN --mount=type=cache,target=/var/cache/apt \
    --mount=type=cache,target=/var/lib/apt \
    apt-get update && apt-get install -y \
    python3.12 python3.12-dev python3.12-venv python3-pip \
    git wget patch curl ca-certificates cmake build-essential ninja-build \
    gcc-aarch64-linux-gnu g++-aarch64-linux-gnu \
    && rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /app

# Create virtual env
RUN python3.12 -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Upgrade pip
RUN --mount=type=cache,target=/root/.cache/pip /opt/venv/bin/pip install --upgrade pip

# Set essential environment variables
ENV TORCH_CUDA_ARCH_LIST="12.1a"
ENV TRITON_PTXAS_PATH=/usr/local/cuda/bin/ptxas
ENV CUDA_HOME=/usr/local/cuda
ENV LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
# vLLM cu130 wheels link against libcudart.so.12 but CUDA 13.0 ships libcudart.so.13
RUN ln -s /usr/local/cuda/lib64/libcudart.so.13 /usr/local/cuda/lib64/libcudart.so.12
ENV TORCH_USE_CUDA_DSA=0

# Install vLLM from GitHub release (cu130 aarch64).
# Use pytorch cu130 as primary index so torch resolves to the exact CUDA build
# vLLM was compiled against; pypi.org as extra for everything else.
RUN --mount=type=cache,target=/root/.cache/pip \
    /opt/venv/bin/pip install \
      https://github.com/vllm-project/vllm/releases/download/v0.18.0/vllm-0.18.0+cu130-cp38-abi3-manylinux_2_35_aarch64.whl \
      --index-url https://download.pytorch.org/whl/cu130 \
      --extra-index-url https://pypi.org/simple

# Install Ray explicitly — the cu130 vLLM wheel omits it, but PP=2 over Ray requires it
RUN --mount=type=cache,target=/root/.cache/pip \
    /opt/venv/bin/pip install "ray[default]>=2.9,!=2.10.0"

# Install additional inference deps (torch already pinned by vLLM above)
RUN --mount=type=cache,target=/root/.cache/pip \
    /opt/venv/bin/pip install xgrammar triton && \
    /opt/venv/bin/pip install -U --pre flashinfer-python --index-url https://flashinfer.ai/whl/nightly --no-deps && \
    /opt/venv/bin/pip install flashinfer-python && \
    /opt/venv/bin/pip install -U --pre flashinfer-cubin --index-url https://flashinfer.ai/whl/nightly && \
    /opt/venv/bin/pip install -U --pre flashinfer-jit-cache --index-url https://flashinfer.ai/whl/cu130 && \
    /opt/venv/bin/pip install torchao>=0.14.0

# Pre-cache HarmonyGptOss vocab (identical to o200k_base — confirmed same SHA256)
RUN mkdir -p /opt/tiktoken-cache && \
    wget -q "https://openaipublic.blob.core.windows.net/encodings/o200k_base.tiktoken" \
         -O /opt/tiktoken-cache/HarmonyGptOss.tiktoken && \
    cp /opt/tiktoken-cache/HarmonyGptOss.tiktoken \
       "/opt/tiktoken-cache/446a9538cb6c348e3516120d7c08b09f57c36495e2acfffe59a5bf8b0cfb1a2d" && \
    echo "446a9538cb6c348e3516120d7c08b09f57c36495e2acfffe59a5bf8b0cfb1a2d  /opt/tiktoken-cache/HarmonyGptOss.tiktoken" | sha256sum -c

ENV TIKTOKEN_RS_CACHE_DIR=/opt/tiktoken-cache
ENV TIKTOKEN_ENCODINGS_BASE=http://127.0.0.1:18889

# Clean up build artifacts
RUN rm -rf /tmp/*

# Set environment
ENV PATH="/opt/venv/bin:$PATH"
ENV NVIDIA_VISIBLE_DEVICES=all
ENV NVIDIA_DRIVER_CAPABILITIES=compute,utility

EXPOSE 8000

ENTRYPOINT ["vllm", "serve"]
