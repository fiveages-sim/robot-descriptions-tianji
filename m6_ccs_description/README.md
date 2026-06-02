# Tianji Marvin M6-S CCS Description

This package contains the description files for Marvin M6 CCS Manipulator.

Robot models are defined under **`xacro/`** only (no static `urdf/`). OCS2 and `ocs2_arm_controller` generate planning URDF from `xacro/robot.xacro` (cached under `/tmp/ocs2_ros2/m6_ccs/planning_urdf/`). `xacro/eefs.xacro` is also used by `fiveages_w2_description` for split-body control.

## 1. Build

```bash
cd ~/ros2_ws
colcon build --packages-up-to m6_ccs_description --symlink-install
```

## 2. Visualize the robot

### 2.1 Basic Arm Configuration
* Dual Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs collider:=simple
  ```
* Left Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=left collider:=simple
  ```
  ![Marvin M6 CCS](../.images/m6_ccs.png)
* Right Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=right collider:=simple
  ```

### 2.2 Arm Configuration for scene

* USB HIL task with AG2F90-C Gripper
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=usb-90c collider:=simple
  ```
* Dual Arm on Desktop
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=desktop collider:=simple
  ```
* Dual Arm on Desktop with AG2F90-C Gripper
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=desktop_90c collider:=simple
  ```

### 2.3 Arm configuration for humanoid

* Dual Arm with ChangingTek AG2F90-C or AG2F90-D
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=ag2f90_c collider:=simple
  ```
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=ag2f90_d collider:=simple
  ```
* Dual Arm with Jodell RG75
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=rg75 collider:=simple
  ```
* Dual Arm with Jodell RG75 Heavy Pad
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=rg75_heavy collider:=simple
  ```
* Dual Arm with LinkerHand O6
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=linkerhand_o6 collider:=simple
  ```
* Dual Arm with LinkerHand L6
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=linkerhand_l6 collider:=simple
  ```
* Dual Arm with LinkerHand O7
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=linkerhand_o7 collider:=simple
  ```

### 2.4 Asymmetric end effectors (left ≠ right)

`xacro/robot.xacro` accepts **`left_type`** and **`right_type`** instead of a single symmetric `type`. Keys match §2.3 (`rg75`, `ag2f90_c`, `linkerhand_o7`, …). Leave a side empty for no end effector on that arm.

**Note:** `manipulator.launch.py` only forwards symmetric `type:=` today. For asymmetric models, use §3 OCS2 launches or `robot_profile` below.

**Launch args** (`robot_common_launch` prefix routing):

| Prefix | Example | Purpose |
|--------|---------|---------|
| `xacro_left_type` / `xacro_right_type` | `xacro_left_type:=rg75` | Planning / visualization xacro |
| `control_left` / `control_right` | `control_left:=rg75` | ros2_control controller YAML per side (`demo.launch.py` only) |

Do **not** pass `type:=` when both `xacro_left_type` and `xacro_right_type` are set.

**`robot.local.yaml`** (persistent machine config):

```yaml
xacro:
  left_type: rg75
  right_type: linkerhand_o7
control:
  left: rg75
  right: linkerhand_o7
```

```bash
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs robot_profile:=/path/to/robot.local.yaml
```

### 2.5 Arm mount (single source per host robot)

**唯一来源**：各宿主 description 包的 `config/xacro/mount.yaml`（仅 `arm_half_spacing` / `arm_mount_z`）。

| Package | File |
|---------|------|
| `fiveages_w2_description` | `config/xacro/mount.yaml` |
| `m6_ccs_description` | `config/xacro/mount.yaml`（standalone） |
| `m6_ccs_description` | `config/xacro/mount_desktop.yaml`（`type:=desktop` 时由 launch 选用） |

`robot_common_launch` 在 `robot:=…` 时读取该 yaml，注入 xacro mappings，用于：

- 宿主 **ros2_control URDF**（如 W2 整机）
- **分体 OCS2 planning URDF**（`planning_scope=arms`：臂链用 `ocs2_arm_controller.robot_name` 指定的包，**安装位姿仍来自 launch `robot` 宿主包**）

```bash
ros2 launch ocs2_arm_controller split_body.launch.py robot:=fiveages_w2
# 控制 URDF + planning URDF 均使用 fiveages_w2/config/xacro/mount.yaml

ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs type:=desktop
# 使用 m6_ccs/config/xacro/mount_desktop.yaml
```

调试 override（不改变 SSOT 文件）：`xacro_arm_half_spacing:=0.075`

## 3. OCS2 Demo

### 3.1 Official OCS2 Mobile Manipulator Demo

Planning URDF from xacro (no ros2_control). Example with end effector:

  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator_ocs2.launch.py robot_name:=m6_ccs
  # Asymmetric (§2.4):
  # ros2 launch robot_common_launch manipulator_ocs2.launch.py robot_name:=m6_ccs \
  #   xacro_left_type:=rg75 xacro_right_type:=linkerhand_o7
  ```
  
### 3.2 OCS2 arm controller (`demo.launch.py`)

Mock simulation by default (`hardware:=mock_components`). Planning URDF from xacro (same as §3.1).

```bash
source ~/ros2_ws/install/setup.bash

# Dual / single arm
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs type:=left    # or type:=right

# End effector or scene — same `type` as §2.3, e.g.:
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs type:=rg75
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs type:=ag2f90_c
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs type:=usb-90c

# Asymmetric end effectors (§2.4) — omit `type:=`:
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs \
  xacro_left_type:=rg75 xacro_right_type:=linkerhand_o7 \
  control_left:=rg75 control_right:=linkerhand_o7
```

Other symmetric `type` values: `ag2f90_d`, `ag2f90_c_soft`, `rg75_heavy`, `linkerhand_o6`, `linkerhand_l6`, `linkerhand_o7`, `desktop`, `desktop_90c`.

## 4. Real Robot Deploy

### 4.1 Build Hardware Interfaces

```bash
cd ~/ros2_ws
colcon build --packages-up-to marvin_ros2_control --symlink-install
```

### 4.2 Launch Control

#### 4.2.1 Launch without Gripper
* Dual Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs hardware:=real
  ```
* Single Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs hardware:=real type:=left
  ```
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs hardware:=real type:=right
  ```

#### 4.2.2 With ChangingTek AG2F90-C Gripper
* Dual Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs hardware:=real type:=ag2f90_c
  ```