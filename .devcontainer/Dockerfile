FROM mcr.microsoft.com/vscode/devcontainers/miniconda:0-3

# Make this an NVIDIA image
ENV NVARCH x86_64
ENV CUDA_VERSION 11.5.0
ENV NV_CUDA_COMPAT_PACKAGE cuda-compat-11-5
ENV NV_CUDA_CUDART_VERSION 11.5.50-1
ENV NVIDIA_REQUIRE_CUDA "cuda>=11.5"

RUN apt-get update && apt-get install -y --no-install-recommends \
    gnupg2 curl ca-certificates \
    && curl -fsSL https://developer.download.nvidia.com/compute/cuda/repos/debian11/${NVARCH}/3bf863cc.pub | gpg --dearmor > /usr/share/keyrings/cuda-archive-keyring.gpg \
    && echo "deb [signed-by=/usr/share/keyrings/cuda-archive-keyring.gpg] https://developer.download.nvidia.com/compute/cuda/repos/debian11/${NVARCH} /" > /etc/apt/sources.list.d/cuda.list \
    && apt-get purge --autoremove -y curl \
    && rm -rf /var/lib/apt/lists/*

# For libraries in the cuda-compat-* package: https://docs.nvidia.com/cuda/eula/index.html#attachment-a
RUN apt-get update && apt-get install -y --no-install-recommends \
    cuda-cudart-11-5=${NV_CUDA_CUDART_VERSION} \
    ${NV_CUDA_COMPAT_PACKAGE} \
    && ln -s cuda-11.5 /usr/local/cuda && \
    rm -rf /var/lib/apt/lists/*

# Required for nvidia-docker v1
RUN echo "/usr/local/nvidia/lib" >> /etc/ld.so.conf.d/nvidia.conf \
    && echo "/usr/local/nvidia/lib64" >> /etc/ld.so.conf.d/nvidia.conf

ENV PATH /usr/local/nvidia/bin:/usr/local/cuda/bin:${PATH}
ENV LD_LIBRARY_PATH /usr/local/nvidia/lib:/usr/local/nvidia/lib64

# nvidia-container-runtime
ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES compute,utility

RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends \
    git-lfs \
    libfile-homedir-perl \
    liblog-dispatch-perl \
    libyaml-tiny-perl \
    texlive-full \
    vim \
    zsh \
    && rm -rf /var/lib/apt/lists/*

# Copy environment.yml (if found) to a temp location so we update the environment. Also
# copy "noop.txt" so the COPY instruction does not fail if no environment.yml exists.
COPY environment.yml* .devcontainer/noop.txt /tmp/conda-tmp/
RUN if [ -f "/tmp/conda-tmp/environment.yml" ]; then \
    umask 0002 \
    && /opt/conda/bin/conda install -n base -c conda-forge mamba \
    && /opt/conda/condabin/mamba env update -n base -f /tmp/conda-tmp/environment.yml; \
    fi \
    && rm -rf /tmp/conda-tmp
