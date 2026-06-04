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

Do **not** pass `type:=` when using per-side types. One launch arg per side updates URDF, planning xacro, and ros2_control compose.

```bash
ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs \
  left_type:=rg75 right_type:=linkerhand_o7 collider:=simple

ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs \
  left_type:=rg75 right_type:=linkerhand_o7
```

Optional: `robot_profile:=/path/to/robot.local.yaml` with `defaults.end_effectors.left` / `.right` (same type keys).

### 2.5 Arm mount (`DualArmMount` macro)

`m6_ccs_description/xacro/arm_mount.xacro` 提供 `DualArmMount`：在已有 `arm_base`（或任意 parent）上固定左右臂基座。

- 默认：`left_xyz="0 0.062 0"`、`right_xyz="0 -0.062 0"`（与 W2 整机一致）
- `type:=desktop` 时在 `robot.xacro` 内写死展台位姿：`0 0.12 0.701` / `0 -0.12 0.701`
- 需要其它间距时，在调用处改 macro 参数，或 launch 传 xacro mapping：`left_xyz:=0.075 0 0`

```xml
<xacro:DualArmMount parent="arm_base"/>
<xacro:DualArmMount parent="arm_base" left_xyz="0 0.12 0.701" right_xyz="0 -0.12 0.701"/>
```

### 2.6 Visual skins (`skin` / `variant`)

`xacro/arm.xacro` 中 `M6_CCS` 通过 **`skin`**（集成方外观）和 **`variant`**（同皮肤下的子款，如中石化）选择 link1–link7 的 visual mesh。**碰撞**始终使用包根目录 `meshes/link*.glb`，不受皮肤影响。整机（躯干/头/底盘 + 臂）的 `platform.variant` 见 `fiveages_w2_description`。

| `skin` | `variant` | Visual mesh 目录 |
|--------|-----------|------------------|
| `default`（省略） | — | `meshes/link*.glb` |
| `fiveages_w2` | — | `meshes/fiveages_w2/` |
| `fiveages_w2` | `sinopec` | `meshes/fiveages_w2/sinopec/` |
| `agibot_g2` | — | `meshes/agibot_g2/` |

以下均为 **双臂** `manipulator.launch`（`type` 默认 `dual`，可省略）：

* 默认天机外观（同 §2.1）
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs collider:=simple
  ```
* FiveAges W2 臂皮肤
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs collider:=simple skin:=fiveages_w2
  ```
* FiveAges W2 + 中石化臂皮肤
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs collider:=simple \
    skin:=fiveages_w2 variant:=sinopec
  ```
* Agibot G2 臂皮肤
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs collider:=simple skin:=agibot_g2
  ```

在宿主 xacro 中引用 `M6_CCS` 时传相同参数，例如 `skin="fiveages_w2" variant="${platform_variant}"`（见 `fiveages_w2_description/xacro/robot.xacro`）。

检查双臂 URDF 中的 mesh 路径：

```bash
source ~/ros2_ws/install/setup.bash
xacro $(ros2 pkg prefix --share m6_ccs_description)/xacro/robot.xacro \
  collider:=simple skin:=fiveages_w2 variant:=sinopec 2>/dev/null | grep 'fiveages_w2/sinopec' | head -3
```

## 3. OCS2 Demo

### 3.1 Official OCS2 Mobile Manipulator Demo

Planning URDF from xacro (no ros2_control). Example with end effector:

  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator_ocs2.launch.py robot_name:=m6_ccs
  # Asymmetric (§2.4):
  # ros2 launch robot_common_launch manipulator_ocs2.launch.py robot_name:=m6_ccs \
  #   left_type:=rg75 right_type:=linkerhand_o7
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
  left_type:=rg75 right_type:=linkerhand_o7
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