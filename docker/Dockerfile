#FROM ros:melodic
FROM ros:noetic

ENV TZ=America/New_York
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Consolidate apt-get commands into a single RUN layer to minimize image size and properly clear cache
# Consolidate apt-get commands into a single RUN layer to minimize image size and properly clear cache
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -q -y --no-install-recommends \
      apt-utils \
      net-tools iproute2 iputils-ping \
      git tmux vim neovim ca-certificates \
      build-essential unzip pkg-config autoconf \
      libboost-all-dev \
      libjpeg-dev libpng-dev libtiff-dev \
      libvtk7-dev libgtk-3-dev \
      libatlas-base-dev gfortran \
      libparmetis-dev \
      python3-wstool python3-catkin-tools python3-pip \
      ros-noetic-desktop-full \
      ros-noetic-image-geometry ros-noetic-pcl-ros ros-noetic-cv-bridge \
      libtbb-dev \
      libtool \
      opencl-headers ocl-icd-opencl-dev \
      tmuxp python3-vcstool \
      wget && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN pip3 install --upgrade cmake==3.26.4

ENV ROS_DIR=/opt/ros/${ROS_DISTRO}
ENV HOME=/root/
ENV WORKSPACE=/root/catkin_ws
RUN mkdir -p $WORKSPACE/src

WORKDIR $WORKSPACE/src

# Install kimera-multi dependencies
RUN git clone https://github.com/MIT-SPARK/Kimera-Multi.git
RUN bash Kimera-Multi/install/dependencies.sh

# Clone source code
COPY "./copy/kimera_multi_https.repos" .

# FIX: Tell Git to trust the directory to prevent "dubious ownership" exit code 1 errors
RUN git config --global --add safe.directory '*'
RUN vcs import . --input kimera_multi_https.repos --recursive

# Apply patches
COPY "./copy/kimera_multi_lcd.patch" /root
COPY "./copy/kimera_semantics.patch" /root
WORKDIR $WORKSPACE/src/kimera_multi_lcd
RUN git apply /root/kimera_multi_lcd.patch
WORKDIR $WORKSPACE/src/kimera_semantics
RUN git apply /root/kimera_semantics.patch

# catkin build
SHELL ["/bin/bash", "-c"]
WORKDIR ${WORKSPACE}
RUN catkin init
RUN catkin config -a --cmake-args -DCMAKE_BUILD_TYPE=RelWithDebInfo -DGTSAM_TANGENT_PREINTEGRATION=OFF -DGTSAM_BUILD_WITH_MARCH_NATIVE=OFF -DOPENGV_BUILD_WITH_MARCH_NATIVE=OFF
RUN catkin config --merge-devel
# RUN source $ROS_DIR/setup.bash && catkin build -c -s -j6
RUN source $ROS_DIR/setup.bash && catkin build -c -s -j2; exit 0

# Download vocabulary file
WORKDIR ${WORKSPACE}/src/kimera_multi_lcd
RUN pip install gdown
RUN gdown 'https://drive.google.com/uc?export=download&confirm=9iBg&id=1N4y0HbgA3PHQ73ZxFJvy5dgvV_0cTBYF'
RUN unzip -o vocab.zip

WORKDIR /root

COPY "./copy/entrypoint.bash" /
ENTRYPOINT ["/entrypoint.bash"]