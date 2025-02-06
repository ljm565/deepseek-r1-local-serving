# Use the official PyTorch 2.5.1 image
FROM pytorch/pytorch:2.5.1-cuda12.4-cudnn9-devel

# Set the working directory
WORKDIR /workspace/llm

# Install C extensions, git, etc.
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    git \
    vim \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*
