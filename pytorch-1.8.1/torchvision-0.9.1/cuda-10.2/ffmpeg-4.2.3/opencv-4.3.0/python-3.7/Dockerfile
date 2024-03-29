# ffmpeg - http://ffmpeg.org/download.html
#
# From https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu
#
# ffmpeg docker - https://hub.docker.com/r/jrottenberg/ffmpeg/
#
# opencv docker - https://github.com/Valian/docker-python-opencv-ffmpeg


# Associated pytorch version will not be set automatically. Scroll down and set appropriate versions.

FROM    ghcr.io/minostauros/cuda-ffmpeg-opencv-docker:4.3.0-cu102-py37

ARG PYTHON_VERSION=3.7
ARG PYTORCH_VERSION=1.8.1
ARG TORCHVISION_VERSION=0.9.1
ARG TORCHAUDIO_VERSION=0.8.1
ARG TARGET_CUDA_VERSION=cu102
ARG CUDA_VERSION=10.2

# environmental variable for apex
# https://en.wikipedia.org/wiki/CUDA#GPUs_supported
# https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#features-and-technical-specifications
# https://github.com/NVIDIA/apex/blob/d6b5ae5d04f531ff862f651e67f241fef88fd159/setup.py#L37
# Kepler(partial) to Turing (3.5-7.5) with CUDA 10.2
ENV TORCH_CUDA_ARCH_LIST=3.5;3.7;5.0;5.2;6.0;6.1;6.2;7.0;7.5

#;some apt packages
RUN echo "Asia/Seoul" > /etc/timezone && \
    apt-get update && \
    DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true apt-get install -y --no-install-recommends \
        cmake \
        libncurses5-dev \
        libncursesw5-dev \
        rsync \
        htop \
        tmux \
        nano \
        net-tools \
        curl \
        openssh-server \
        libpng-dev \
        swig \
        locales \
        tzdata \
        psmisc \
        git-lfs \
        ninja-build \
        rclone \
        libdrm-dev libsystemd-dev libudev-dev \
        python${PYTHON_VERSION}-dev && \
        apt-get clean -y && \
# Language & Timezone setting
    echo "Asia/Seoul" > /etc/timezone && \
    DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true dpkg-reconfigure -f noninteractive tzdata && \
    sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
    echo 'LANG="en_US.UTF-8"'>/etc/default/locale && \
    DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true dpkg-reconfigure --frontend=noninteractive locales && \
    update-locale LANG=en_US.UTF-8 && \
# PyTorch (Mind cuda version and associated torch and torchvision versions)
    python${PYTHON_VERSION} -m pip install --upgrade pip && \
    pip${PYTHON_VERSION} install torch==${PYTORCH_VERSION} torchvision==${TORCHVISION_VERSION} torchaudio==${TORCHAUDIO_VERSION} -f https://download.pytorch.org/whl/${TARGET_CUDA_VERSION}/torch_stable.html && \
# python packages for running FlowNet2.0 and install tensorflow
    pip${PYTHON_VERSION} install cffi tensorboardX tensorflow tensorboard torch_tb_profiler tqdm scikit-image colorama==0.3.7 setproctitle pytz && \
# python packages for minostauros
    pip${PYTHON_VERSION} install h5py munkres scikit-learn visdom youtube_dl matplotlib nibabel pypng moviepy Pillow einops torchinfo && \
    pip${PYTHON_VERSION} install timm decord simplejson av psutil fire fairscale && \
# nvidia apex
    DIR=/tmp/apex && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    git clone https://github.com/minostauros/apex . && \
    pip${PYTHON_VERSION} install -v --disable-pip-version-check --no-cache-dir --global-option="--cpp_ext" --global-option="--cuda_ext" ./ && \
    rm -rf ${DIR} && \
    cd /tmp/ && \
# nvtop
    DIR=/tmp/nvtop && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    git clone https://github.com/Syllo/nvtop.git . && \
    mkdir -p build && \
    cd build && \
    cmake .. && \
    make && \
    make install && \
    rm -rf ${DIR} && \
    cd /tmp/ && \
# codeserver (VSCode)
    curl -fsSL https://code-server.dev/install.sh | sh && \
# starship
    sh -c "$(curl -fsSL https://starship.rs/install.sh)" -- -y && \
    echo -e "\n# starship\neval \"\$(starship init bash)\"\n" >> ~/.bashrc && \
# detection related
# detectron2 includes fvcore iopath hydra-core omegaconf yacs termcolor pycocotools
    python${PYTHON_VERSION} -m pip install 'git+https://github.com/facebookresearch/detectron2.git' && \
    pip${PYTHON_VERSION} install motmetrics easydict && \
# Neural Rendering related
    pip${PYTHON_VERSION} install configargparse lpips && \
# Huggingface (pytorch >= 1.13.0 for bettertransformer)
    pip${PYTHON_VERSION} install transformers tokenizers datasets accelerate \
        optimum huggingface_hub huggingface_hub[cli,torch] diffusers[torch] \
        onnx onnxruntime peft && \
# remove futile apt packages and pip cache
   apt-get -y remove \
        libncurses5-dev \
        libncursesw5-dev \
    && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /root/.cache/pip/*

ENV LANG="en_US.UTF-8" \
    LANGUAGE="en_US.UTF-8" \
    LC_ALL="en_US.UTF-8" \
    PATH="/usr/local/cuda-${CUDA_VERSION}/bin:${PATH}" \
    LD_LIBRARY_PATH="/usr/local/cuda-${CUDA_VERSION}/lib64:${LD_LIBRARY_PATH}" \
    PYTHONPATH="$PYTHONPATH:/usr/lib/python${PYTHON_VERSION}/site-packages:/usr/local/lib/python${PYTHON_VERSION}/site-packages"

CMD  ["/bin/bash"]
WORKDIR /workspace
