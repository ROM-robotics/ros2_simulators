# Gazebo Fortress + ROS 2 Humble Desktop — Docker

Ubuntu 22.04 base image တွင် **Ignition Fortress** နှင့် **ROS 2 Humble Desktop** တို့ကို တပ်ဆင်ထားသော Docker environment။

## Stack

| Component | Version |
|---|---|
| OS | Ubuntu 22.04 (Jammy) |
| ROS 2 | Humble Desktop |
| Gazebo | Ignition Fortress |
| ROS-GZ Bridge | ros-humble-ros-gz |
| Default User | `mr_robot` (UID 1000, passwordless sudo) |
| Workspace | `/home/mr_robot/ros2_ws` |

---

## Prerequisites

- [Docker](https://docs.docker.com/engine/install/ubuntu/) ติดตั้งပြီးဖြစ်ရမည်
- [Docker Compose](https://docs.docker.com/compose/install/) v2.x+
- GUI (Gazebo) ကြည့်ရှုလိုပါက X11 ရှိရမည်

---

## How to build docker image

```bash
cd gz_fortress_ubun22_humble

docker compose build
```

custom UID/GID သုံးလိုပါက:

```bash
docker compose build \
  --build-arg USER_UID=$(id -u) \
  --build-arg USER_GID=$(id -g)
```

---

## How to run docker image

### for pc without nvidia

```bash
# Host တွင် X11 access ခွင့်ပြုပါ
xhost +local:docker

docker compose up -d
docker exec -it gz_fortress_humble bash
```

Container ထဲရောက်သောအခါ:

```bash
# Gazebo Fortress စမ်းသပ်ပါ
ign gazebo

# ROS 2 စမ်းသပ်ပါ
ros2 topic list
```

### for pc with nvidia

`docker-compose.yml` တွင် အောက်ပါ section များ ထပ်ထည့်ပါ:

```yaml
services:
  gz_fortress_humble:
    # ... existing config ...
    environment:
      - DISPLAY=${DISPLAY}
      - QT_X11_NO_MITSHM=1
      - NVIDIA_VISIBLE_DEVICES=all
      - NVIDIA_DRIVER_CAPABILITIES=graphics,utility,compute
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
```

ထို့နောက် run ပါ:

```bash
xhost +local:docker
docker compose up -d
docker exec -it gz_fortress_humble bash
```

---

## Workspace

Host ၏ `./ros2_ws` folder သည် container ထဲ `/home/mr_robot/ros2_ws` သို့ mount ဖြစ်သည်။

```bash
# ROS 2 package တည်ဆောက်ပါ
cd ~/ros2_ws
colcon build --symlink-install
source install/setup.bash
```

---

## Stop / Remove

```bash
# Container ရပ်ပါ
docker compose down

# Image ဖျက်ပါ
docker rmi gz_fortress_ubun22_humble:latest
```
