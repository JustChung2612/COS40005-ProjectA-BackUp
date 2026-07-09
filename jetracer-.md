# 🔴 Jetracer Training Material 🦾 🦾 🦾

## High-Level Objectives:

- Accurately identify and locate the position of the person who sent the signal
- Autonomously navigate and transport the tray containing the water-filled cup from Position B (water dispenser location) to Position A (person's location)
- Detect and avoid obstacles (such as furniture and other objects) during navigation
- Successfully deliver the tray to the person without spilling the water

## Technical Stack

### Overview

The JetRacer is an **Autonomous Mobile Robot (AMR)** running on a **Jetson Nano** device. Its primary responsibility is to navigate from a pickup station to delivery destinations, avoiding obstacles using onboard sensing.

### Technology Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Navigation & SLAM** | ROS2 + SLAM + Nav2 | Navigation, localization/SLAM, obstacle avoidance |
| **Hardware Base** | JetRacer on Jetson Nano | Mobile platform for autonomous navigation |
| **Sensors** | LiDAR, Wheel Encoders, Camera | Navigation, localization, obstacle detection |
| **Drive Interface** | Ackermann Control | Converts command velocity to Ackermann drive commands |
| **Compute** | Jetson Nano (Legacy) | 4GB RAM edge AI processing |
| **Development OS** | Ubuntu 24.04 + ROS 2 Jazzy | Native development environment |
| **Build System** | colcon, rosdep | Build and dependency management |

### Package Breakdown

#### 1. ackermann_control (cmdvel_to_ackermann)
- **Drive interface** for Ackermann steering
- Converts Nav2's `/cmd_vel` (geometry_msgs/Twist) → `/ackermann_cmd` (AckermannDrive)
- Enables steering control on JetRacer platform

#### 2. navigation/slam_custom
- **SLAM bring-up** configuration
- Launches `slam_toolbox` (online-async mode)
- Includes preconfigured `slam_custom.rviz` for visualization
- Real-time mapping and localization

### 3. navigation/carter_navigation
- **Nav2 configuration** (adapted from Isaac's Carter sample)
- Maps configuration files (*_navigation.yaml)
- Navigation parameters (carter_navigation_params.yaml)
- RViz configuration for visualization
- Tailored for JetRacer AMR platform

#### 4. slam_toolbox
- **SLAM backend** (upstream ROS2 package)
- Saved maps:
  - test_map_outer_v*
  - my_map_*
  - Other test environments
- Online async SLAM for real-time mapping

### Hardware Specifications

| Component | Specification | Notes |
|-----------|---------------|-------|
| **Robot Base** | JetRacer | Mobile platform |
| **Compute** | Jetson Nano (Legacy) | 4GB RAM, entry-level edge AI |
| **LiDAR** | Light Detection and Ranging | Distance measurement, point cloud generation, obstacle detection |
| **Wheel Encoders** | Rotation sensors | Odometry, distance/speed estimation |
| **Camera** | Onboard camera | Navigation support, visual feedback |

### Key Sensors & Capabilities

| Sensor | Purpose | Output |
|--------|---------|--------|
| **LiDAR** | Obstacle detection & mapping | Point cloud for SLAM/Nav2 |
| **Wheel Encoders** | Odometry estimation | Position & speed tracking |
| **Camera** | Visual navigation | Real-time environment context |

### Core Technologies

#### ROS2 + Nav2 Stack
- **SLAM (Simultaneous Localization and Mapping):** Building maps while tracking position
- **Localization:** Determining robot position within known map
- **Nav2:** Path planning, control, and obstacle avoidance
- **Pub/Sub Messaging:** Node-to-node communication via topics and services

### Development & Testing
| Aspect | Details |
|--------|---------|
| **Simulator** | Isaac Sim with ROS 2 Bridge (Jazzy) |
| **Target OS** | Ubuntu 24.04 |
| **ROS Distribution** | ROS 2 Jazzy |
| **Build Tools** | colcon, rosdep |
| **Deployment** | Native (Ubuntu 24.04) on Jetson Nano |


## Dependencies

- ROS 2 Jazzy
- Nav2 (navigation stack)
- slam_toolbox (SLAM backend)
- OpenCV (vision processing)
- Geometry msgs (ROS2)
- Ackermann steering messages
- colcon (build system)
