# Gazebo Harmonic + ROS 2 Humble Desktop — Docker

Ubuntu 22.04 base image တွင် **Gazebo Harmonic** နှင့် **ROS 2 Humble Desktop** တို့ကို တပ်ဆင်ထားသော Docker environment။

## 1. Stack

| Component | Version |
|---|---|
| OS | Ubuntu 22.04 (Jammy) |
| ROS 2 | Humble Desktop |
| Gazebo | Harmonic (gz-harmonic) |
| ROS-GZ Bridge | ros-humble-ros-gzharmonic |
| GZ ros2 control | ros-humble-gz-ros2-control |
| Default User | `mr_robot` (UID 1000, passwordless sudo) |
| Workspace | `/home/mr_robot/ros2_ws` |

---

## 2. Prerequisites

- [Docker](https://docs.docker.com/engine/install/ubuntu/) install လုပ်ပြီးဖြစ်ရမည်
- [Docker Compose](https://docs.docker.com/compose/install/) v2.x+
- GUI (Gazebo) ကြည့်ရှုလိုပါက X11 ရှိရမည်

---

## 3. How to use ?

docker hub တွင် ရှိပါက docker compose ဖြင့် build လုပ်ရန်မလို။
```bash
docker pull romrobotics/gz_harmonic_ubun22_humble:base
# or
docker pull romrobotics/gz_harmonic_ubun22_humble:reeman_clone
# or
docker pull romrobotics/gz_harmonic_ubun22_humble:px4_autopilot
```

---

## 4. How to build docker image

```bash
cd gz_harmonic_ubun22_humble

docker compose build
```

---

## 5. How to run docker image

### 5.1 for pc without nvidia

```bash
#!/usr/bin/bash
docker stop simulator_01
docker rm simulator_01
xhost +local:root
docker run -it --network='host' \
-p 80:80 \
-p 9090:9090 \
--env='DISPLAY' \
--env='QT_X11_NO_MITSHM=1' \
--env='XDG_RUNTIME_DIR=/run/user/${UID}' \
--env='GZ_VERSION=harmonic' \
--volume='/tmp/.X11-unix:/tmp/.X11-unix:rw' \
--name simulator_01 \
gz_harmonic_ubun22_humble:latest bash
docker stop simulator_01
docker rm simulator_01
```

Container ထဲရောက်သောအခါ:

```bash
# Gazebo Harmonic စမ်းသပ်ပါ
gz sim

# ROS 2 စမ်းသပ်ပါ
ros2 topic list
```

### 5.2 for pc with nvidia

`docker-compose.yml` တွင် အောက်ပါ section များ ထပ်ထည့်ပါ:

```yaml
services:
  gz_harmonic_humble:
    # ... existing config ...
    environment:
      - DISPLAY=${DISPLAY}
      - QT_X11_NO_MITSHM=1
      - GZ_VERSION=harmonic
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
docker exec -it gz_harmonic_humble bash
```
