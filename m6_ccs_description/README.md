# Tianji Marvin M6-S CCS Description

Marvin M6 CCS 双臂机械臂 URDF / ros2_control 描述包。模型仅在 **`xacro/`**（无静态 `urdf/`）。OCS2 规划 URDF 由 `xacro/robot.xacro` 生成（缓存于 `/tmp/ocs2_ros2/m6_ccs/planning_urdf/`）。`xacro/eefs.xacro` 亦被 `fiveages_w2_description` 复用。

```bash
cd ~/ros2_ws && colcon build --packages-up-to m6_ccs_description --symlink-install
source install/setup.bash
```

## 1. 可视化与 Launch

通用入口：`robot_common_launch manipulator.launch.py robot:=m6_ccs`  
控制 / 真机：`ocs2_arm_controller demo.launch.py robot:=m6_ccs`  
OCS2 规划：`robot_common_launch manipulator_ocs2.launch.py robot_name:=m6_ccs`

| 场景 | 示例 |
|------|------|
| 双臂（默认） | `… manipulator.launch.py robot:=m6_ccs collider:=simple` |
| 单臂 | 加 `type:=left` 或 `type:=right` |
| 末端（对称） | 加 `type:=<key>`（如 `erg32`、`rg75`），见 [§2](#2-末端执行器) |
| 末端（左右不同） | `left_type:=… right_type:=…`，**不要**再传 `type:=` |
| 展台 | `variant:=desktop`（与 `type` 正交，见 [§3](#3-平台与外观)） |
| 真机 | `demo.launch.py … hardware:=real` 或 `real_usb`；驱动方式见 [§2.1](#21-真机驱动方式) |
| Mock 控制 | `demo.launch.py …`（默认 `hardware:=mock_components`） |

![Marvin M6 CCS](../.images/m6_ccs.png)

**不对称末端示例**

```bash
ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs \
  left_type:=rg75 right_type:=linkerhand_o7 collider:=simple
```

也可在 `robot_profile` / `robot.local.yaml` 的 `defaults.end_effectors.left` / `.right` 中配置（键名同 §2）。

## 2. 末端执行器

类型映射：`xacro/eef_type_resolver.xacro`；真机接线：`xacro/ros2_control/robot.xacro`。

### 2.1 真机驱动方式

每侧独立配置：`left_ee_drive_type` / `right_ee_drive_type`（launch 或 `robot.local.yaml`）。

| 驱动键 | 典型 launch | 物理链路 | 插件 / 说明 |
|--------|-------------|----------|-------------|
| `marvin` | `hardware:=real`（默认） | 臂体 **RS485**（Marvin COM1） | [`marvin_ros2_control`](https://github.com/fiveages-sim/marvin-ros2-control#1-支持的末端执行器) |
| `tianji_can` | `left_ee_drive_type:=tianji_can` 等 | 臂体 **场总线 CAN**（`ee_channel=1`） | 同上；**仅部分灵巧手**，见下表 |
| `usb` | `hardware:=real_usb`（默认） | 外接 **USB 串口** | [`modbus_ros2_control`](https://github.com/fiveages-sim/modbus-ros2-control#1-支持的末端执行器) |
| `usb_can` | `left_ee_drive_type:=usb_can` 等 | 外接 **SocketCAN**（`can0` / `can1`） | [`can_ros2_control`](https://github.com/fiveages-sim/can-ros2-control#1-支持的末端执行器) |

夹爪**仅** `marvin`（RS485）。**无** 外接 USB/CAN 夹爪路径。

### 2.2 类型与支持矩阵

| Type key | 产品 | `marvin` | `tianji_can` | `usb` | `usb_can` |
|----------|------|:--------:|:------------:|:-----:|:---------:|
| `rg75` | Jodell RG75-300 | ✅ | — | — | — |
| `erg32` | Jodell ERG32（旋转 + 夹爪，2-DOF） | ✅ | — | — | — |
| `ag2f90`, `ag2f90_d` | ChangingTek AG2F90-D | ✅ | — | — | — |
| `ag2f90_c`, `ag2f90_c_soft` | ChangingTek AG2F90-C | ✅ | — | — | — |
| `ag2f120s`, `ag2f120s_d` | ChangingTek AG2F120S | ✅ | — | — | — |
| `epgi180` | Eincinx EPGI180 | ✅ | — | — | — |
| `linkerhand_o6` / `l6` / `o7` | LinkerHand | ✅ | — | ✅ | — |
| `freedom_v1` | Freedom V1 | ✅ | ✅ | ✅ | ✅ |
| `freedom_v2` | Freedom V2 | ✅ | — | ✅ | — |
| `inspire_e2` | Inspire RH56E2 | — | — | ✅ | ✅ |
| `inspire_f2` | Inspire RH56F2 | ✅ | ✅ | ✅ | ✅ |
| `xhand1` | XHand1 | — | — | ✅ | — |
| `theohand_std16a` | TheoHand STD16A | ✅ | — | ✅ | — |

要点：

- **`tianji_can` ≠ 全部 CAN 化** — 仅 **Freedom V1**、**Inspire F2** 走臂体场总线 CAN/CAN FD；其余末端仍用 RS485（`marvin`）或外接总线。
- **`freedom_v2`** — Marvin 仅 RS485；`tianji_can` 与 `usb_can` **均不支持**（V2 外接请用 `usb` + Modbus）。
- **`inspire_e2`** — 仿真 / 外接 `usb`·`usb_can`；**不走 Marvin**。现场 Inspire 手请用 `inspire_f2`。
- **`usb_can`** — 仅 **Freedom V1**、**Inspire E2/F2**；LinkerHand 无外接 CAN 接线（`can_ros2_control/O6CanHardware` 未在 m6_ccs 中启用）。

## 3. 平台与外观

### 3.1 `variant:=desktop`（m6_ccs 展台）

展台网格、臂间距、OCS2 `desktop.info`。与末端 `type` 组合，例如：

```bash
ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs \
  variant:=desktop type:=rg75 collider:=simple
```

`robot.local.yaml` 片段：

```yaml
platform:
  variant: desktop
defaults:
  end_effectors:
    type: rg75
hardware:
  device_ip: 192.168.10.190
  arm_ctrl_mode: JOINT_IMPEDANCE
  max_joint_speed: 50
  max_joint_acceleration: 30
  usb_left_port: /dev/ttyUSB0
  usb_right_port: /dev/ttyUSB2
```

Launch 可覆盖：`hardware_device_ip:=…`、`hardware_max_joint_speed:=…`、`hardware_max_joint_acceleration:=…`、`hardware_usb_left_port:=…`、`hardware_arm_ctrl_mode:=…`、`hardware_left_dyn_param:=…`。

**迁移：** 已移除 `type:=desktop` / `type:=desktop_90c` → 改用 `variant:=desktop` + 独立 `type`（如 `ag2f90_c_soft`）。详见 [m6_ccs_dual_desktop.md](../docs/m6_ccs_dual_desktop.md)。

### 3.2 `skin` / `variant`（臂外观）

`xacro/arm.xacro` 中 `M6_CCS` 的 **visual** 由 `skin` / `variant` 选择；**碰撞** 始终用 `meshes/link*.glb`。

| `skin` | `variant` | Visual |
|--------|-----------|--------|
| 默认 | — | `meshes/link*.glb` |
| `fiveages_w2` | — | `meshes/fiveages_w2/` |
| `fiveages_w2` | `sinopec` | `meshes/fiveages_w2/sinopec/` |
| `agibot_g2` | — | `meshes/agibot_g2/` |

```bash
ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs \
  collider:=simple skin:=fiveages_w2 variant:=sinopec
```

### 3.3 `DualArmMount`（臂安装位姿）

`xacro/arm_mount.xacro`：默认间距 `0 ±0.062 m`；`variant:=desktop` 时为 `0 ±0.12 m, z=0.701`。

```xml
<xacro:DualArmMount parent="arm_base"/>
<xacro:DualArmMount parent="arm_base" left_xyz="0 0.12 0.701" right_xyz="0 -0.12 0.701"/>
```

## 4. 真机部署

```bash
colcon build --packages-up-to marvin_ros2_control modbus_ros2_control can_ros2_control --symlink-install
source install/setup.bash

# 臂体 RS485 + 夹爪 / 手（Marvin）
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs hardware:=real type:=ag2f90_c
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs hardware:=real type:=erg32
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs hardware:=real type:=inspire_f2

# 臂体场总线 CAN（仅 freedom_v1 / inspire_f2）
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs hardware:=real type:=inspire_f2 \
  left_ee_drive_type:=tianji_can right_ee_drive_type:=tianji_can

# 外接 USB 串口（Modbus）
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs hardware:=real_usb type:=linkerhand_o7

# 外接 SocketCAN（仅 freedom_v1 / inspire_e2 / inspire_f2）
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs hardware:=real_usb type:=inspire_f2 \
  left_ee_drive_type:=usb_can right_ee_drive_type:=usb_can
```

## 5. ros2_control 共享宏

`xacro/ros2_control/robot.xacro` 为薄封装；可复用模块供 `fiveages_w2_description` 等 include：

| 文件 | 宏 | 说明 |
|------|-----|------|
| `interfaces.xacro` | `m6_interfaces` | 单臂 7-DOF 关节接口 |
| `end_effector_interfaces.xacro` | `side_hand_interfaces`、`side_gripper_joint`、`dual_arm_ee_block` | 末端接口；真机夹爪 collapsed 单关节，仿真 full 链 |
| `marvin_hardware.xacro` | `marvin_hardware_block` | Marvin 插件参数（含 `ee_dyn_param`） |
| `external_ee_systems.xacro` | `external_ee_systems`、`side_usb_hand_system` | 外接 usb / usb_can；串口默认左 `/dev/ttyUSB0`、右 `/dev/ttyUSB2`（`usb_left_port` / `usb_right_port`，各类型手共用） |

仿真硬件插件见 [`robot_common_launch`](../../common/robot_common_launch/xacro/ros2_control/simulation_hardware.xacro)（`simulation_hardware_plugin`：mock / gz / isaac）。

外接 USB 串口：`usb_left_port` / `usb_right_port`（默认左 `/dev/ttyUSB0`、右 `/dev/ttyUSB2`）。在 `robot.local.yaml` 的 `hardware:` 中配置，或 launch 使用 `hardware_usb_left_port:=…`。

URDF 层复用：`eef_type_resolver.xacro`、`eefs.xacro`（`ee_dyn_param`）。launch 入参（`device_ip`、`left_ee_drive_type` 等）仍在各机器人顶层 `robot.xacro` 声明，经宏 params 传入共享块。
