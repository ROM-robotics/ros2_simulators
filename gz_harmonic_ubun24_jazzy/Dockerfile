FROM ubuntu:24.04

# Prevent interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive
ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

# ── Basic dependencies ────────────────────────────────────────────────────────
RUN apt-get update && apt-get install -y \
    locales \
    curl \
    gnupg2 \
    lsb-release \
    software-properties-common \
    wget \
    ca-certificates \
    && locale-gen en_US en_US.UTF-8 \
    && update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 \
    && rm -rf /var/lib/apt/lists/*

# ── ROS 2 Jazzy ──────────────────────────────────────────────────────────────
RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
        -o /usr/share/keyrings/ros-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
        http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" \
        > /etc/apt/sources.list.d/ros2.list

RUN apt-get update && apt-get install -y \
    ros-jazzy-desktop \
    python3-argcomplete \
    python3-colcon-common-extensions \
    python3-rosdep \
    python3-vcstool \
    && rm -rf /var/lib/apt/lists/*

# ── Gazebo Harmonic ───────────────────────────────────────────────────────────
# Jazzy ၏ default paired Gazebo သည် Harmonic ဖြစ်သည်
RUN curl -sSL https://packages.osrfoundation.org/gazebo.gpg \
        -o /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] \
        http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" \
        > /etc/apt/sources.list.d/gazebo-stable.list

RUN apt-get update && apt-get install -y \
    gz-harmonic \
    && rm -rf /var/lib/apt/lists/*

# ── ROS 2 – Gazebo Harmonic bridge & drivers ─────────────────────────────────
# ros-jazzy-ros-gz           : ROS 2 Jazzy <-> Gazebo Harmonic full bridge
# ros-jazzy-gz-ros2-control  : ros2_control hardware interface for Gazebo Harmonic
RUN apt-get update && apt-get install -y \
    ros-jazzy-ros-gz \
    ros-jazzy-gz-ros2-control \
    && rm -rf /var/lib/apt/lists/*

# ── rosdep initialisation ────────────────────────────────────────────────────
RUN rosdep init && rosdep update

# ── User: mr_robot ────────────────────────────────────────────────────────────
ARG USERNAME=mr_robot
ARG USER_UID=1000
ARG USER_GID=1000

RUN groupadd --gid ${USER_GID} ${USERNAME} \
    && useradd --uid ${USER_UID} --gid ${USER_GID} --create-home --shell /bin/bash ${USERNAME} \
    && apt-get update && apt-get install -y sudo \
    && echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/${USERNAME} \
    && chmod 0440 /etc/sudoers.d/${USERNAME} \
    && rm -rf /var/lib/apt/lists/*

# ── Environment setup (mr_robot) ──────────────────────────────────────────────
RUN echo "source /opt/ros/jazzy/setup.bash" >> /home/${USERNAME}/.bashrc \
    && echo "export GZ_VERSION=harmonic" >> /home/${USERNAME}/.bashrc \
    && echo "export GZ_SIM_RESOURCE_PATH=/usr/share/gz" >> /home/${USERNAME}/.bashrc \
    && echo "source /opt/ros/jazzy/setup.bash" >> /root/.bashrc \
    && echo "export GZ_VERSION=harmonic" >> /root/.bashrc \
    && echo "export GZ_SIM_RESOURCE_PATH=/usr/share/gz" >> /root/.bashrc

# ── Workspace ─────────────────────────────────────────────────────────────────
RUN mkdir -p /home/${USERNAME}/ros2_ws/src \
    && chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}/ros2_ws

USER ${USERNAME}
WORKDIR /home/${USERNAME}/ros2_ws

CMD ["/bin/bash"]
