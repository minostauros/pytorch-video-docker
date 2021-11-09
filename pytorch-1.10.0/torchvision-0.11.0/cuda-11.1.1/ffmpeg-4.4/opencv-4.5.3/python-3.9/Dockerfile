# ffmpeg - http://ffmpeg.org/download.html
#
# From https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu
#
# ffmpeg docker - https://hub.docker.com/r/jrottenberg/ffmpeg/
#
# opencv docker - https://github.com/Valian/docker-python-opencv-ffmpeg


# Associated pytorch version will not be set automatically. Scroll down and set appropriate versions.

FROM    tigerdockermediocore/cuda-ffmpeg-opencv-docker:4.5.3-cu111-py39

ARG PYTHON_VERSION=3.9
ARG PYTORCH_VERSION=1.10.0
ARG TORCHVISION_VERSION=0.11.0
ARG TORCHAUDIO_VERSION=0.10.0

# environmental variable for apex
# https://en.wikipedia.org/wiki/CUDA#GPUs_supported
# https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#features-and-technical-specifications
# https://github.com/NVIDIA/apex/blob/d6b5ae5d04f531ff862f651e67f241fef88fd159/setup.py#L37
# Kepler(partial) to Ampere (6.0-8.6) with CUDA 11.1.1
ENV TORCH_CUDA_ARCH_LIST=3.5;3.7;5.0;5.2;6.0;6.1;6.2;7.0;7.5;8.0;8.6
ENV TZ=Asia/Seoul

# some apt packages
RUN echo "Asia/Seoul" > /etc/timezone && \
    apt-get update && \
    DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true apt-get install -y --no-install-recommends \
        rsync \
        htop \
        tmux \
        nano \
        net-tools \
        curl \
        openssh-server \
        #libpng-dev \ # included in base image
        swig \
        locales \
        tzdata \
        psmisc \
        git-lfs \
        ninja-build \
        python${PYTHON_VERSION}-dev && \
        apt-get clean -y && \
        rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
# Language & Timezone setting
    echo "Asia/Seoul" > /etc/timezone && \
    ln -sf /usr/share/zoneinfo/${TZ} /etc/localtime && \
    DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true dpkg-reconfigure -f noninteractive tzdata && \
    sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
    echo 'LANG="en_US.UTF-8"'>/etc/default/locale && \
    DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true dpkg-reconfigure --frontend=noninteractive locales && \
    update-locale LANG=en_US.UTF-8 && \
# PyTorch (Mind cuda version and associated torch and torchvision versions)
    python${PYTHON_VERSION} -m pip install --upgrade pip && \
    pip${PYTHON_VERSION} install torch==${PYTORCH_VERSION} torchvision==${TORCHVISION_VERSION} torchaudio==${TORCHAUDIO_VERSION} -f https://download.pytorch.org/whl/cu111/torch_stable.html && \
# python packages for running FlowNet2.0 and install tensorboard
    pip${PYTHON_VERSION} install cffi tensorboardX tensorboard tqdm scikit-image colorama==0.3.7 setproctitle pytz && \
# python packages for minostauros
    pip${PYTHON_VERSION} install h5py munkres sklearn scikit-learn visdom youtube_dl matplotlib nibabel pypng moviepy Pillow einops torchinfo && \
    pip${PYTHON_VERSION} install timm decord simplejson av psutil && \
# nvidia apex
    DIR=/tmp/apex && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    git clone https://github.com/minostauros/apex . && \
    pip install -v --disable-pip-version-check --no-cache-dir --global-option="--cpp_ext" --global-option="--cuda_ext" ./ && \
    rm -rf ${DIR} && \
    cd /tmp/ && \
# detection related
# detectron2 includes fvcore iopath hydra-core omegaconf yacs termcolor pycocotools
    python${PYTHON_VERSION} -m pip install 'git+https://github.com/facebookresearch/detectron2.git' && \
# remove pip cache
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /root/.cache/pip/*


ENV LANG="en_US.UTF-8" \
    LANGUAGE="en_US.UTF-8" \
    LC_ALL="en_US.UTF-8" \
    PATH="/usr/local/cuda-11.1/bin:${PATH}" \
    LD_LIBRARY_PATH="/usr/local/cuda-11.1/lib64:${LD_LIBRARY_PATH}"

CMD  ["/bin/bash"]
WORKDIR /workspace
