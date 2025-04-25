# ffmpeg - http://ffmpeg.org/download.html
#
# From https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu
#
# ffmpeg docker - https://hub.docker.com/r/jrottenberg/ffmpeg/
#
# opencv docker - https://github.com/Valian/docker-python-opencv-ffmpeg


# Associated pytorch version will not be set automatically. Scroll down and set appropriate versions.

FROM    ghcr.io/minostauros/cuda-ffmpeg-opencv-docker:4.10.0-cu121-py310

ARG PYTHON_VERSION=3.10
ARG PYTORCH_VERSION=2.5.1
ARG TORCHVISION_VERSION=0.20.1
ARG TORCHAUDIO_VERSION=2.5.1
ARG XFORMERS_VERSION=0.0.29
ARG TARGET_CUDA_VERSION=cu121
ARG CUDA_VERSION=12.1
# TensorRT for cuda12.0 supports cuda12.x
ARG TENSORRT_VERSION="10.8.0.43-1+cuda12.8"
ARG PIP_TENSORRT=tensorrt-cu12
ARG ONNXRUNTIME_VERSION=1.21.1

# environmental variable for apex
# https://en.wikipedia.org/wiki/CUDA#GPUs_supported
# https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#features-and-technical-specifications
# https://github.com/NVIDIA/apex/blob/d6b5ae5d04f531ff862f651e67f241fef88fd159/setup.py#L37
# Maxwell to Hopper (5.0-9.0) with CUDA 12.1.1
ENV TORCH_CUDA_ARCH_LIST=5.0;5.2;6.0;6.1;6.2;7.0;7.5;8.0;8.6;8.7;8.9;9.0
ENV CMAKE_CUDA_ARCHITECTURES=50;52;60;61;62;70;75;80;86;87;89;90
ENV TZ=Asia/Seoul

# some apt packages
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
        #libpng-dev \ # included in base image
        swig \
        locales \
        tzdata \
        psmisc \
        git-lfs \
        ninja-build \
        rclone \
        iputils-ping \
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
    pip${PYTHON_VERSION} install torch==${PYTORCH_VERSION}+${TARGET_CUDA_VERSION} torchvision==${TORCHVISION_VERSION}+${TARGET_CUDA_VERSION} torchaudio==${TORCHAUDIO_VERSION}+${TARGET_CUDA_VERSION} --index-url https://download.pytorch.org/whl/${TARGET_CUDA_VERSION} && \
# CMake higher version than apt-get
    pip${PYTHON_VERSION} install cmake packaging && \
# python packages for running FlowNet2.0 and install tensorboard
    pip${PYTHON_VERSION} install cffi tensorboardX tensorboard torch_tb_profiler tqdm scikit-image colorama==0.3.7 setproctitle pytz && \
# python packages for minostauros
    pip${PYTHON_VERSION} install virtualenv h5py munkres scikit-learn visdom youtube_dl matplotlib nibabel pypng moviepy Pillow pillow-heif einops torchinfo qdrant-client chromadb && \
    pip${PYTHON_VERSION} install timm decord ffmpeg-python simplejson av psutil fire fairscale ftfy loguru uvicorn fastapi backoff wandb ipympl jupyter aiofiles && \
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
# VLMEvalKit
    pip${PYTHON_VERSION} install sentencepiece validators python-dotenv sty xlsxwriter && \
# Huggingface (pytorch >= 1.13.0 for bettertransformer)
    pip${PYTHON_VERSION} install transformers tokenizers datasets accelerate \
        optimum huggingface_hub huggingface_hub[cli,torch] diffusers[torch] \
        onnx onnxsim peft loralib sentence-transformers && \
# attention accelerations
    pip${PYTHON_VERSION} install flash-attn --no-build-isolation && \
    pip${PYTHON_VERSION} install xformers==${XFORMERS_VERSION} --index-url https://download.pytorch.org/whl/${TARGET_CUDA_VERSION} && \
# TensorRT libraries & onnxruntime with tensorrt
    apt list -a libnvinfer10 && \
    apt-get install -y libnvinfer10=${TENSORRT_VERSION} libnvinfer-plugin10=${TENSORRT_VERSION} \
        libnvonnxparsers10=${TENSORRT_VERSION} && \
        # exclude devel packages to save storage
        # libnvinfer-dev=${TENSORRT_VERSION} libnvinfer-plugin-dev=${TENSORRT_VERSION} \
        # libnvonnxparsers-dev=${TENSORRT_VERSION} \
        # libnvinfer-headers-dev=${TENSORRT_VERSION} libnvinfer-headers-plugin-dev=${TENSORRT_VERSION} \
        # libnvinfer-bin=${TENSORRT_VERSION} libnvinfer-dispatch-dev=${TENSORRT_VERSION} \
        # libnvinfer-dispatch10=${TENSORRT_VERSION} libnvinfer-lean-dev=${TENSORRT_VERSION} \
        # libnvinfer-lean10=${TENSORRT_VERSION} libnvinfer-samples=${TENSORRT_VERSION} \
        # libnvinfer-vc-plugin-dev=${TENSORRT_VERSION} libnvinfer-vc-plugin10=${TENSORRT_VERSION} \
        # python3-libnvinfer=${TENSORRT_VERSION} python3-libnvinfer-lean=${TENSORRT_VERSION} \
        # python3-libnvinfer-dispatch=${TENSORRT_VERSION} python3-libnvinfer-dev=${TENSORRT_VERSION} \
        # tensorrt=${TENSORRT_VERSION} && \
    pip${PYTHON_VERSION} install ${PIP_TENSORRT} && \
    # onnxrumtime-gpu versions: https://aiinfra.visualstudio.com/PublicPackages/_artifacts/feed/onnxruntime-cuda-12/PyPI/onnxruntime-gpu/versions/
    pip${PYTHON_VERSION} install onnxruntime-gpu==${ONNXRUNTIME_VERSION} --extra-index-url https://aiinfra.pkgs.visualstudio.com/PublicPackages/_packaging/onnxruntime-cuda-12/pypi/simple/ && \
    # DIR=/tmp/onnxruntime && \
    # mkdir -p ${DIR} && \
    # cd ${DIR} && \
    # git clone --recursive --branch v${ONNXRUNTIME_VERSION} https://github.com/Microsoft/onnxruntime . && \
    # # git clone --recursive https://github.com/Microsoft/onnxruntime . && \
    # export CUDACXX=/usr/local/cuda-${CUDA_VERSION}/bin/nvcc && \
    # git config --add safe.directory ${DIR} && \
    # bash build.sh --config Release  --build_shared_lib \
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
