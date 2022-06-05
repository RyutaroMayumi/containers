FROM ubuntu:18.04

ENV DEBIAN_FRONTEND noninteractive

ARG UID=1000

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

RUN useradd -m -u ${UID} user
RUN mkdir -p /etc/sudoers.d \
    && touch /etc/sudoers.d/user
RUN echo "%user	ALL=(ALL)	NOPASSWD:	ALL" >> /etc/sudoers.d/user \
    && chmod 0440 /etc/sudoers.d/user

RUN sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
RUN curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add -
RUN apt-get update && apt-get install -y ros-melodic-desktop-full
RUN apt-get update && apt-get install -y \
    ros-melodic-can-msgs \
    ros-melodic-uvc-camera

RUN apt-get update && apt-get install -y python-rosinstall python-catkin-tools python-rosdep
RUN rosdep init
RUN rosdep update

RUN sudo -u user mkdir -p /home/user/catkin_ws/src && \
    cd /home/user/catkin_ws/src && \
    sudo -u user /bin/bash -c "source /opt/ros/melodic/setup.bash; catkin_init_workspace" && \
    cd /home/user/catkin_ws && \
    sudo -u user /bin/bash -c "source /opt/ros/melodic/setup.bash; catkin_make" && \
    echo "" >> /home/user/.bashrc && \
    echo "#additional statements" >> /home/user/.bashrc && \
    echo "source /opt/ros/melodic/setup.bash" >> /home/user/.bashrc && \
    echo "source /home/user/catkin_ws/devel/setup.bash" >> /home/user/.bashrc && \
    echo "export ROS_PACKAGE_PATH=\${ROS_PACKAGE_PATH}:/home/user/catkin_ws" >> /home/user/.bashrc && \
    echo "export ROS_WORKSPACE=/home/user/catkin_ws" >> /home/user/.bashrc

RUN rm -rf /home/user/catkin_ws/src && sudo -u user ln -s /home/user/Share/ros_src /home/user/catkin_ws/src

RUN chsh -s /bin/bash user
USER user
WORKDIR /home/user
