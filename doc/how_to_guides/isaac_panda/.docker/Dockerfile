FROM osrf/ros:humble-desktop-jammy

SHELL ["/bin/bash", "-c", "-o", "pipefail"]

ARG CUDA_MAJOR_VERSION=11
ARG CUDA_MINOR_VERSION=7

RUN echo "deb [trusted=yes] https://raw.githubusercontent.com/moveit/moveit2_packages/jammy-humble/ ./" \
    | sudo tee /etc/apt/sources.list.d/moveit_moveit2_packages.list
RUN echo "yaml https://raw.githubusercontent.com/moveit/moveit2_packages/jammy-humble/local.yaml humble" \
    | sudo tee /etc/ros/rosdep/sources.list.d/1-moveit_moveit2_packages.list

# Bring the container up to date to get the latest ROS2 humble sync on 01/27/23
# hadolint ignore=DL3008, DL3013
RUN apt-get update && apt-get upgrade -y && rosdep update

# Install packages required to run the demo
RUN apt-get install -y --no-install-recommends \
    ros-humble-moveit
    
# Create Colcon workspace and clone the needed source code to run the demo
RUN mkdir -p /root/isaac_moveit_tutorial_ws/src
WORKDIR /root/isaac_moveit_tutorial_ws/src
RUN git clone https://github.com/DarK404/UR_Isaac-sim .\
    && git clone https://github.com/Dark404/robotiq_85_gripper.git \
    && cd .. \
    && source /opt/ros/humble/setup.bash && colcon build --symlink-install

RUN git clone https://github.com/PickNikRobotics/topic_based_ros2_control.git

WORKDIR /root/isaac_moveit_tutorial_ws/src
RUN git clone https://github.com/ros-planning/moveit_resources.git -b humble

WORKDIR /root/isaac_moveit_tutorial_ws/src
COPY ./ moveit2_tutorials

WORKDIR /root/isaac_moveit_tutorial_ws
# hadolint ignore=SC1091
RUN source /opt/ros/humble/setup.bash \
    && apt-get update -y \
    && rosdep install --from-paths src --ignore-src --rosdistro "$ROS_DISTRO" -y \
    && rm -rf /var/lib/apt/lists/*

# Use Fast DDS as middleware and load the required config for NVIDIA Isaac Sim.
ENV RMW_IMPLEMENTATION=rmw_fastrtps_cpp
RUN mkdir -p /opt/.ros
COPY ./doc/how_to_guides/isaac_panda/.docker/fastdds.xml /opt/.ros/fastdds.xml
ENV FASTRTPS_DEFAULT_PROFILES_FILE=/opt/.ros/fastdds.xml

# Build the Colcon workspace for the user
RUN source /opt/ros/humble/setup.bash && colcon build 

# Set up the entrypoint for both container start and interactive terminals.
COPY ./doc/how_to_guides/isaac_panda/.docker/ros_entrypoint.sh /opt/.ros/
RUN echo "source /opt/.ros/ros_entrypoint.sh" >> ~/.bashrc
ENTRYPOINT [ "/opt/.ros/ros_entrypoint.sh" ]
