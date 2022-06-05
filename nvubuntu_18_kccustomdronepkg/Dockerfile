# nvidia openglが利用できるimageでスタート
FROM nvidia/opengl:1.0-glvnd-runtime-ubuntu18.04

ENV DEBIAN_FRONTEND=noninteractive
RUN mkdir temp
WORKDIR /root/temp

# MAVPROXY
RUN apt-get update && apt-get install -y python3-dev  python3-pip python-pip
RUN pip install --upgrade pip
RUN pip install MAVProxy
RUN echo 'export PATH=$PATH:$HOME/.local/bin' >> ~/.bashrc

# added at 20211002 ryutaro.mayumi to deal with gpg key expired
RUN apt-get update && apt-get install -y curl
RUN curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add -

# ROS setup
RUN sh -c 'echo "deb http://packages.ros.org/ros/ubuntu bionic main" > /etc/apt/sources.list.d/ros-latest.list'

#RUN apt-key adv --keyserver 'hkp://ha.pool.sks-keyservers.net:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
RUN cat /etc/lsb-release
# comment out at 20211002
# RUN apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
RUN apt-get update && apt-get install -y ros-melodic-desktop-full

# MAVROS, other ros packages
RUN apt-get update && apt-get install -y ros-melodic-mavros \
        ros-melodic-mavros-extras \
        ros-melodic-socketcan-bridge \
        ros-melodic-timed-roslaunch \
        ros-melodic-joy
RUN apt-get update && sudo apt-get install -y ros-melodic-grid-map \
        ros-melodic-jsk-rviz-plugins \
        ros-melodic-velodyne-pointcloud \
        ros-melodic-move-base \
        ros-melodic-robot-localization

RUN /opt/ros/melodic/lib/mavros/install_geographiclib_datasets.sh

# gazebo9
RUN sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list'
RUN wget http://packages.osrfoundation.org/gazebo.key -O - | sudo apt-key add -
RUN apt-get update && apt-get install -y gazebo9 \
        libgazebo9-dev \
        ros-melodic-gazebo9-* \
        libignition-math2

RUN apt-get update && apt-get upgrade -y libignition-math2

# cartographer
RUN apt-get update && apt-get install -y ros-melodic-cartographer*

# GUI周りの設定
# Replace 1000 with your user / group id
ARG user=user
ARG uid=1000
ARG gid=1000
RUN groupadd -g ${uid} ${user} && \
        useradd -u ${gid} -g ${user} -r ${user} && \
        mkdir /home/${user} && \
        chown ${uid}:${gid} -R /home/${user}

RUN apt-get update && apt-get install -y git 
# ${user}のsudoのパスワード無効化
RUN echo "${user} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/${user}
RUN chmod 0440 /etc/sudoers.d/${user}

## ここまで rootで実行、以下は通常のユーザ(指定がなければ$(user)="user")
USER ${user}
WORKDIR /home/${user}
RUN mkdir work

## ardupilot

WORKDIR /home/${user}/work
RUN git clone https://github.com/ArduPilot/ardupilot.git

WORKDIR /home/${user}/work/ardupilot
RUN sed --in-place 's#git://#https://#g' .gitmodules
RUN git submodule update --init --recursive
ENV USER=${user}
RUN Tools/environment_install/install-prereqs-ubuntu.sh -y


## ardupilot_gazebo
WORKDIR /home/${user}/work
RUN git clone https://github.com/khancyr/ardupilot_gazebo.git
WORKDIR /home/${user}/work/ardupilot_gazebo
RUN mkdir build
WORKDIR /home/${user}/work/ardupilot_gazebo/build
RUN cmake ..
RUN make
RUN sudo make install

## setup catkin_ws
WORKDIR /home/${user}
RUN mkdir -p catkin_ws/src
WORKDIR /home/${user}/catkin_ws/src
RUN . /opt/ros/melodic/setup.sh && catkin_init_workspace
WORKDIR /home/${user}/catkin_ws
RUN . /opt/ros/melodic/setup.sh && catkin_make

## copy&build kc_custom_drone_pkgs
COPY --chown=${user}:${user} . /home/${user}/catkin_ws/src/kc_custom_drone_pkgs
WORKDIR /home/${user}/catkin_ws
RUN . /home/${user}/catkin_ws/devel/setup.sh && catkin_make
RUN echo 'source ~/catkin_ws/devel/setup.bash' >> ~/.bashrc

## additional packages
RUN sudo apt-get update && sudo apt-get install -y ros-melodic-map-server
RUN sudo apt-get update && sudo apt-get install -y vim
RUN sudo addgroup user input


