# ♿ Autonomous Wheelchair Fleet — Leader-Follower System

> Autonomous fleet of motorized wheelchairs where follower units track a lead robot using computer vision and real-time control — designed for people with reduced mobility (PRM).

![ROS2](https://img.shields.io/badge/ROS2-Humble-blue?logo=ros) ![C++](https://img.shields.io/badge/C++-17-blue?logo=cplusplus) ![OpenCV](https://img.shields.io/badge/OpenCV-4.x-green?logo=opencv) ![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

**JUNIA-ISEN Lille** · Engineering project · January – May 2025

---

## 📋 Overview

This project implements an autonomous **leader-follower convoy system** for a fleet of motorized wheelchairs. A lead robot navigates freely while follower wheelchairs track it autonomously using a combination of computer vision (ArUco marker detection) and a PD controller — maintaining safe, accurate inter-vehicle distance without any human intervention on the follower units.

The system is designed to assist people with reduced mobility (PRM), enabling them to follow a guide (person or robot) hands-free.

Key results:
- **±10 cm** tracking precision in real test conditions
- **> 92%** tracking stability over full test distance
- **−35%** trajectory error vs baseline model with optimized PD controller

---

## 🎬 Demo

### 1 — Real-world test · Leader-follower tracking

https://github.com/vinny-juniors/wheelchair-fleet/raw/main/docs/test_chaise1.mp4

> Follower wheelchair autonomously tracks the lead robot in real corridor conditions. Tracking precision: ±10 cm, stability > 92%.

---

### 2 — MATLAB + CoppeliaSim simulation · Fleet navigation

https://github.com/vinny-juniors/wheelchair-fleet/raw/main/docs/simulation_matlab_coppelia.mp4

> Full fleet navigation simulation built in MATLAB, synchronized in real time with CoppeliaSim. Validates the leader-follower control law before deployment on hardware.

---

### 3 — Custom velocity operator · QR code angle estimation

https://github.com/vinny-juniors/wheelchair-fleet/raw/main/docs/operator_velocity_qrcode.mp4

> Custom ROS2 operator designed to capture the leader's velocity data and compute the heading angle from QR code detection — feeding the PD controller with accurate angular correction in real time.

---

## 🏗️ System Architecture

```
┌─────────────────────────────────┐
│         LEAD ROBOT              │
│   (manual or autonomous nav)    │
│                                 │
│   [ArUco marker — rear-mounted] │
└────────────────┬────────────────┘
                 │  Visual detection
                 ▼
┌─────────────────────────────────┐
│       FOLLOWER WHEELCHAIR       │
│                                 │
│  [Camera] → [ArUco Detection]   │
│       (OpenCV)                  │
│                  │              │
│         [Pose Estimation]       │
│         (distance + angle)      │
│                  │              │
│          [PD Controller]        │
│          (C++ ROS2 node)        │
│                  │              │
│         [Motor Commands]        │
│         /cmd_vel topic          │
└─────────────────────────────────┘
```

**The same follower stack can be replicated across N wheelchairs** — each unit tracks the one in front of it, forming a convoy chain.

---

## ⚙️ How It Works

### 1. Leader detection
The follower's camera continuously detects an **ArUco marker** mounted on the rear of the leader. OpenCV computes the marker's pose (position + orientation relative to the camera).

### 2. Control law (PD controller)
```cpp
// Distance error
float dist_error = target_distance - measured_distance;

// Angle error
float angle_error = measured_angle;  // 0 when perfectly aligned

// PD control outputs
float linear_vel  = Kp_dist  * dist_error  + Kd_dist  * d_dist_error;
float angular_vel = Kp_angle * angle_error + Kd_angle * d_angle_error;
```

**Tuned gains:**
```yaml
Kp_dist:  1.2
Kd_dist:  0.30
Kp_angle: 1.5
Kd_angle: 0.35
target_distance: 0.8   # meters
max_linear_vel:  0.3   # m/s
max_angular_vel: 0.8   # rad/s
```

### 3. Safety
- If the marker is **lost** for > 1 second → emergency stop
- Speed is capped to prevent collisions

---

## 📁 Repository Structure

```
wheelchair-fleet/
├── src/
│   ├── wheelchair_perception/    # ArUco detection + pose estimation (C++)
│   ├── wheelchair_control/       # PD controller node (C++)
│   ├── wheelchair_bringup/       # Launch files
│   └── wheelchair_interfaces/    # Custom ROS2 messages
├── config/
│   ├── controller_params.yaml    # PD gains: Kp, Kd, target distance
│   └── camera_calibration.yaml  # Camera intrinsic parameters
├── docs/
│   └── test_chaise1.mp4         # Real-world demo video ← HERE
└── tests/
    └── real_world_validation/
```

---

## 🚀 Getting Started

### Prerequisites

```bash
# ROS2 Humble
sudo apt install ros-humble-desktop

# OpenCV with ArUco
sudo apt install libopencv-dev
pip install opencv-contrib-python --break-system-packages
```

### Build & Run

```bash
git clone https://github.com/vinny-juniors/wheelchair-fleet.git
cd wheelchair-fleet

colcon build --symlink-install
source install/setup.bash

# Launch follower node
ros2 launch wheelchair_bringup follower.launch.py

# With RViz2 visualization
ros2 launch wheelchair_bringup follower_viz.launch.py
```

---

## 📊 Results

| Metric | Baseline | Optimized | Improvement |
|---|---|---|---|
| Tracking precision | ~±35 cm | **±10 cm** | **−71%** |
| Trajectory error | 100% | 65% | **−35%** |
| Tracking stability | — | **> 92%** | ✅ |

---

## 👤 Author

**Vinny Juniors** — Engineering student, Robotics & Mechatronics · JUNIA-ISEN Lille
📧 vinnyjuniors.ngon@student.junia.com
🔗 [linkedin.com/in/vinny-ngon-708253342](https://linkedin.com/in/vinny-ngon-708253342)


---

## 📄 License

Academic project — JUNIA-ISEN Lille. All rights reserved.