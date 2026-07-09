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

## Setup steps

# JetRacer AMR (`jetracer_ws`) — SLAM + Nav2 on an Ackermann Chassis in Isaac Sim
 
The **workstation-side** control stack for the JetRacer autonomous mobile robot
(AMR): a simulated Ackermann-steered chassis drives around an Isaac Sim scene,
builds a map with `slam_toolbox`, and navigates to goals with Nav2. A small
converter node bridges the gap between Nav2's differential-drive `Twist` output
and the JetRacer's Ackermann steering.
 
> **This runs on the workstation, not on the JetRacer.** There is **no on-device
> JetRacer firmware in this repo yet** — the stack currently drives the robot in
> Isaac Sim and will drive the real chassis once firmware is added. The Isaac →
> ROS contract below is what a real driver would eventually have to satisfy.
 
> **⚠️ ROS distro:** `jetracer_ws` targets **ROS 2 Humble** and runs **inside the
> `Dockerfile.dev` container** — *not* the native Jazzy stack `ra_ws` uses. The
> two still interoperate over DDS as long as they share the same
> `ROS_DOMAIN_ID`.
 
### 1. Prerequisites
 
| Component | Version / notes |
|---|---|
| OS | Ubuntu (Docker host) |
| ROS 2 | **Humble** — provided by the `Dockerfile.dev` image, not installed on the host |
| Docker | with `nvidia-container-toolkit` if you want GPU (`USE_GPU=true`) |
| Isaac Sim | Any recent release with the ROS 2 Bridge extension enabled (set to Humble) |
| Build tools | `colcon`, `rosdep` — already in the container |
 
The Humble toolchain, `slam_toolbox`, Nav2, `pointcloud_to_laserscan`, and the
`ackermann_msgs` deps all ship inside the Dev image, so there is no host-side ROS
install to manage. See the repo [README](../README.md#humble-docker-setup) for
building the image and configuring `network.env`.
 
### 2. Workspace layout
 
Project packages under `jetracer_ws/src/` (the rest are upstream NVIDIA Isaac
samples used as-is):
 
| Package | What it is |
|---|---|
| `ackermann_control/cmdvel_to_ackermann` | Converts Nav2's `/cmd_vel` `Twist` → `/ackermann_cmd` `AckermannDriveStamped` (bicycle-model steering angle from `track_width`) |
| `navigation/slam_custom` | `slam_toolbox` online-async SLAM + a preconfigured RViz view |
| `navigation/carter_navigation` | Nav2 bringup + `pointcloud_to_laserscan` (3D lidar → `/scan`) + RViz |
| `navigation/iw_hub_navigation` | Alternate Nav2 bringup for the iw.hub chassis |
| `isaac_compressed_image_decoder` | Decodes H264 `CompressedImage` from Isaac → raw `Image` |
| `isaacsim`, `isaac_tutorials`, `isaac_ros2_messages`, `custom_message` | Isaac ROS 2 Bridge helpers / message types |
 
The JetRacer Isaac Sim scene lives under the repo's `simulation/` folder — open
it in Isaac Sim before running.
 
### 3. Enter the container and build
 
The Dev container mounts an Isaac ROS workspace at `/ros2_ws`; build the
JetRacer packages there. From the repo root:
 
```bash
# Terminal 1 — start the Humble container (reads network.env, forwards X11):
./jetracer_ws/run_workstation.sh
```
 
Inside the container:
 
```bash
cd /ros2_ws
source /opt/ros/humble/setup.bash
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install
source install/setup.bash
```
 
Set `USE_GPU=true` before the run script if you have an NVIDIA GPU +
`nvidia-container-toolkit`. Attach more terminals to the same container with
`docker exec -it isaacsim_humble_ws bash` (then re-source ROS + the workspace).
 
### 4. Isaac Sim setup (the ROS contract)
 
Open the JetRacer scene, enable the ROS 2 Bridge extension (set to **Humble**),
and press **Play**. The stack expects the scene's action graph to publish/subscribe:
 
| Direction | Topic | Type | Notes |
|---|---|---|---|
| ROS → Isaac | `/ackermann_cmd` | `ackermann_msgs/AckermannDriveStamped` | drive speed + steering angle; what moves the chassis |
| Isaac → ROS | `/front_2d_lidar/scan` | `sensor_msgs/LaserScan` | used directly by SLAM |
| Isaac → ROS | `/front_3d_lidar/lidar_points` | `sensor_msgs/PointCloud2` | flattened to `/scan` by Nav2's `pointcloud_to_laserscan` |
| Isaac → ROS | `/clock` | `rosgraph_msgs/Clock` | everything runs with `use_sim_time:=true` |
| Isaac → ROS | TF | `tf2` | tree is `odom → base_footprint → laser_frame` (there is **no** `base_link`) |
 
Verify the contract in a sourced terminal:
 
```bash
ros2 topic hz /clock
ros2 topic hz /front_2d_lidar/scan
ros2 topic echo /ackermann_cmd --once     # after you send a /cmd_vel
```
 
### 5. Run the stack
 
**5a. Ackermann bridge** — turn `Twist` commands into steering (needed for Nav2
and for manual teleop):
 
```bash
ros2 launch cmdvel_to_ackermann cmdvel_to_ackermann.launch.py
# then, e.g., drive manually:
ros2 run teleop_twist_keyboard teleop_twist_keyboard      # publishes /cmd_vel
```
 
**5b. SLAM** — build a map:
 
```bash
ros2 launch slam_custom slam_custom.launch.py
```
 
This starts `slam_toolbox` (online-async) plus RViz with the project view. Drive
the robot around (5a) to build the map, then save it:
 
```bash
ros2 run nav2_map_server map_saver_cli -f ~/my_map
```
 
**5c. Nav2** — navigate against a saved map:
 
```bash
ros2 launch carter_navigation carter_navigation.launch.py map:=/path/to/my_map.yaml
```
 
Set the initial pose and send goals from RViz. The pipeline is:
**Nav2 → `/cmd_vel` → `cmdvel_to_ackermann` → `/ackermann_cmd` → Isaac.**
 
### 6. Key configuration (launch args)
 
`cmdvel_to_ackermann` (also settable as node params):
 
| Arg | Default | Purpose |
|---|---|---|
| `track_width` | `0.24` | wheelbase used to convert `v, ω` → steering angle (m) |
| `publish_period_ms` | `20` | `/ackermann_cmd` publish rate |
| `acceleration` | `0.0` | `0` = change speed as fast as possible (m/s²) |
| `steering_velocity` | `0.0` | `0` = change steering angle as fast as possible (rad/s) |
 
`slam_custom`:
 
| Arg | Default | Purpose |
|---|---|---|
| `use_sim_time` | `True` | use Isaac's `/clock`; set `false` for real hardware |
| `slam_params_file` | package `slam_toolbox_params.yaml` | override SLAM tuning |
| `startup_delay` | `5.0` | seconds to let the sim clock stabilise before SLAM starts |
 
`carter_navigation`: `map`, `params_file`, `use_sim_time`.
 
### 7. Troubleshooting
 
| Symptom | Likely cause / fix |
|---|---|
| Robot never moves | Isaac not in **Play**, or not subscribed to `/ackermann_cmd`. Check `ros2 topic echo /ackermann_cmd` while teleoping. |
| `/cmd_vel` sent but no motion | `cmdvel_to_ackermann` not running — it's the bridge Nav2/teleop depend on. |
| SLAM map is empty / no scan | `/front_2d_lidar/scan` silent, or the lidar OmniGraph isn't active while playing. Check `ros2 topic hz /front_2d_lidar/scan`. |
| Nav2 has no `/scan` | `pointcloud_to_laserscan` needs `/front_3d_lidar/lidar_points` streaming from Isaac. |
| TF errors about `base_link` | This robot uses `base_footprint`, not `base_link` — check remaps/params reference the right frame. |
| Everything is slow / time jumps | `/clock` not published, or a node started without `use_sim_time:=true`. |
 
### 8. Notes for maintainers
 
- The whole stack runs on **sim time** (`use_sim_time:=true`); keep `/clock` flowing.
- The steering conversion is a bicycle model: `steering = atan(track_width / (v/ω))`;
  it emits `0` steering for pure-rotation commands (a car can't turn in place).
- TF tree is `odom → base_footprint → laser_frame` — there is deliberately **no**
  `base_link`; frame params across SLAM/Nav2 assume `base_footprint`.
 
---


