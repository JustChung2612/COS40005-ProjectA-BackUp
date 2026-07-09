# 🔴 Jetracer Training Material 🦾 🦾 🦾

## 🤖 High-Level Objectives:

- Accurately identify and locate the position of the person who sent the signal
- Autonomously navigate and transport the tray containing the water-filled cup from Position B (water dispenser location) to Position A (person's location)
- Detect and avoid obstacles (such as furniture and other objects) during navigation
- Successfully deliver the tray to the person without spilling the water

## 🤖 Technical Stack

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
| **Development OS** | Ubuntu 24.04 + ROS 2 Humble | Native development environment |
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
| **Simulator** | Isaac Sim with ROS 2 Bridge (Humble) |
| **Target OS** | Ubuntu 18 |
| **ROS Distribution** | ROS 2 Humble |
| **Build Tools** | colcon, rosdep |
| **Deployment** | Native (Ubuntu 24.04) on Jetson Nano |

### Dependencies
- ROS 2 Humble
- Nav2 (navigation stack)
- slam_toolbox (SLAM backend)
- OpenCV (vision processing)
- Geometry msgs (ROS2)
- Ackermann steering messages
- colcon (build system)

## 🤖 Setup steps
# JetRacer — Getting Started
 
The physical robot stack. Runs **on the JetRacer** (Waveshare JetRacer, ROS 2
Humble): base driver, RPLidar, EKF odometry, Nav2 navigation, and AprilTag
docking.
 
---
 
## 1. Build
 
The `start_*.sh` scripts source **`ws_setup.bash`**, which sources ROS + every
installed package individually. This is a deliberate workaround for an
incomplete merged `install/setup.bash` on this device — use it instead of
`source install/setup.bash`.
 
```bash
cd jetracer_ws
colcon build --symlink-install
source ws_setup.bash
```
 
> ⚠️ Keep the robot **still for ~2 s** at driver startup while the gyro
> calibrates. Moving during calibration corrupts odometry.
 
---
 
## 2. The layers
 
The stack is split so you can bring up only what you need:
 
| Script                | What it starts                                                     | Publishes                                      |
| --------------------- | ----------------------------------------------------------------- | ---------------------------------------------- |
| `./start_driver.sh`   | Base driver only (`/cmd_vel` → serial)                            | `/odom`, `/imu`                                |
| `./start_lidar.sh`    | RPLidar A1 + `base_footprint→laser_frame` TF                      | `/scan`                                        |
| `./start_hardware.sh` | **driver + lidar + EKF + static TFs + camera/AprilTag** (no Nav2) | `/odom`, `/imu`, `/scan`, `/odometry/filtered` |
| `./start_nav2.sh`     | Nav2 (map_server, AMCL, controller, planner, BT nav)              | drives `/cmd_vel`                              |
| `./start_mapping.sh`  | Nav2 motion + `explore_lite` frontier exploration (no map_server) | autonomous map building                        |
 
`start_hardware.sh` is the base every workflow needs. `start_nav2.sh` and
`start_mapping.sh` both bring up the Nav2 motion nodes — **don't run both.**
 
---
 
## 3. Common workflows
 
### Navigate on a known map
 
Easiest — tmux brings up hardware (left pane) then Nav2 (right pane, after 8 s):
 
```bash
cd jetracer_ws
./start_tmux.sh
# detach: Ctrl-b d   reattach: tmux attach -t jetracer   kill: tmux kill-session -t jetracer
```
 
Or manually, in two terminals:
 
```bash
./start_hardware.sh
./start_nav2.sh map:=/ros2_ws/maps/test_map_outer_v6.yaml
```
 
Maps live in `jetracer_ws/maps/` (`test_map_outer_v6.yaml` is the default). Then
set a Nav2 goal from RViz.
 
### Build a new map
 
1. `./start_hardware.sh`
2. Run SLAM (`slam_toolbox`) against the robot's `/scan` + TF.
3. Drive around — teleop, **or** `./start_mapping.sh` for autonomous frontier
   exploration.
4. Save the map and drop the `.yaml`/`.pgm` into `jetracer_ws/maps/`.
 
### Docking (AprilTag)
 
`start_hardware.sh` also brings up the CSI camera + AprilTag detector.
`jetracer_bringup/scripts/jetracer_docker.py` runs the dock/undock state machine
(sequencing driven by `/docking_state`). Round-trip demo, with the full stack
already running:
 
```bash
./dock_cycle.sh dock1 dock0     # dock A → undock → dock B → undock
```
 
Camera intrinsics and the dock-tag layout are in `jetracer_bringup/config/`.
See `CALIBRATION.md` — docking accuracy depends on the camera TF + calibration.
 
---
 
## 4. Robot Web Bridge (ordering app)
 
The application layer — a FastAPI + HTMX mobile web UI + HTTP API for commanding
the robot (the QR-code "Get Water" / "Refill" ordering flow). It lives in a
separate workspace, **`orchestrator_ws/`**, and talks to this stack over the ROS
graph: it publishes `/dock_robot`, `/abort_docking`, `/cmd_vel`, `/initialpose`
and subscribes `/docking_state` + `/chassis/odom`. So the robot stack from §3
(hardware + Nav2 + docking) must already be running, on the **same
`ROS_DOMAIN_ID`**.
 
### Start it
 
```bash
cd ../orchestrator_ws
./run_web_bridge.sh          # serves on http://<host>:8088
```
 
The script sources `network.env` + ROS, then runs `ros2 run robot_web_bridge
server`. It expects to run inside the Humble container (or a ROS-sourced shell).
Override the port with `ROBOT_WEB_BRIDGE_PORT=9000`, and set the operator PIN via
`ROBOT_WEB_BRIDGE_ADMIN_PIN` for the admin routes.
 
### Expose it for phones (QR codes)
 
In another shell, tunnel the port to a public URL so phones can scan and order:
 
```bash
./run_tunnel.sh              # prints an https://<...>.trycloudflare.com URL
```
 
### Without the robot (dev / demo)
 
```bash
./run_web_bridge_sim.sh      # SimBackend: no ROS, each leg completes on a timer
```
 
Verify the mode at runtime: `GET /api/state` → `{"mode": "robot" | "simulation"}`.
Health check: `GET /healthz`.
 
---
 
## 5. Overriding defaults
 
Extra args pass straight through to the launch files:
 
```bash
./start_hardware.sh base_port:=/dev/ttyACM1 lidar_port:=/dev/ttyACM0
./start_nav2.sh     map:=/ros2_ws/maps/my_map.yaml
```
 
Defaults: base port `/dev/ttyACM0`, lidar port `/dev/ttyACM1`.
 
---
 
## Troubleshooting
 
- **`ros2` or packages (nav2, robot_localization, jetracer_bringup) missing** →
  you sourced the merged setup. Use `source ws_setup.bash`.
- **Odometry drifts from the start** → the robot moved during the ~2 s gyro
  calibration. Restart the driver and hold it still.
- **Wrong serial port** → override with `base_port:=` / `lidar_port:=`.
- **Nav2 won't move / no path** → check `/scan` and TF are live
  (`ros2 topic hz /scan`), and that AMCL is localized on the right map
 
