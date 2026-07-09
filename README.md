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


## Setup steps
### 1. Prerequisites

| Component | Version / notes |
|---|---|
| OS | Ubuntu 24.04 |
| ROS 2 | **Jazzy** (`ros-jazzy-desktop`) |
| Isaac Sim | Any recent release with the ROS 2 Bridge extension enabled |
| Build tools | `colcon`, `rosdep`, `git` |
| Python (perception) | numpy, opencv-python, ultralytics, scipy |

> Isaac Sim's ROS 2 Bridge must be configured for Jazzy (set the bridge to use
> your system ROS, or source your ROS 2 install before launching Isaac).

### 2. Workspace layout

Only these four packages are part of the arm project — everything else is a
standard upstream dependency you install separately (see §3):

| Package | What it is |
|---|---|
| `ra_ws/src/so_arm_description` | SO-ARM 101 URDF + meshes |
| `ra_ws/src/so_arm_moveit_config` | MoveIt 2 config (SRDF, kinematics, OMPL, controllers, `ros2_control`) |
| `ra_ws/src/so_arm_perception` | Cup + tray + AprilTag perception nodes (YOLO / HSV / OpenCV) |
| `ra_ws/src/mtc_tutorial` | `mtc_node` — the grasp → servo → fill → place pipeline, plus launch files |

The Isaac Sim scene (robot, table, mug, tray, dispenser, cameras) lives under
the repo's `simulation/` folder — open it in Isaac Sim before running.

### 3. Install dependencies

#### 3.1 ROS 2 Jazzy + MoveIt 2
```bash
sudo apt update
sudo apt install ros-jazzy-desktop ros-jazzy-moveit \
     ros-jazzy-topic-based-ros2-control \
     ros-jazzy-joint-trajectory-controller \
     ros-jazzy-position-controllers \
     ros-jazzy-joint-state-broadcaster \
     python3-colcon-common-extensions python3-rosdep
```

#### 3.2 MoveIt Task Constructor (MTC)
MTC drives the pick pipeline. If a Jazzy binary is available:
```bash
sudo apt install ros-jazzy-moveit-task-constructor-core
```
Otherwise clone it into `ra_ws/src/` and let `colcon` build it:
```bash
cd ra_ws/src && git clone -b jazzy https://github.com/moveit/moveit_task_constructor.git
```

#### 3.3 Perception Python packages
```bash
python3 -m pip install "numpy>=1.24" "opencv-python>=4.8" "ultralytics>=8.3" "scipy>=1.11"
```
`cv_bridge` comes from apt: `sudo apt install ros-jazzy-cv-bridge`.
The YOLO weights (`yolo11n.pt`) auto-download on first run; no manual step needed.

#### 3.4 Resolve the rest with rosdep
```bash
cd ra_ws
rosdep install --from-paths src --ignore-src -r -y
```

### 4. Build the workspace

```bash
cd ra_ws
source /opt/ros/jazzy/setup.bash
colcon build --symlink-install
source install/setup.bash
```

### 5. Isaac Sim setup (the ROS contract)

The ROS side talks to Isaac Sim through **`topic_based_ros2_control`**. Open the
scene and make sure its ROS 2 action graph publishes/subscribes these topics:

| Direction | Topic | Type | Notes |
|---|---|---|---|
| Isaac → ROS | `/isaac_joint_states` | `sensor_msgs/JointState` | all 6 joints: `Rotation, Pitch, Elbow, Wrist_Pitch, Wrist_Roll, Jaw` |
| ROS → Isaac | `/isaac_joint_commands` | `sensor_msgs/JointState` | position commands; drive the joints to these |
| Isaac → ROS | `/clock` | `rosgraph_msgs/Clock` | everything runs with `use_sim_time:=true` |
| Isaac → ROS | top-cam RGB + `camera_info` | `sensor_msgs/Image`, `CameraInfo` | overhead camera (namespace `top_cam`) |
| Isaac → ROS | arm-cam RGB + depth + `camera_info` | `sensor_msgs/Image`, `CameraInfo` | eye-in-hand camera (namespace `arm_cam`) |

The camera namespaces are parameters of the perception node
(`camera_eth_ns` = `top_cam`, `camera_eih_ns` = `arm_cam`) — match Isaac's camera
topics to these, or override the params.

**Steps:**
1. Open the arm scene (under `simulation/`) in Isaac Sim.
2. Confirm the ROS 2 Bridge extension is enabled and set to ROS 2 Jazzy.
3. Press **Play** (physics + camera render products must be running, or
   `/isaac_joint_states` and the camera topics stay silent).
4. In a sourced terminal, verify the contract:
   ```bash
   ros2 topic hz /isaac_joint_states
   ros2 topic hz /clock
   ros2 topic list | grep -E "top_cam|arm_cam"
   ```

### 6. Run the pipeline

One command brings up MoveIt (`move_group` + controllers + RViz), perception, and
`mtc_node`, staggered so each layer's dependencies are up first:

```bash
source install/setup.bash
ros2 launch mtc_tutorial bringup.launch.py
```

What happens:
1. `move_group`, `ros2_control`, and the `arm_group` / `hand_group` controllers start.
2. Perception starts (YOLO cup detector, pink-tray detector, AprilTag detector).
3. `mtc_node` waits for `/detected_object/position`, then per cup:
   gross move → visual servo onto the mug → close claw → carry to the AprilTag
   dispenser → lean/press to "fill" → place in the tray.

The **number of cups** is not configured — it's however many the **top cam**
detects at start (if none, it defaults to one), and they're spread evenly across
the tray (a single cup is centred).

### 7. Key configuration (launch args)

All are args to `pick_place_demo.launch.py` (forward them through `bringup`):

| Arg | Default | Purpose |
|---|---|---|
| `servo_image_mode` | `true` | image-based (IBVS) arm-cam servo |
| `grasp_yaw_bias` | `-0.5` | approach angle so the mug lands in the single-jaw gap |
| `servo_grasp_z` | `0.05986` | side-grasp height (mug mid-height) |
| `dispenser_standoff` | `0.10` | hold-back from the tag before pressing |
| `dispenser_fill_depth` | `-0.08` | how far the cup ends vs the tag (negative = stops short) |

Example: `ros2 launch mtc_tutorial bringup.launch.py servo_grasp_z:=0.05`.

### 8. Troubleshooting

| Symptom | Likely cause / fix |
|---|---|
| Arm never moves | Isaac not in **Play**, or Isaac isn't subscribed to `/isaac_joint_commands`. Check `ros2 topic echo /isaac_joint_commands`. |
| `mtc_node` blocks on "waiting for /detected_object/position" | Perception isn't publishing — check camera topics are streaming and match the `camera_*_ns` params. |
| Cameras silent | Isaac render products / camera OmniGraph not active while playing. |
| Planning fails "Start state out of bounds" | A joint settled a hair past a URDF limit; the node re-seats before release, but check `joint_limits.yaml`. |
| Everything is slow / time jumps | `/clock` not published, or a node started without `use_sim_time:=true`. |

### 9. Notes for maintainers

- The whole arm stack runs on **sim time** (`use_sim_time:=true`); keep `/clock` flowing.
- The gripper opens via the `hand_group_controller/gripper_cmd` GripperCommand
  action; the arm is commanded on `/arm_group_controller/joint_trajectory`.
- Perception's overhead camera also publishes the static `world → top_sim_camera`
  TF used for ray-plane unprojection (in `perception.launch.py`).
