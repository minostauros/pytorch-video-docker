# ffmpeg - http://ffmpeg.org/download.html
#
# From https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu
#
# ffmpeg docker - https://hub.docker.com/r/jrottenberg/ffmpeg/
#
# opencv docker - https://github.com/Valian/docker-python-opencv-ffmpeg


# Associated pytorch version will not be set automatically. Scroll down and set appropriate versions.

FROM    ghcr.io/minostauros/cuda-ffmpeg-opencv-docker:4.6.0-cu116-py39

ARG PYTHON_VERSION=3.9
ARG PYTORCH_VERSION=1.13.0
ARG TORCHVISION_VERSION=0.14.0
ARG TORCHAUDIO_VERSION=0.13.0
ARG TARGET_CUDA_VERSION=cu116
ARG CUDA_VERSION=11.6
# TensorRT for cuda11.8 supports cuda11.x
ARG TENSORRT_VERSION="8.6.1.6-1+cuda11.8"
ARG ONNXRUNTIME_VERSION=1.15.1

# environmental variable for apex
# https://en.wikipedia.org/wiki/CUDA#GPUs_supported
# https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#features-and-technical-specifications
# https://github.com/NVIDIA/apex/blob/d6b5ae5d04f531ff862f651e67f241fef88fd159/setup.py#L37
# Kepler(Late) to Ampere (3.5-8.7) with CUDA 11.6.1
ENV TORCH_CUDA_ARCH_LIST=3.5;3.7;5.0;5.2;6.0;6.1;6.2;7.0;7.5;8.0;8.6;8.7
ENV CMAKE_CUDA_ARCHITECTURES=35;37;50;52;60;61;62;70;75;80;86;87
ENV TZ=Asia/Seoul

# some apt packages
RUN echo "Asia/Seoul" > /etc/timezone && \
    apt-get update && \
    DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true apt-get install -y --no-install-recommends \
        bc \
        libncurses5-dev \
        libncursesw5-dev \
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
        rclone \
        libdrm-dev libsystemd-dev \
        python${PYTHON_VERSION}-dev && \
        apt-get clean -y && \
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
    pip${PYTHON_VERSION} install torch==${PYTORCH_VERSION}+${TARGET_CUDA_VERSION} torchvision==${TORCHVISION_VERSION}+${TARGET_CUDA_VERSION} torchaudio==${TORCHAUDIO_VERSION}+${TARGET_CUDA_VERSION} --extra-index-url https://download.pytorch.org/whl/${TARGET_CUDA_VERSION} && \
# CMake higher version than apt-get
    pip${PYTHON_VERSION} install cmake packaging && \
# python packages for running FlowNet2.0 and install tensorboard
    pip${PYTHON_VERSION} install cffi tensorboardX tensorboard torch_tb_profiler tqdm scikit-image colorama==0.3.7 setproctitle pytz && \
# python packages for minostauros
    pip${PYTHON_VERSION} install h5py munkres scikit-learn visdom youtube_dl matplotlib nibabel pypng moviepy Pillow einops torchinfo && \
    pip${PYTHON_VERSION} install timm decord simplejson av psutil fire fairscale ftfy && \
# nvidia apex
    DIR=/tmp/apex && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    git clone https://github.com/minostauros/apex . && \
    pip${PYTHON_VERSION} install -v --disable-pip-version-check --no-cache-dir --no-build-isolation --global-option="--cpp_ext" --global-option="--cuda_ext" ./ && \
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
        onnx onnxsim peft loralib && \
# attention accelerations
    pip${PYTHON_VERSION} install flash-attn --no-build-isolation && \
# TensorRT libraries & onnxruntime with tensorrt
# onnxruntime: --disable_types float8 for CUDA < 11.8
    apt list -a libnvinfer8 && \
    apt-get install -y libnvinfer8=${TENSORRT_VERSION} libnvinfer-plugin8=${TENSORRT_VERSION} \
        libnvinfer-dev=${TENSORRT_VERSION} libnvinfer-plugin-dev=${TENSORRT_VERSION} \
        libnvonnxparsers8=${TENSORRT_VERSION} libnvonnxparsers-dev=${TENSORRT_VERSION} \
        libnvparsers8=${TENSORRT_VERSION}  libnvparsers-dev=${TENSORRT_VERSION} \
        libnvinfer-headers-dev=${TENSORRT_VERSION} libnvinfer-headers-plugin-dev=${TENSORRT_VERSION} \
        libnvinfer-bin=${TENSORRT_VERSION} libnvinfer-dispatch-dev=${TENSORRT_VERSION} \
        libnvinfer-dispatch8=${TENSORRT_VERSION} libnvinfer-lean-dev=${TENSORRT_VERSION} \
        libnvinfer-lean8=${TENSORRT_VERSION} libnvinfer-samples=${TENSORRT_VERSION} \
        libnvinfer-vc-plugin-dev=${TENSORRT_VERSION} libnvinfer-vc-plugin8=${TENSORRT_VERSION} \
        tensorrt=${TENSORRT_VERSION} && \
    pip${PYTHON_VERSION} install tensorrt && \
    pip${PYTHON_VERSION} install onnxruntime-gpu==${ONNXRUNTIME_VERSION} && \
    # DIR=/tmp/onnxruntime && \
    # mkdir -p ${DIR} && \
    # cd ${DIR} && \
    # git clone --recursive --branch v${ONNXRUNTIME_VERSION} https://github.com/Microsoft/onnxruntime . && \
    # export CUDACXX=/usr/local/cuda-${CUDA_VERSION}/bin/nvcc && \
    # git config --add safe.directory ${DIR} && \
    # bash build.sh --config Release  --build_shared_lib --parallel \
    #     --use_cuda --cuda_version "${CUDA_VERSION}" \
    #     --cuda_home /usr/local/cuda-${CUDA_VERSION} \
    #     --cudnn_home /usr/include/ \
    #     --build_wheel --skip_tests \
    #     --use_tensorrt --tensorrt_home /usr/src/tensorrt/ \
    #     --cmake_extra_defines "onnxruntime_BUILD_UNIT_TESTS=OFF" \
    #     --cmake_extra_defines "CMAKE_CUDA_ARCHITECTURES=${CMAKE_CUDA_ARCHITECTURES}" \
    #     --allow_running_as_root && \
    # pip${PYTHON_VERSION} install build/Linux/Release/dist/onnxruntime_gpu-*.whl && \
    # rm -rf ${DIR} && \
    # cd /tmp/ && \
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
