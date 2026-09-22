# Tianji Marvin M20S-CCS Description

Marvin M20S CCS 双臂 URDF / ros2_control 描述包。模型在 **`xacro/`**。OCS2 规划 URDF 由 `xacro/robot.xacro` 生成。末端类型与挂载由本包 `eef_type_resolver.xacro` / `eefs.xacro` 定义。真机经 `marvin_ros2_control/MarvinHardware` 驱动（接线方式参考 M6-CCS）。

```bash
cd ~/ros2_ws && colcon build --packages-up-to m20s_ccs_description --symlink-install
source install/setup.bash
```

## 1. 可视化与 Launch

通用入口：`robot_common_launch manipulator.launch.py robot:=m20s_ccs`  
控制：`ocs2_arm_controller demo.launch.py robot:=m20s_ccs`  
OCS2 规划：`robot_common_launch manipulator_ocs2.launch.py robot_name:=m20s_ccs`

| 场景 | 示例 |
|------|------|
| 双臂（默认） | `… manipulator.launch.py robot:=m20s_ccs collider:=simple` |
| 单臂 | 加 `type:=left` 或 `type:=right` |
| 末端（对称） | 加 `type:=<key>`（如 `rg75`、`ag2f90`），见 [§2](#2-末端执行器) |
| 末端（左右不同） | `left_type:=… right_type:=…`，**不要**再传 `type:=` |
| 真机 | `demo.launch.py … hardware:=real`；末端驱动方式见 [§2.1](#21-真机驱动方式) |
| Mock 控制 | `demo.launch.py …`（默认 `hardware:=mock_components`） |

![Marvin M20S CCS](../.images/m20s_ccs.png)

**不对称末端示例**

```bash
ros2 launch robot_common_launch manipulator.launch.py robot:=m20s_ccs \
  left_type:=rg75 right_type:=linkerhand_o7 collider:=simple
```

`type:=left` / `right` 只选单臂拓扑，不是末端键。

## 2. 末端执行器

类型映射：`xacro/eef_type_resolver.xacro`。几何挂载：`xacro/eefs.xacro`。真机接线：`xacro/ros2_control/robot.xacro`。

### 2.1 真机驱动方式

每侧独立配置：`left_ee_drive_type` / `right_ee_drive_type`（launch 或 `robot.local.yaml` 的 `hardware:` 段）。

| 驱动键 | 典型 launch | 物理链路 | 插件 / 说明 |
|--------|-------------|----------|-------------|
| `marvin` | `hardware:=real`（默认） | 臂体 **RS485**（Marvin COM1） | [`marvin_ros2_control`](https://github.com/fiveages-sim/marvin-ros2-control#1-支持的末端执行器) |
| `tianji_can` | `left_ee_drive_type:=tianji_can` 等 | 臂体 **场总线 CAN**（`ee_channel=1`） | 同上 |
| `usb` | `left_ee_drive_type:=usb` 等 | 外接 **USB / 板载串口** | [`modbus_ros2_control`](https://github.com/fiveages-sim/modbus-ros2-control#1-支持的末端执行器) |
| `usb_can` | `left_ee_drive_type:=usb_can` 等 | 外接 **SocketCAN**（`can0` / `can1`） | [`can_ros2_control`](https://github.com/fiveages-sim/can-ros2-control#1-支持的末端执行器) |

> **依赖提示**：`usb` / `usb_can` 的外接 side system 由本包 `xacro/ros2_control/external_ee_systems.xacro` **自包含**提供（内联定义 LinkerHand / Jodell RG75 / ChangingTek AG2F90-D 的插件宏），不依赖 `m6_ccs_description`，也不依赖公共描述包的 `side_systems.xacro`。仅覆盖 §2.2 中列出外接路径的末端。

### 2.2 支持矩阵

| Type key | 产品 | `marvin` | `tianji_can` | `usb` | `usb_can` |
|----------|------|:--------:|:------------:|:-----:|:---------:|
| `rg75` / `rg75_heavy` | Jodell RG75 | ✅ | ✅ | ✅ | — |
| `erg32` | Jodell ERG32（旋转 + 夹爪，2-DOF） | ✅ | ✅ | — | — |
| `ag2f90`, `ag2f90_d` | ChangingTek AG2F90-D | ✅ | ✅ | ✅ | — |
| `ag2f90_c`, `ag2f90_c_soft` | ChangingTek AG2F90-C | ✅ | ✅ | — | — |
| `ag2f120s`, `ag2f120s_d` | ChangingTek AG2F120S | ✅ | ✅ | — | — |
| `linkerhand_o6` / `linkerhand_l6` / `linkerhand_o7` | LinkerHand | ✅ | ✅ | ✅ | ✅ |

要点：

- **真机夹爪**（rg75 / ag2f90 / ag2f120s）在臂体总线上折叠为单关节 `${side}_gripper_joint`（仅 `position` 命令 / 状态）。
- **ERG32** 是 2-DOF 工具，真机下保留完整接口（`${side}_hand_rotate_joint` / `${side}_hand_gripper_joint`）。
- **只读接口名**与仿真一致（`left_gripper_joint` / `right_gripper_joint`），`adaptive_gripper_controller` 配置无需区分仿真与真机。

## 3. 平台

双臂默认按 M6-CCS 侧挂：`arm_base` 上左右臂 `xyz = (0, ±0.062, 0)`，`rpy = (∓π/2, 0, 0)`。安装位姿由本包 `xacro/arm_mount.xacro` 的 `DualArmMount` 参数覆盖。

`collider:=simple` 时各连杆用包围盒拟合的圆柱碰撞体；`convex` 用 glb mesh。OCS2 `task.info` 已开启双臂自碰撞（`arm_base` 与左右 `link3/5/7`）。

## 4. 真机部署

```bash
colcon build --packages-up-to marvin_ros2_control modbus_ros2_control can_ros2_control --symlink-install
source install/setup.bash

# 臂体 RS485 + 夹爪 / 手（Marvin）
ros2 launch ocs2_arm_controller demo.launch.py robot:=m20s_ccs hardware:=real type:=rg75
ros2 launch ocs2_arm_controller demo.launch.py robot:=m20s_ccs hardware:=real type:=ag2f90_d
ros2 launch ocs2_arm_controller demo.launch.py robot:=m20s_ccs hardware:=real type:=linkerhand_o7

# 臂体场总线 CAN
ros2 launch ocs2_arm_controller demo.launch.py robot:=m20s_ccs hardware:=real type:=rg75 \
  left_ee_drive_type:=tianji_can right_ee_drive_type:=tianji_can

# 外接串口 / SocketCAN（本包自包含，见 §2.1）
ros2 launch ocs2_arm_controller demo.launch.py robot:=m20s_ccs hardware:=real type:=linkerhand_o7 \
  left_ee_drive_type:=usb right_ee_drive_type:=usb
```

`robot.local.yaml` 片段：

```yaml
hardware:
  device_ip: 192.168.10.190
  arm_ctrl_mode: JOINT_IMPEDANCE
  max_joint_speed: 50
  max_joint_acceleration: 30
  usb_left_port: /dev/ttyUSB0
  usb_right_port: /dev/ttyUSB2
  # 阻抗增益（各 7 维；留空/注释时用 MarvinHardware 内置默认值）：
  # joint_k_gains: [2.0, 2.0, 2.0, 1.6, 1.0, 1.0, 1.0]
  # joint_d_gains: [0.4, 0.4, 0.4, 0.4, 0.4, 0.4, 0.4]
  # cart_k_gains:  [1800.0, 1800.0, 1800.0, 40.0, 40.0, 40.0, 20.0]
  # cart_d_gains:  [0.6, 0.6, 0.6, 0.4, 0.4, 0.4, 0.4]
  # left_dyn_param / right_dyn_param: 可选负载覆盖（空列表跳过）
  # left_ee_drive_type / right_ee_drive_type: marvin | tianji_can | usb | usb_can
```

> **方向映射**：`MarvinHardware` 要求 `arm_type ∈ {left, right, dual}`，本包由 `selected_arm_config` 直接传入（`type:=left` / `right` / `dual`）。

### 4.1 六维力传感器（KWR75）

`left_ft` / `right_ft` 取值 `none`（默认）/ `kwr75_485` / `kwr75_usb`：

- `kwr75_485` — 走 Marvin 臂体 COM2，`marvin_ft_sensor_interfaces` 在系统块内声明 `left_ft_sensor` / `right_ft_sensor`。
- `kwr75_usb` — 独立 `modbus_ros2_control/Kwr75ForceTorqueSensor` 传感器系统（`ft_sensor_systems.xacro`），导出同样的接口名。

`config/ros2_control/common.yaml` 中的 `left_ft_broadcaster` / `right_ft_broadcaster` 会自动检测传感器是否声明。

## 5. ros2_control 共享宏

`xacro/ros2_control/robot.xacro` 为薄封装。本包**不再 `exec_depend` 于 `m6_ccs_description`**：真机硬件、力传感器、末端接口与外接末端宏全部由本包提供。

| 文件 | 宏 | 说明 |
|------|-----|------|
| `interfaces.xacro` | `m20s_interfaces` | 单臂 7-DOF 关节接口 |
| `end_effector_interfaces.xacro` | `side_hand_interfaces`、`side_gripper_joint`、`dual_arm_ee_block`、`single_arm_ee_block` | 末端接口（真机夹爪 collapsed 单关节，仿真 full 链） |
| `xacro/eefs.xacro` | `ee_dyn_param` | `marvin_hardware_block` 所需负载参数 |
| `ros2_control/marvin_hardware.xacro` | `marvin_hardware_block`、`marvin_ft_sensor_interfaces` | Marvin 插件参数（含 `ee_dyn_param` 调用） |
| `ros2_control/ft_sensor_systems.xacro` | `ft_sensor_systems`、`side_ft_sensor_system` | KWR75 独立传感器系统 |
| `ros2_control/external_ee_systems.xacro` | `external_ee_systems`、`side_usb_hand_system`、`linkerhand_usb_can_side_system`、`changingtek_usb_side_system`、`jodell_usb_side_system` | 外接 `usb` / `usb_can`；**自包含**内联定义，不引用公共包的 `side_systems.xacro` |

### 5.1 include 顺序

`xacro/ros2_control/robot.xacro` 只需按顺序 include 本包文件，不存在同名宏互相覆盖的问题：

```xml
<xacro:include filename="$(find m20s_ccs_description)/xacro/ros2_control/marvin_hardware.xacro"/>
<xacro:include filename="$(find m20s_ccs_description)/xacro/ros2_control/external_ee_systems.xacro"/>
```

- `marvin_hardware.xacro` 会引入本包 `xacro/eefs.xacro`（提供 `ee_dyn_param`）并定义 Marvin 参数块。
- `external_ee_systems.xacro` 会引入本包 `end_effector_interfaces.xacro`（`dual_arm_ee_block` / `single_arm_ee_block` / `side_gripper_joint`）与 `ft_sensor_systems.xacro`，并自包含外接 side system。因此它可**无条件**引入，不必再像旧方案那样按驱动类型条件加载。

`external_ee_systems` 宏末尾**无条件**调用 `ft_sensor_systems`（仅 `kwr75_usb` 时生成独立传感器系统），故 `marvin` / `tianji_can`（臂体总线）与 `usb` / `usb_can`（外接）走同一路径。

### 5.2 外接末端与 `rw_rate`

`external_ee_systems.xacro` 内联定义并仅分派 §2.2 中具有外接路径的末端：

| 末端 | 驱动键 | 插件 |
|------|--------|------|
| `linkerhand_o6` / `linkerhand_l6` / `linkerhand_o7` | `usb_can` | `can_ros2_control/{O6,L6,O7}CanHardware` |
| `linkerhand_o6` / `linkerhand_l6` / `linkerhand_o7` | `usb` | `modbus_ros2_control/DexterousHandHardware` |
| `ag2f90_d` | `usb` | `modbus_ros2_control/ModbusHardware`（`gripper_type=changingtek`、`variant=90d`） |
| `rg75` | `usb` | `modbus_ros2_control/ModbusHardware`（`gripper_type=jodell`） |

`left_ee_rw_rate` / `right_ee_rw_rate` 为空时**不下发** `rw_rate` 属性，保留各硬件默认值（LinkerHand USB-CAN 默认 50）；非空时按左右分别写入对应 `<ros2_control>`。

仿真硬件插件见 [`robot_common_launch`](../../common/robot_common_launch/xacro/ros2_control/simulation_hardware.xacro)（`simulation_hardware_plugin`：mock / gz / isaac）。

URDF 层复用：`eef_type_resolver.xacro`、`eefs.xacro`。launch 入参（`device_ip`、`left_ee_drive_type` 等）仍在各机器人顶层 `robot.xacro` 声明，经宏 params 传入共享块。
