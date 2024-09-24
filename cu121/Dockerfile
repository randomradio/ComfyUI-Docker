################################################################################
# Dockerfile that builds 'yanwk/comfyui-boot:cu121'
# A runtime environment for https://github.com/comfyanonymous/ComfyUI
# Using CUDA 12.1 & Python 3.12
################################################################################

FROM opensuse/tumbleweed:latest

LABEL maintainer="YAN Wenkun <code@yanwk.fun>"

RUN set -eu

################################################################################
# Configure to use Chinese mirrors and install Python and tools

RUN --mount=type=cache,target=/var/cache/zypp \
    zypper modifyrepo --disable --all \
    && zypper addrepo --check --refresh --gpgcheck \
    'https://mirrors.tuna.tsinghua.edu.cn/opensuse/tumbleweed/repo/oss/' mirror-oss \
    && zypper addrepo --check --refresh --gpgcheck \
    'https://mirrors.tuna.tsinghua.edu.cn/opensuse/tumbleweed/repo/non-oss/' mirror-non-oss \
    && zypper addrepo --check --refresh --priority 90 \
    'https://mirrors.tuna.tsinghua.edu.cn/packman/suse/openSUSE_Tumbleweed/Essentials/' mirror-packman-essentials \
    && zypper --gpg-auto-import-keys \
    install --no-confirm --auto-agree-with-licenses \
    python312-devel \
    python312-pip \
    python312-wheel \
    python312-setuptools \
    python312-Cython \
    python312-py-build-cmake \
    python312-aiohttp \
    python312-dbm \
    python312-ffmpeg-python \
    python312-GitPython \
    python312-httpx \
    python312-joblib \
    python312-lark \
    python312-matplotlib \
    python312-mpmath \
    python312-numba-devel \
    python312-numpy \
    python312-onnx \
    python312-opencv \
    python312-pandas \
    python312-qrcode \
    python312-rich \
    python312-scikit-build \
    python312-scikit-build-core-pyproject \
    python312-scikit-image \
    python312-scikit-learn \
    python312-scipy \
    python312-svglib \
    python312-tqdm \
    Mesa-libGL1 \
    Mesa-libEGL-devel \
    libgthread-2_0-0 \
    make \
    ninja \
    git \
    aria2 \
    fish \
    fd \
    vim \
    opencv \
    opencv-devel \
    ffmpeg \
    x264 \
    x265 \
    google-noto-sans-fonts \
    google-noto-sans-cjk-fonts \
    google-noto-coloremoji-fonts \
    && rm /usr/lib64/python3.12/EXTERNALLY-MANAGED \
    && update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.12 100

################################################################################
# GCC 13 
# Compatible with CUDA 12.1

RUN --mount=type=cache,target=/var/cache/zypp \
    zypper --gpg-auto-import-keys \
    install --no-confirm --auto-agree-with-licenses \
    gcc13 \
    gcc13-c++ \
    cpp13 \
    && update-alternatives --install /usr/bin/c++ c++ /usr/bin/g++-13 90 \
    && update-alternatives --install /usr/bin/cc  cc  /usr/bin/gcc-13 90 \
    && update-alternatives --install /usr/bin/cpp cpp /usr/bin/cpp-13 90 \
    && update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-13 90 \
    && update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-13 90 \
    && update-alternatives --install /usr/bin/gcc-ar gcc-ar /usr/bin/gcc-ar-13 90 \
    && update-alternatives --install /usr/bin/gcc-nm gcc-nm /usr/bin/gcc-nm-13 90 \
    && update-alternatives --install /usr/bin/gcc-ranlib gcc-ranlib /usr/bin/gcc-ranlib-13 90 \
    && update-alternatives --install /usr/bin/gcov gcov /usr/bin/gcov-13 90 \
    && update-alternatives --install /usr/bin/gcov-dump gcov-dump /usr/bin/gcov-dump-13 90 \
    && update-alternatives --install /usr/bin/gcov-tool gcov-tool /usr/bin/gcov-tool-13 90 

################################################################################
# Configure to use Chinese mirrors, install Python packages

ENV PIP_INDEX_URL="https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple"

# PyTorch, xFormers
RUN --mount=type=cache,target=/root/.cache/pip \
    pip list \
    && pip install \
    --upgrade pip wheel setuptools \
    && pip install \
    xformers torchvision torchaudio \
    --index-url https://download.pytorch.org/whl/cu121 \
    --extra-index-url https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple

# Bind environment variables (dependency library .so files)
ENV LD_LIBRARY_PATH="${LD_LIBRARY_PATH}\
    :/usr/local/lib64/python3.12/site-packages/torch/lib\
    :/usr/local/lib/python3.12/site-packages/nvidia/cuda_cupti/lib\
    :/usr/local/lib/python3.12/site-packages/nvidia/cuda_runtime/lib\
    :/usr/local/lib/python3.12/site-packages/nvidia/cudnn/lib\
    :/usr/local/lib/python3.12/site-packages/nvidia/cufft/lib\
    :/usr/local/lib/python3.12/site-packages/nvidia/cublas/lib\
    :/usr/local/lib/python3.12/site-packages/nvidia/cuda_nvrtc/lib\
    :/usr/local/lib/python3.12/site-packages/nvidia/curand/lib\
    :/usr/local/lib/python3.12/site-packages/nvidia/cusolver/lib\
    :/usr/local/lib/python3.12/site-packages/nvidia/cusparse/lib\
    :/usr/local/lib/python3.12/site-packages/nvidia/nccl/lib\
    :/usr/local/lib/python3.12/site-packages/nvidia/nvjitlink/lib\
    :/usr/local/lib/python3.12/site-packages/nvidia/nvtx/lib"

COPY builder-scripts/.  /builder-scripts/

# 1. Install dependencies for ComfyUI and extensions
# 2. Handle ONNX Runtime error "missing CUDA provider" and add CUDA 12 support, refer to: https://onnxruntime.ai/docs/install/
# 3. Handle MediaPipe's dependency error (requires protobuf<4)
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install \
    -r /builder-scripts/pak3.txt \
    && pip install \
    -r /builder-scripts/pak5.txt \
    && pip install \
    -r /builder-scripts/pak7.txt \
    && pip uninstall --break-system-packages --yes \
    onnxruntime-gpu \
    && pip --no-cache-dir install --break-system-packages \
    onnxruntime-gpu \
    --index-url https://aiinfra.pkgs.visualstudio.com/PublicPackages/_packaging/onnxruntime-cuda-12/pypi/simple/ \
    --extra-index-url https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple \
    && pip install \
    mediapipe \
    && pip list

################################################################################

# Use Chinese mirror for HuggingFace

ENV HF_ENDPOINT=https://hf-mirror.com

################################################################################

RUN du -ah /root \
    && find /root/ -mindepth 1 -delete

COPY scripts/.  /scripts/

USER root
VOLUME /root
WORKDIR /root
EXPOSE 8188
ENV CLI_ARGS=""
CMD ["bash","/scripts/entrypoint.sh"]
