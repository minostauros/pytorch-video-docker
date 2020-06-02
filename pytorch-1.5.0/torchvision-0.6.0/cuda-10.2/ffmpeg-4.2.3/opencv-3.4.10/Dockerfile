# ffmpeg - http://ffmpeg.org/download.html
#
# From https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu
#
# ffmpeg docker - https://hub.docker.com/r/jrottenberg/ffmpeg/
#
# opencv docker - https://github.com/Valian/docker-python-opencv-ffmpeg


# Associated pytorch version will not be set automatically. Scroll down and set appropriate versions.

FROM    jrottenberg/ffmpeg:4.2.3-nvidia1804

# Install OpenCV

ENV OPENCV_VERSION 3.4.10
WORKDIR    /

# Install all dependencies for OpenCV
RUN apt-get -y update && \
    apt-get -y install \
        python3 \
        python3-dev \
        git \
        wget \
        unzip \
        cmake \
        build-essential \
        pkg-config \
        libatlas-base-dev \
        gfortran \
        libjasper-dev \
        libgtk2.0-dev \
        libavcodec-dev \
        libavformat-dev \
        libswscale-dev \
        libjpeg-dev \
        libpng-dev \
        libtiff-dev \
        libjasper-dev \
        libv4l-dev 


# install python dependencies
RUN    wget https://bootstrap.pypa.io/get-pip.py && \
    python3 get-pip.py && \
    rm get-pip.py && \
    pip3 install --upgrade pip && \
    pip3 install numpy 


# Install OpenCV
RUN    wget https://github.com/opencv/opencv/archive/$OPENCV_VERSION.zip -O opencv3.zip && \
    unzip -q opencv3.zip && \
    mv /opencv-$OPENCV_VERSION /opencv && \
    rm opencv3.zip && \
    wget https://github.com/opencv/opencv_contrib/archive/$OPENCV_VERSION.zip -O opencv_contrib3.zip && \
    unzip -q opencv_contrib3.zip && \
    mv /opencv_contrib-$OPENCV_VERSION /opencv_contrib && \
    rm opencv_contrib3.zip 


# Prepare build
RUN    mkdir /opencv/build && cd /opencv/build && \
    cmake -D CMAKE_BUILD_TYPE=RELEASE \
      -D BUILD_PYTHON_SUPPORT=ON \
      -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D OPENCV_EXTRA_MODULES_PATH=/opencv_contrib/modules \
      -D BUILD_EXAMPLES=OFF \
      -D PYTHON_DEFAULT_EXECUTABLE=/usr/bin/python3 \
      -D BUILD_opencv_python3=ON \
      -D BUILD_opencv_python2=OFF \
      -D WITH_IPP=OFF \
      -D WITH_FFMPEG=ON \
      -D WITH_CUDA=ON \
      -D ENABLE_FAST_MATH=1 \
      -D CUDA_FAST_MATH=1 \
      -D CUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda \
      -D WITH_CUBLAS=ON \
      -D WITH_V4L=ON .. 


# Install
RUN    cd /opencv/build && \
    make -j$(nproc) && \
    make install && \
    ldconfig 

# Clean
RUN  apt-get clean && \
     rm -rf /opencv /opencv_contrib /var/lib/apt/lists/*

# Installing FFMPEG and OpenCV finished
# Following is packages for custom use

# PyTorch (Mind cuda version and associated torch and torchvision versions)
RUN pip3 install torch==1.5.0 torchvision==0.6.0 torchaudio==0.5.0 -f https://download.pytorch.org/whl/cu102/torch_stable.html

# python packages for running FlowNet2.0 and install tensorflow
RUN pip3 install cffi tensorboardX tensorflow tensorboard tqdm scikit-image colorama==0.3.7 setproctitle pytz

RUN apt-get update && apt-get install -y --no-install-recommends \
        rsync \
        htop \
        openssh-server \
        libpng-dev \
        software-properties-common \
        swig && \
        apt-get autoremove -y && \
        apt-get clean -y && \
        rm -rf /var/lib/apt/lists/*

# python packages for minostauros
RUN pip3 install h5py munkres sklearn scikit-learn visdom youtube_dl
RUN pip3 install matplotlib nibabel pypng moviepy Pillow

# set python3 as default
RUN rm -rf /usr/bin/python && ln -s /usr/bin/python3.5 /usr/bin/python

#RUN sed -i.bak -e "s/== TensorFlow/== MINOSTAUROS/g" \
#               -e "s/================/=================/g" \
#                    /usr/local/bin/nvidia_entrypoint.sh
CMD  ["/bin/bash"]
ENV         LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/lib"
WORKDIR /workspace