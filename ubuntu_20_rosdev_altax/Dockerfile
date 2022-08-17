FROM ubuntu:20.04

ENV DEBIAN_FRONTEND noninteractive

# add sudo user
ARG UID=1000
RUN useradd -m -u ${UID} user
RUN mkdir -p /etc/sudoers.d \
    && touch /etc/sudoers.d/user
RUN echo "%user	ALL=(ALL)	NOPASSWD:	ALL" >> /etc/sudoers.d/user \
    && chmod 0440 /etc/sudoers.d/user

# install utilities
RUN sed -i.org -e 's|ports.ubuntu.com|jp.archive.ubuntu.com|g' /etc/apt/sources.list \
    && apt-get update && apt-get upgrade -y && apt-get install -y \
       sudo \
       iputils-ping \
       iproute2 \
       x11-apps \
       curl \
       lsb-core \
       git \
       vim \
       can-utils \
       kmod \
       v4l-utils \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# install ros
RUN sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
RUN curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add -
RUN apt-get update && apt-get install -y ros-noetic-desktop-full
RUN apt-get update && apt-get install -y python3-pip python3-rosinstall python3-catkin-tools python3-rosdep
RUN rosdep init
RUN rosdep update

# install ros packages
# for running gazebo
RUN apt-get update && apt-get install -y \
    gazebo11 \
    libgazebo11-dev \
    ros-noetic-gazebo-ros-pkgs \
    ros-noetic-gazebo-ros-control
# for building mavros, mavros-extras
RUN apt-get update && apt-get install -y \
    ros-noetic-geographic-msgs \
    ros-noetic-geodesy \
    ros-noetic-nmea-msgs \
    libgeographic-dev \
    geographiclib-tools
# for building hdl_graph_slam
RUN apt-get update && apt-get install -y \
    ros-noetic-libg2o
# for building global_planner
RUN apt-get update && apt-get install -y \
    ros-noetic-octomap-*
# for running altax_autonomy, simulation
RUN apt-get update && apt-get install -y \
    ros-noetic-velodyne \
    ros-noetic-velodyne-simulator
# other packages
RUN apt-get update && apt-get install -y \
    ros-noetic-can-msgs \
    ros-noetic-usb-cam

# install PX4-Autopilot
# after install, you should set install path to ROS_PACKAGE_PATH and GAZEBO_MODEL_PATH
# export ROS_PACKAGE_PATH=${ROS_PACKAGE_PATH}:/usr/local/px4
# export GAZEBO_MODEL_PATH=${GAZEBO_MODEL_PATH}:/usr/local/px4/Tools/sitl_gazebo/models
RUN pip3 install kconfiglib jinja2 packaging jsonschema toml future pyros-genmsg
RUN git clone https://github.com/PX4/PX4-Autopilot.git --recursive /opt/PX4-Autopilot
RUN mkdir -p /opt/PX4-Autopilot/build && \
    cd /opt/PX4-Autopilot/build && \
    cmake .. && \
    make && \
    make install

# prepare ros environment
# default user settings
RUN chsh -s /bin/bash user
USER user
WORKDIR /home/user

# create catkin workspace
RUN mkdir -p /home/user/catkin_ws/src && \
    cd /home/user/catkin_ws && \
    /bin/bash -c "source /opt/ros/noetic/setup.bash; catkin build"

# revise .bashrc
RUN echo "" >> /home/user/.bashrc && \
    echo "#additional statements" >> /home/user/.bashrc && \
    echo "source /opt/ros/noetic/setup.bash" >> /home/user/.bashrc && \
    echo "source /home/user/catkin_ws/devel/setup.bash" >> /home/user/.bashrc && \
    echo "export ROS_PACKAGE_PATH=\${ROS_PACKAGE_PATH}:/home/user/catkin_ws" >> /home/user/.bashrc && \
    echo "export ROS_PACKAGE_PATH=\${ROS_PACKAGE_PATH}:/usr/local/px4" >> /home/user/.bashrc && \
    echo "export ROS_WORKSPACE=/home/user/catkin_ws" >> /home/user/.bashrc && \
    echo "export GAZEBO_MODEL_PATH=\${GAZEBO_MODEL_PATH}:/usr/local/px4/Tools/sitl_gazebo/models" >> /home/user/.bashrc && \
    echo "export GAZEBO_MODEL_PATH=\${GAZEBO_MODEL_PATH}:/home/user/catkin_ws/src/altax_autonomy/models/models" >> /home/user/.bashrc

# locate shared files
RUN rm -rf /home/user/catkin_ws/src && ln -s /home/user/Share/altax_src /home/user/catkin_ws/src
RUN ln -s /home/user/Share/set_proxy.sh /home/user/set_proxy.sh

# print message
RUN echo "\n##################################################" && \
    echo "After running a container made from this image," && \
    echo "1. rebuild altax_autonomy packages, as below," && \
    echo "    \$ cd /home/user/catkin_ws && catkin clean -b -y && catkin build" && \
    echo "2. set proxy (if needed), as below," && \
    echo "    \$ ./setup_proxy" && \
    echo "3. install geographiclib datasets, as below," && \
    echo "    \$ sudo bash /home/user/catkin_ws/src/mavros/mavros/scripts/install_geooraphiclib_datasets.sh" && \
    echo "(if execute this command over proxy, use sudo with option '-E')" && \
    echo "##################################################\n"

