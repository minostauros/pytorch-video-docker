# ffmpeg - http://ffmpeg.org/download.html
#
# From https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu
#
# ffmpeg docker - https://hub.docker.com/r/jrottenberg/ffmpeg/
#
# opencv docker - https://github.com/Valian/docker-python-opencv-ffmpeg


# Associated pytorch version will not be set automatically. Scroll down and set appropriate versions.

FROM    ghcr.io/minostauros/cuda-ffmpeg-opencv-docker:3.4.10-cu92-py37

ARG PYTHON_VERSION=3.7
ARG PYTORCH_VERSION=1.8.1
ARG TORCHVISION_VERSION=0.9.1
ARG TORCHAUDIO_VERSION=0.8.1

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
        libpng-dev \
        swig \
        locales \
        tzdata \
        psmisc \
        git-lfs \
        python${PYTHON_VERSION}-dev && \
        apt-get clean -y && \
        rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
# Language & Timezone setting
    echo "Asia/Seoul" > /etc/timezone && \
    DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true dpkg-reconfigure -f noninteractive tzdata && \
    sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
    echo 'LANG="en_US.UTF-8"'>/etc/default/locale && \
    DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true dpkg-reconfigure --frontend=noninteractive locales && \
    update-locale LANG=en_US.UTF-8 && \
# PyTorch (Mind cuda version and associated torch and torchvision versions)
    python${PYTHON_VERSION} -m pip install --upgrade pip && \
    pip${PYTHON_VERSION} install torch==${PYTORCH_VERSION} torchvision==${TORCHVISION_VERSION} torchaudio==${TORCHAUDIO_VERSION} -f https://download.pytorch.org/whl/cu92/torch_stable.html && \
# python packages for running FlowNet2.0 and install tensorflow
    pip${PYTHON_VERSION} install cffi tensorboardX tensorflow tensorboard tqdm scikit-image colorama==0.3.7 setproctitle pytz && \
# codeserver (VSCode)
    curl -fsSL https://code-server.dev/install.sh | sh && \
# starship
    sh -c "$(curl -fsSL https://starship.rs/install.sh)" -- -y && \
    echo -e "\n# starship\neval \"\$(starship init bash)\"\n" >> ~/.bashrc && \
# python packages for minostauros
    pip${PYTHON_VERSION} install h5py munkres sklearn scikit-learn visdom youtube_dl matplotlib nibabel pypng moviepy Pillow einops easydict && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /root/.cache/pip/*


ENV LANG="en_US.UTF-8" \
    LANGUAGE="en_US.UTF-8" \
    LC_ALL="en_US.UTF-8"

CMD  ["/bin/bash"]
WORKDIR /workspace
