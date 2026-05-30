FROM ubuntu:22.04

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

# ── ROS 2 Humble ─────────────────────────────────────────────────────────────
RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
        -o /usr/share/keyrings/ros-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
        http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" \
        > /etc/apt/sources.list.d/ros2.list

RUN apt-get update && apt-get install -y \
    ros-humble-desktop \
    python3-argcomplete \
    python3-colcon-common-extensions \
    python3-rosdep \
    python3-vcstool \
    tmux \
    tmuxinator \
    git \
    && rm -rf /var/lib/apt/lists/*

# ── Gazebo Fortress (ignition-fortress) ──────────────────────────────────────
RUN curl -sSL https://packages.osrfoundation.org/gazebo.gpg \
        -o /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] \
        http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" \
        > /etc/apt/sources.list.d/gazebo-stable.list

RUN apt-get update && apt-get install -y \
    ignition-fortress \
    && rm -rf /var/lib/apt/lists/*

# ── ROS 2 – Gazebo Fortress bridge ───────────────────────────────────────────
RUN apt-get update && apt-get install -y \
    ros-humble-ros-gz \
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

COPY --chown=${USERNAME}:${USERNAME} tmux/.tmux.conf \
     /home/${USERNAME}/.tmux.conf
    
COPY --chown=${USERNAME}:${USERNAME} tmux/.tmuxinator.yml \
     /home/${USERNAME}/.tmuxinator.yml


# ── Build ros2_cli_ws (custom ros2cli with topic subscribers list) ────────────
USER ${USERNAME}
RUN mkdir -p /home/${USERNAME}/ros2_cli_ws/src \
    && git clone -b humble-distro/feature-topic-subscribers-list \
        https://github.com/ROM-robotics/ros2cli.git \
        /home/${USERNAME}/ros2_cli_ws/src/ros2cli \
    && cd /home/${USERNAME}/ros2_cli_ws \
    && . /opt/ros/humble/setup.sh \
    && colcon build
USER root


# ── Environment setup (mr_robot) ──────────────────────────────────────────────
RUN echo "source /opt/ros/humble/setup.bash" >> /home/${USERNAME}/.bashrc \
    && echo "source /home/${USERNAME}/ros2_cli_ws/install/setup.bash" >> /home/${USERNAME}/.bashrc \
    && echo "export IGN_GAZEBO_RESOURCE_PATH=/usr/share/ignition" >> /home/${USERNAME}/.bashrc \
    && echo "source /opt/ros/humble/setup.bash" >> /root/.bashrc \
    && echo "source /home/${USERNAME}/ros2_cli_ws/install/setup.bash" >> /root/.bashrc \
    && echo "export IGN_GAZEBO_RESOURCE_PATH=/usr/share/ignition" >> /root/.bashrc

# ── Workspace ─────────────────────────────────────────────────────────────────
RUN mkdir -p /home/${USERNAME}/ros2_ws/src \
    && chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}/ros2_ws

USER ${USERNAME}
WORKDIR /home/${USERNAME}/ros2_ws

CMD ["/bin/bash"]
