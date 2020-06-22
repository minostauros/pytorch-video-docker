# ffmpeg - http://ffmpeg.org/download.html
#
# From https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu
#
# ffmpeg docker - https://hub.docker.com/r/jrottenberg/ffmpeg/
#
# opencv docker - https://github.com/Valian/docker-python-opencv-ffmpeg


# Associated pytorch version will not be set automatically. Scroll down and set appropriate versions.

FROM    tigerdockermediocore/cuda-ffmpeg-opencv-docker:3.4.10-cu92-py37

ARG PYTHON_VERSION=3.7

# PyTorch (Mind cuda version and associated torch and torchvision versions)
RUN pip${PYTHON_VERSION} install torch==1.5.0 torchvision==0.6.0 torchaudio==0.5.0 -f https://download.pytorch.org/whl/cu101/torch_stable.html && \
# python packages for running FlowNet2.0 and install tensorflow
RUN pip${PYTHON_VERSION} install cffi tensorboardX tensorflow tensorboard tqdm scikit-image colorama==0.3.7 setproctitle pytz && \
# some apt packages
    apt-get update && apt-get install -y --no-install-recommends \
        rsync \
        htop \
        openssh-server \
        libpng-dev \
        software-properties-common \
        swig && \
        apt-get autoremove -y && \
        apt-get clean -y && \
        rm -rf /var/lib/apt/lists/* && \
# python packages for minostauros
    pip${PYTHON_VERSION} install h5py munkres sklearn scikit-learn visdom youtube_dl \
                                 matplotlib nibabel pypng moviepy Pillow && \
# Language & Timezone setting
    echo "Asia/Seoul" > /etc/timezone && \
    dpkg-reconfigure -f noninteractive tzdata && \
    sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
    echo 'LANG="en_US.UTF-8"'>/etc/default/locale && \
    dpkg-reconfigure --frontend=noninteractive locales && \
    update-locale LANG=en_US.UTF-8

ENV LANG en_US.UTF-8 \
    LANGUAGE en_US.UTF-8 \
    LC_ALL en_US.UTF-8 \
    LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/lib"

CMD  ["/bin/bash"]
WORKDIR /workspace
