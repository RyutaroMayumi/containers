FROM ubuntu:20.04

ENV DEBIAN_FRONTEND noninteractive
ARG UID=1000
ARG LOCAL_USER=user

# change time zone
ENV TZ Asia/Tokyo
RUN apt-get update \
    && apt-get install -y tzdata \
    && rm -rf /var/lib/apt/lists/* \
    && echo "${TZ}" > /etc/timezone \
    && rm /etc/localtime \
    && ln -s /usr/share/zoneinfo/Asia/Tokyo /etc/localtime \
    && dpkg-reconfigure -f noninteractive tzdata

# install utilities
RUN sed -i.org -e 's|ports.ubuntu.com|jp.archive.ubuntu.com|g' /etc/apt/sources.list
RUN apt-get update && apt-get upgrade -y && apt-get install -y \
    can-utils \
    curl \
    git \
    iproute2 \
    iputils-ping \
    kmod \
    lsb-core \
    sudo \
    v4l-utils \
    vim \
    wget \
    x11-apps \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# install ros
RUN sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
RUN curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add -
RUN apt-get update
RUN apt-get install -y ros-noetic-desktop-full
RUN apt-get install -y python3-pip python3-rosinstall python3-catkin-tools python3-rosdep
# for solving "Website may be down" error during "running rodep init", remove proxy settings in apt.conf and logoff/login.
RUN rosdep init
RUN rosdep update

# install ros packages
# for running gazebo
RUN apt-get install -y \
    gazebo11 \
    libgazebo11-dev \
    ros-noetic-gazebo-ros-pkgs \
    ros-noetic-gazebo-ros-control
# for running mavros, mavros-extras
RUN apt-get install -y \
    ros-noetic-mavros \
    ros-noetic-mavros-extras
# to avoid "This script require root privileges!" error, use sudo in executing sh file
RUN wget https://raw.githubusercontent.com/mavlink/mavros/master/mavros/scripts/install_geographiclib_datasets.sh && \
    chmod +x install_geographiclib_datasets.sh && \
    sudo ./install_geographiclib_datasets.sh
# other packages
RUN apt-get install -y \
    ros-noetic-velodyne \
    ros-noetic-velodyne-simulator \
    ros-noetic-socketcan-bridge \
    ros-noetic-can-msgs \
    ros-noetic-usb-cam \
    ros-noetic-joy

# install librealsense (ref. https://github.com/IntelRealSense/librealsense/blob/master/doc/distribution_linux.md)
RUN apt-get install -y software-properties-common
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE \
    || apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE
RUN add-apt-repository "deb https://librealsense.intel.com/Debian/apt-repo $(lsb_release -cs) main" -u
RUN apt-get update
RUN sudo apt-get install -y \
    librealsense2-dkms \
    librealsense2-utils \
    librealsense2-dev \
    librealsense2-dbg

# install PX4-Autopilot
# after install, you should set install path to ROS_PACKAGE_PATH and GAZEBO_MODEL_PATH
# export ROS_PACKAGE_PATH=${ROS_PACKAGE_PATH}:/usr/local/px4
# export GAZEBO_MODEL_PATH=${GAZEBO_MODEL_PATH}:/usr/local/px4/Tools/sitl_gazebo/models
RUN pip3 install kconfiglib jinja2 packaging jsonschema toml future pyros-genmsg
RUN git clone https://github.com/PX4/PX4-Autopilot.git --recursive /opt/PX4-Autopilot
RUN mkdir -p /opt/PX4-Autopilot/build
RUN cd /opt/PX4-Autopilot/build && cmake .. && make && make install

# install YDLidar-SDK for root
RUN cd /opt && git clone https://github.com/YDLIDAR/YDLidar-SDK.git
RUN mkdir -p /opt/YDLidar-SDK/build
RUN cd /opt/YDLidar-SDK/build && cmake .. && make && make install

# prepare ros environment
# local user settings
# add sudo user
RUN useradd -m -u ${UID} ${LOCAL_USER}
RUN mkdir -p /etc/sudoers.d && \
    touch /etc/sudoers.d/${LOCAL_USER}
RUN echo "%${LOCAL_USER} ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/${LOCAL_USER} && \
    chmod 0440 /etc/sudoers.d/${LOCAL_USER}
RUN chsh -s /bin/bash ${LOCAL_USER}
USER ${LOCAL_USER}
WORKDIR /home/${LOCAL_USER}

# create catkin workspace
RUN mkdir -p ~/catkin_ws/src && \
    cd ~/catkin_ws && \
    /bin/bash -c "source /opt/ros/noetic/setup.bash; catkin build"

# revise .bashrc
RUN echo "" >> ~/.bashrc && \
    echo "#additional statements" >> ~/.bashrc && \
    echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc && \
    echo "source /home/${LOCAL_USER}/catkin_ws/devel/setup.bash" >> ~/.bashrc && \
    echo "export ROS_PACKAGE_PATH=\${ROS_PACKAGE_PATH}:/home/${LOCAL_USER}/catkin_ws" >> ~/.bashrc && \
    echo "export ROS_PACKAGE_PATH=\${ROS_PACKAGE_PATH}:/usr/local/px4" >> ~/.bashrc && \
    echo "export ROS_WORKSPACE=/home/${LOCAL_USER}/catkin_ws" >> ~/.bashrc && \
    echo "export GAZEBO_MODEL_PATH=\${GAZEBO_MODEL_PATH}:/usr/local/px4/Tools/sitl_gazebo/models" >> ~/.bashrc

# build local ros packages
RUN sudo apt-get update
# realsense-ros
# install dependencies
RUN sudo apt-get install -y \
    ros-noetic-ddynamic-reconfigure
# clone repository
RUN cd ~/catkin_ws/src && git clone https://github.com/IntelRealSense/realsense-ros.git
# build
RUN catkin build -w ~/catkin_ws

# PX4-Avoidance
# install dependencies
RUN sudo apt-get install -y \
    libpcl1 \
    ros-noetic-pcl-ros
RUN sudo apt-get install -y \
    ros-noetic-octomap-*
#RUN sudo apt-get install -y \
#    ros-noetic-octomap-dbgsym \
#    ros-noetic-octomap-mapping \
#    ros-noetic-octomap-msgs \
#    ros-noetic-octomap-ros \
#    ros-noetic-octomap-ros-dbgsym
#    ros-noetic-octomap-rviz-plugins \
#    ros-noetic-octomap-rviz-plugins-dbgsym \
#    ros-noetic-octomap-server \
#    ros-noetic-octomap-server-dbgsym
# clone repository
RUN cd ~/catkin_ws/src && git clone https://github.com/PX4/avoidance.git
# build
RUN catkin build -w ~/catkin_ws

# ydlidar ros driver
# clone repository
RUN cd ~/catkin_ws/src && \
    git clone https://github.com/YDLIDAR/ydlidar_ros_driver.git
# build
RUN catkin build -w ~/catkin_ws

# hdl_graph_slam, hdl_localization
# install dependencies
RUN sudo apt install -y \
    ninja-build \
    libopencv-dev
RUN pip3 install opencv-python
RUN sudo apt-get install -y \
    ros-noetic-geodesy \
    ros-noetic-pcl-ros \
    ros-noetic-nmea-msgs \
    ros-noetic-libg2o \
    ros-noetic-tf-conversions
# clone repository
RUN cd ~/catkin_ws/src && \
    git clone https://github.com/koide3/ndt_omp.git && \
    git clone https://github.com/SMRT-AIST/fast_gicp.git --recursive && \
    git clone https://github.com/koide3/hdl_graph_slam && \
    git clone https://github.com/koide3/hdl_localization && \
    git clone https://github.com/koide3/hdl_global_localization
# build
RUN catkin build -w ~/catkin_ws

# package environment setup
RUN /bin/bash -c "source ~/.bashrc"

