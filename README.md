# 🔴 Robot Arm 🦾 🦾 🦾

## High-Level Objectives:

- Autonomously pick up an empty cup from the designated location
- Transport the cup to the water dispenser
- Dispense water by activating the dispenser button
- Detect when the cup is full
- Retrieve the filled cup from the dispenser
- Measure and calculate the weight of the water-filled cup
- Place the cup onto the tray for transport

## Technical Stack

### Overview
The Robot Arm is a **5-DOF SO-ARM 101** with a **1-DOF gripper** designed to grasp cups, move them to a water dispenser, fill them, and place them on a tray for the AMR (JetRacer) to transport.

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Communication & Control** | ROS2 + MoveIt2 | Motion planning, arm control, and perception for grasping and placing operations |
| **Simulation** | Isaac Sim / Isaac Lab | Development, training, and sim-to-real transfer validation |
| **Development OS** | Ubuntu 24.04 + ROS 2 Jazzy | Native development environment |
| **Build System** | colcon, rosdep | Build and dependency management |
| **Compute** | Jetson Nano / Jetson-class compute | Edge AI processing for arm control and perception |

### Packages & Modules

#### 1. so_arm_description
- **URDF** (Unified Robot Description Format) + 3D meshes
- **Joint definitions:** 
  - Rotation
  - Pitch
  - Elbow
  - Wrist_Pitch
  - Wrist_Roll
  - Jaw (gripper)
- Hardware description and kinematics model

#### 2. so_arm_moveit_config
- **SRDF** (Semantic Robot Description Format) configuration
- **Kinematics solvers:** position-only IK for 5-DOF reachability
- **Motion planning:** OMPL (Open Motion Planning Library) using RRTConnect algorithm
- **Integration:** ros2_control + controller integration for joint control

#### 3. so_arm_perception
- **Cup Detection:**
  - YOLO (yolo11n) real-time object detector
  - HSV color-space fallback (for simulation)
- **Tray Detection:** Pink-tray detector (AprilTag-based localization)
- **Dispenser Detection:** AprilTag detector for water dispenser positioning
- **Camera Inputs:**
  - `top_cam` (overhead view)
  - `arm_cam` (eye-in-hand camera mounted on the arm)

#### 4. mtc_tutorial
- **MTC Node (Motion Task Commander):** Orchestrates the complete grasp-and-place pipeline
- **Pipeline Stages:** 
  1. grasp
  2. servo (visual servoing)
  3. fill
  4. place
- Launch files and task definitions

#### Development & Testing

| Aspect | Details |
|--------|---------|
| **Simulator** | Isaac Sim with ROS 2 Bridge (Jazzy) |
| **Target OS** | Ubuntu 24.04 |
| **ROS Distribution** | ROS 2 Jazzy |
| **Build Tools** | colcon, rosdep |
| **Deployment** | Native (Ubuntu 24.04) |

#### Hardware Specifications

| Hardware | Specification | Notes |
|----------|---------------|-------|
| **Robotic Arm** | SO-ARM 101 (5-DOF) | Main manipulator |
| **Gripper** | 1-DOF (Jaw) | Cup grasping mechanism |
| **Compute** | Jetson Nano / Jetson-class | Edge AI processing |
| **RAM** | 4GB (Jetson Nano Legacy) | Sufficient for ROS2 + perception |
