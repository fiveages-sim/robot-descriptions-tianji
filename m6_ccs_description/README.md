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
| 末端（对称） | 加 `type:=<key>`（如 `erg32`、`rg75`、`wuji_hand2`），见 [§2](#2-末端执行器) |
| 末端（左右不同） | `left_type:=… right_type:=…`，**不要**再传 `type:=` |
| 展台 | `variant:=desktop`（与 `type` 正交，见 [§3](#3-平台与外观)） |
| 真机 | `demo.launch.py … hardware:=real`；末端驱动方式见 [§2.1](#21-真机驱动方式) |
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
| `usb` | `left_ee_drive_type:=usb` 等 | 外接 **USB 串口** | [`modbus_ros2_control`](https://github.com/fiveages-sim/modbus-ros2-control#1-支持的末端执行器) |
| `usb_can` | `left_ee_drive_type:=usb_can` 等 | 外接 **SocketCAN**（`can0` / `can1`） | [`can_ros2_control`](https://github.com/fiveages-sim/can-ros2-control#1-支持的末端执行器) |
| `ethernet` | `left_ee_drive_type:=ethernet` 等 | 外接 **以太网**（整机扫描，不写 IP） | `wuji_ros2_control`；**仅** `wuji_hand2` |

夹爪默认 `marvin`（臂体 RS485）。独立外接 **RS485** 仅 **rg75**、**ag2f90 / ag2f90_d**（配置键仍是 `usb`，串口设备不一定是 USB）；其余夹爪无独立外接 / `usb_can` 路径。

### 2.2 类型与支持矩阵

| Type key | 产品 | `marvin` | `tianji_can` | `usb` | `usb_can` | `ethernet` |
|----------|------|:--------:|:------------:|:-----:|:---------:|:---------:|
| `rg75` | Jodell RG75-300 | ✅ | — | ✅ | — | — |
| `erg32` | Jodell ERG32（旋转 + 夹爪，2-DOF） | ✅ | — | — | — | — |
| `ag2f90`, `ag2f90_d` | ChangingTek AG2F90-D | ✅ | — | ✅ | — | — |
| `ag2f90_c`, `ag2f90_c_soft` | ChangingTek AG2F90-C | ✅ | — | — | — | — |
| `ag2f120s`, `ag2f120s_d` | ChangingTek AG2F120S | ✅ | — | — | — | — |
| `epgi180` | Eincinx EPGI180 | ✅ | — | — | — | — |
| `marvin_gripper` | Marvin straight flange gripper | ✅ | — | — | — | — |
| `marvin_gripper45` | Marvin 45° flange gripper | ✅ | — | — | — | — |
| `linkerhand_o6` / `l6` / `o7` | LinkerHand | ✅ | — | ✅ | ✅ | — |
| `freedom_v1` | Freedom V1 | ✅ | ✅ | ✅ | ✅ | — |
| `freedom_v2` | Freedom V2 | ✅ | — | ✅ | — | — |
| `inspire_e2` | Inspire RH56E2 | — | — | ✅ | ✅ | — |
| `inspire_f2` | Inspire RH56F2 | ✅ | ✅ | ✅ | ✅ | — |
| `xhand1` | XHand1 | — | — | ✅ | — | — |
| `theohand_std16a` | TheoHand STD16A | ✅ | — | ✅ | — | — |
| `wuji_hand2` | Wuji Hand2（舞肌） | — | — | — | — | ✅ |

要点：

- **`tianji_can` ≠ 全部 CAN 化** — 仅 **Freedom V1**、**Inspire F2** 走臂体场总线 CAN/CAN FD；其余末端仍用 RS485（`marvin`）或外接总线。
- **`freedom_v2`** — Marvin 仅 RS485；`tianji_can` 与 `usb_can` **均不支持**（V2 外接请用 `usb` + Modbus）。
- **`inspire_e2`** — 仿真 / 外接 `usb`·`usb_can`；**不走 Marvin**。现场 Inspire 手请用 `inspire_f2`。
- **`usb_can`** — 支持 **LinkerHand O6/L6/O7**、**Freedom V1**、**Inspire E2/F2**；LinkerHand 根据型号分别使用 `O6CanHardware`、`L6CanHardware`、`O7CanHardware`。
- **外接 485 夹爪** — 仅 **rg75**、**ag2f90 / ag2f90_d**（独立 side system）。物理链路是外接 RS485，配置键仍写 `usb`（`usb_left_port` / `usb_right_port` 可以是 `/dev/ttyUSB*`、板载串口等，**不一定是 USB 接入**）。其余夹爪仍只走臂体 `marvin` RS485，无 `usb_can`。
- **`wuji_hand2`** — 仅 **`ethernet`**。整机挂臂时走手包 `side_systems` + `wuji_scan()`（按 `hand_side`），**不要**在 `robot.local.yaml` 写地址。臂仍 `hardware:=real`（Marvin），手不走 RS485/USB/CAN。可用 `left_ee_rw_rate` / `right_ee_rw_rate` 覆盖手侧 `rw_rate`（未写默认 **200**）。仿真可直接 `type:=wuji_hand2`（`mock_components`）。

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
    type: rg75                 # 或 left/right 分写
  ft:
    type: kwr75_485            # 或 left/right；随 profile 生效（不受 use_profile_eef）
  tcp_offset:
    xyz: "0 0 0"               # metres
    rpy: "0 0 ${PI/2}"         # radians；支持 ${PI/2} / pi/2 / radians(90)
hardware:
  device_ip: 192.168.10.190
  arm_ctrl_mode: JOINT_IMPEDANCE
  max_joint_speed: 50
  max_joint_acceleration: 30
  usb_left_port: /dev/ttyUSB0
  usb_right_port: /dev/ttyUSB2
  # left_dyn_param / right_dyn_param: 可选负载覆盖（空列表跳过）
```

Launch 可覆盖（CLI > profile）：`ft:=…`、`left_ft:=…`、`tcp_offset_rpy:="0 0 ${PI/2}"`、`left_tcp_offset_xyz:=…`，以及 `hardware_device_ip:=…`、`hardware_arm_ctrl_mode:=…`、`hardware_left_dyn_param:=…` 等。

**迁移：** 已移除 `type:=desktop` / `type:=desktop_90c` → 改用 `variant:=desktop` + 独立 `type`（如 `ag2f90_c_soft`）。详见 [m6_ccs_dual_desktop.md](../docs/m6_ccs_dual_desktop.md)。
FT 从 `defaults.end_effectors.left_ft` 迁到 `defaults.ft`；tcp_offset 从 `platform` 迁到 `defaults.tcp_offset`。

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
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs hardware:=real type:=theohand_std16a

# 臂体场总线 CAN（仅 freedom_v1 / inspire_f2）
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs hardware:=real type:=inspire_f2 \
  left_ee_drive_type:=tianji_can right_ee_drive_type:=tianji_can

# 外接 USB 串口（Modbus，灵巧手）
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs hardware:=real type:=linkerhand_o7 \
  left_ee_drive_type:=usb right_ee_drive_type:=usb
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs hardware:=real type:=theohand_std16a \
  left_ee_drive_type:=usb right_ee_drive_type:=usb

# 外接 RS485 夹爪（rg75 / 90D；drive_type 仍写 usb，串口不一定是 USB）
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs hardware:=real type:=rg75 \
  left_ee_drive_type:=usb right_ee_drive_type:=usb
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs hardware:=real type:=ag2f90_d \
  left_ee_drive_type:=usb right_ee_drive_type:=usb

# 外接 SocketCAN（LinkerHand O6/L6/O7、freedom_v1、inspire_e2/f2）
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs hardware:=real type:=linkerhand_l6 \
  left_ee_drive_type:=usb_can right_ee_drive_type:=usb_can
```

## 5. ros2_control 共享宏

`xacro/ros2_control/robot.xacro` 为薄封装；可复用模块供 `fiveages_w2_description` 等 include：

| 文件 | 宏 | 说明 |
|------|-----|------|
| `interfaces.xacro` | `m6_interfaces` | 单臂 7-DOF 关节接口 |
| `end_effector_interfaces.xacro` | `side_hand_interfaces`、`side_gripper_joint`、`dual_arm_ee_block` | 末端接口；真机夹爪 collapsed 单关节，仿真 full 链 |
| `marvin_hardware.xacro` | `marvin_hardware_block` | Marvin 插件参数（含 `ee_dyn_param`） |
| `external_ee_systems.xacro` | `external_ee_systems`、`side_usb_hand_system`、`linkerhand_usb_can_side_system` | 外接 usb / usb_can；串口和 CAN 接口均支持左右独立配置 |

仿真硬件插件见 [`robot_common_launch`](../../common/robot_common_launch/xacro/ros2_control/simulation_hardware.xacro)（`simulation_hardware_plugin`：mock / gz / isaac）。

外接 USB 串口：`usb_left_port` / `usb_right_port`（默认左 `/dev/ttyUSB0`、右 `/dev/ttyUSB2`）。在 `robot.local.yaml` 的 `hardware:` 中配置，或 launch 使用 `hardware_usb_left_port:=…`。

URDF 层复用：`eef_type_resolver.xacro`、`eefs.xacro`（`ee_dyn_param`）。launch 入参（`device_ip`、`left_ee_drive_type` 等）仍在各机器人顶层 `robot.xacro` 声明，经宏 params 传入共享块。
