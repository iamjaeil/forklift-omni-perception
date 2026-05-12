# Forklift Omni Perception: 360° 3D Object Detection and Tracking for Industrial Safety

<p align="center">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.8-blue">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-1.12.1-red">
  <img alt="Torchvision" src="https://img.shields.io/badge/Torchvision-0.13.1-orange">
  <img alt="CUDA" src="https://img.shields.io/badge/CUDA-11.6-green">
</p>

<p align="center">
  <img src="src/assets/camera_night.gif">
  <img src="src/assets/Lidar_nigh.gif">
</p>

## 📑 Overview

- [Introduction](#-introduction)
- [Key Features](#-key-features)
- [Used Sensors](#-used-sensors)
- [Pipeline](#-pipeline)
  - [1. 2D Detection (DINO)](#1-2d-detection-dino)
  - [2. Tracking (OC-SORT)](#2-tracking-oc-sort)
  - [3. 3D Detection (Ultralytics ROS)](#3-3d-detection-ultralytics-ros)
- [Results](#-results)
- [Getting Started](#-getting-started)
  - [1. Installation](#1-installation)
  - [2. How to Run](#2-how-to-run)
  - [3. Docker](#3-docker)
- [References](#-references)

## 🚀 Introduction

<img width="677" height="913" alt="image" src="https://github.com/user-attachments/assets/2cdc362f-414c-4fc1-a6e3-54a5724ca0b0" />

**Forklift Omni Perception** is a 360-degree 3D perception system designed for autonomous forklifts and industrial safety applications.

The system combines a 360° camera and a 360° LiDAR sensor to detect, track, and localize objects in all directions. It performs robust object detection and tracking using ERP-based 2D projection, DINO, and OC-SORT. The system can be mounted on forklifts or mobile robots and has been evaluated in indoor, outdoor, and real-world industrial factory environments.

## 🔑 Key Features

- **360° Full Surround Perception**  
  ◦ Detects objects in all directions using a single 360° camera and 360° LiDAR  
  ◦ Provides a platform-independent perception setup without requiring complex additional sensor configurations  

- **Efficient ERP-Based Point Matching**  
  ◦ Projects 3D LiDAR points onto an equirectangular projection image  
  ◦ Reduces computational cost by replacing complex 3D-to-3D matching with 2D bounding-box-based point association  

- **Robust Object Detection and Tracking**  
  ◦ Uses DINO for robust object detection on distorted ERP images  
  ◦ Uses OC-SORT for fast and stable multi-object tracking across frames  

- **Real-World Industrial Evaluation**  
  ◦ Tested in a real CJ industrial factory, indoor campus environments, and outdoor night scenes  
  ◦ Quantitatively evaluated using AP, MOTA, and IDF1  
  ◦ Demonstrates practical applicability through risk-level visualization  

## 🤖 Used Sensors

<img width="477" height="634" alt="image" src="https://github.com/user-attachments/assets/d494772a-46fe-4727-bab7-867aad4ee700" />

<br>

**1. Ricoh Theta Z1 360° Camera**  
**2. Livox Mid-360 LiDAR**  
**3. Custom Integrated Sensor Mount**

## 🛠 Pipeline

<img width="1694" height="567" alt="image" src="https://github.com/user-attachments/assets/305b65ba-ad3f-4851-ba96-387f88517d26" />

### 1. 2D Detection (DINO)

- Takes ERP-projected 2D images as input
- Detects human bounding boxes from omnidirectional camera images
- Extracts robust global object features using self-attention, even under ERP distortion

### 2. Tracking (OC-SORT)

- Takes bounding boxes detected by DINO as input
- Maintains consistent object IDs across frames
- Provides robust tracking performance in factory environments with frequent occlusion

### 3. 3D Detection (Ultralytics ROS)

- Transforms LiDAR points into the camera frame using extrinsic calibration
- Projects 3D LiDAR points onto the ERP image plane
- Matches projected points by checking whether each point `(u, v)` lies inside a detected bounding box
- Restores matched points to 3D coordinates
- Estimates the distance from the forklift or robot to each detected object

## 📊 Results

<img width="1126" height="607" alt="image" src="https://github.com/user-attachments/assets/5f469dd9-f303-4ff2-adc6-c4d90d9024e8" />

<br>

### Detection Performance (AP / CD)

| Environment | AP | CD(px) |
|:---:|:---:|:---:|
| Factory | 0.588 | 24 |
| Indoor | **0.700** | 27 |
| Outdoor Night | 0.095 | 30 |

### Tracking Performance (MOTA / IDF1)

| Environment | MOTA | IDF1 |
|:---:|:---:|:---:|
| Factory | **89.1** | 83.5 |
| Indoor | 76.7 | **84.9** |
| Outdoor Night | 53.4 | 67.2 |

## 🔧 Getting Started

### 1. Installation

#### RTX 30 Series

Tested environment:

- Python 3.8
- PyTorch 1.12.1
- Torchvision 0.13.1
- CUDA 11.6

#### i. Environment Setup

```bash
pip install torch==1.12.1+cu116 torchvision==0.13.1+cu116 \
  --extra-index-url https://download.pytorch.org/whl/cu116
```

#### ii. Build Package

```bash
git clone https://github.com/happious/3d_detection
cd 3d_detection
pip install -r requirements.txt
```

```bash
cd src/ultralytics_ros/DINO
mkdir weights
mv ~/Downloads/checkpoint0029_4scale_swin.pth ~/3d_detection/src/ultralytics_ros/DINO/weights/
```

```bash
cd ..
mkdir bag
mv ~/your.bag ~/3d_detection/src/ultralytics_ros/bag
```

#### iii. Catkin Build

```bash
cd ~/3d_detection
catkin_make
source devel/setup.bash
```

---

### 2. How to Run

```bash
roslaunch ultralytics_ros tracking.launch
```

```bash
roslaunch ultralytics_ros tracker_with_cloud_ros1.launch
```

---

### 3. Docker

#### 3D Detection + DINO + OC-SORT

Environment:

- Ubuntu 20.04
- ROS Noetic
- Docker
- PyTorch 1.12.1
- CUDA 11.6

#### 3.1 Create Workspace on Host

```bash
mkdir -p ~/your_ws
cd ~/your_ws
```

#### 3.2 Clone 3D Detection Source on Host

```bash
cd ~/your_ws
git clone https://github.com/happious/3d_detection.git
```

#### 3.3 Prepare DINO Weights on Host

```bash
mkdir -p ~/your_ws/3d_detection/src/ultralytics_ros/DINO/weights
cp ~/Downloads/checkpoint0011_4scale.pth \
   ~/your_ws/3d_detection/src/ultralytics_ros/DINO/weights/
```

#### 3.4 Prepare Bag File on Host

```bash
mkdir -p ~/your_ws/3d_detection/src/ultralytics_ros/bag
cp ~/CJ.bag \
   ~/your_ws/3d_detection/src/ultralytics_ros/bag/
```

#### 3.5 Create Dockerfile on Host

Create the Dockerfile at:

```bash
~/your_ws/Dockerfile
```

Move the Dockerfile:

```bash
mv ~/your_ws/3d_detection/docker/Dockerfile ~/your_ws/Dockerfile
```

#### 3.6 Allow X11 on Host

```bash
xhost +local:docker
```

#### 3.7 Build Docker Image on Host

```bash
cd ~/your_ws
docker build -t 3d_detection_dino .
```

#### 3.8 Run Container with GUI and Volume Mount

```bash
docker run --gpus all -it \
  --name dino_container \
  -v ~/your_ws/3d_detection:/opt/catkin_ws/src/3d_detection \
  -e DISPLAY=$DISPLAY \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  --env="QT_X11_NO_MITSHM=1" \
  3d_detection_dino
```

#### 3.9 Build DINO CUDA Ops

```bash
cd /opt/catkin_ws/src/3d_detection/src/ultralytics_ros/DINO/models/dino/ops

# Clean build
pip3 uninstall -y MultiScaleDeformableAttention || true
rm -rf build/ dist/ MultiScaleDeformableAttention.egg-info
find . -name "MultiScaleDeformableAttention*.so" -delete || true

# CUDA 11.6 build
export CUDA_HOME=/usr/local/cuda

FORCE_CUDA=1 python3 setup.py build_ext --inplace
FORCE_CUDA=1 python3 -m pip install .

# Import test
python3 - << 'EOF'
import torch
import MultiScaleDeformableAttention as MSDA
print("torch:", torch.__version__, "cuda:", torch.version.cuda)
print("cuda.is_available:", torch.cuda.is_available())
print("MSDA:", MSDA)
EOF
```

#### 3.10 Launch

**Terminal 1**

```bash
cd /opt/catkin_ws
roslaunch ultralytics_ros tracking.launch
```

**Terminal 2**

```bash
docker exec -it dino_container bash
roslaunch ultralytics_ros tracker_with_cloud_ros1.launch
```

**Terminal 3**

```bash
docker exec -it dino_container bash
rviz
```

## 📕 References

[1] DINO: https://github.com/IDEA-Research/DINO  
[2] OC-SORT: https://github.com/noahcao/OC_SORT  
[3] Ultralytics ROS: https://github.com/Alpaca-zip/ultralytics_ros
