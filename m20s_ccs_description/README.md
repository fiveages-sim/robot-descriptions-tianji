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

> **依赖提示**：`usb` / `usb_can` 的外接 side system 由公共描述包（`linkerhand_description` / `jodell_description` / `changingtek_description` 的 `xacro/ros2_control/side_systems.xacro`）提供，与 M6-CCS 共用同一套宏，且仅在选用外接驱动时才解析。这些包需为包含 `side_systems.xacro` 的版本；仅 `marvin` / `tianji_can`（臂体总线）、mock / gz 均不依赖它们。

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

双臂默认按 M6-CCS 侧挂：`arm_base` 上左右臂 `xyz = (0, ±0.062, 0)`，`rpy = (∓π/2, 0, 0)`。安装位姿由 `m6_ccs_description` 的 `DualArmMount` 参数覆盖。

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

# 外接串口 / SocketCAN（需含 side_systems.xacro 的公共描述包，见 §2.1）
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

`xacro/ros2_control/robot.xacro` 为薄封装。本包已 `exec_depend` 于 `m6_ccs_description`，因此**真机硬件、力传感器、外接末端三部分直接 include M6 的宏文件**（M6 包无需任何改动）：

| 文件 | 宏 | 归属 |
|------|-----|------|
| `interfaces.xacro` | `m20s_interfaces` | 本包 |
| `end_effector_interfaces.xacro` | `side_hand_interfaces`、`side_gripper_joint`、`dual_arm_ee_block`、`single_arm_ee_block` | 本包（M20S 自有末端几何/接口，真机夹爪 collapsed 单关节，仿真 full 链） |
| `xacro/eefs.xacro` | `ee_dyn_param` | 本包（`marvin_hardware_block` 所需负载参数） |
| `m6_ccs_description: ros2_control/marvin_hardware.xacro` | `marvin_hardware_block`、`marvin_ft_sensor_interfaces` | 复用 M6 |
| `m6_ccs_description: ros2_control/ft_sensor_systems.xacro` | `ft_sensor_systems`、`side_ft_sensor_system` | 复用 M6（自带宏、无 include，可安全直接引入） |
| `m6_ccs_description: ros2_control/external_ee_systems.xacro` | `external_ee_systems` | 复用 M6（外接 `usb` / `usb_can`） |

### 5.1 会被覆盖的宏，以及处理方式

复用 M6 的文件会把 M6 的同名宏注册进同一个命名空间（后 include 者胜、无警告）。实际重名的共 **10 个**，分两组：

**A 组** —— 经 `marvin_hardware.xacro` 带进来的 M6 `xacro/eefs.xacro`（**无条件**引入）：

| 宏 | M6 vs M20 | 影响 |
|----|-----------|------|
| `EndEffectors`、`Ocs2SideEefFrame` | 内容相同 | 无 |
| `SideEndEffector`、`SideEndEffectorLinkerHand` | 内容不同（M6 按 `hand_type` 分派 erg32，M20 按 `gripper_type`） | 无——M20 `xacro/robot.xacro` 对它们的调用早于 M6 include，属时序保护 |
| `ee_dyn_param` | M6 支持 15 个末端 / M20 支持 11 个；**共同末端取值逐一比对全部相同** | 语义风险：若不夺回则用 M6 的参数表，M6 一旦调整即静默串味 |
| `SideEndEffectorERG32` | M6 独有 | 无（M20 不调用） |

**B 组** —— 经 `external_ee_systems.xacro` 带进来的 M6 `ros2_control/end_effector_interfaces.xacro`
（**仅 `usb` / `usb_can` 时**才被间接引入）：

`dual_arm_ee_block`、`single_arm_ee_block`、`side_gripper_joint`、`side_hand_interfaces`、
`side_gripper_interfaces_sim` —— 5 个**全部与 M20 重名且内容不同**。其中 M6 版
`side_gripper_joint` 的签名是 `params="side"`，**没有 `gripper_type` 形参**，被覆盖后 M20 的调用会直接报
`error: Invalid parameter "gripper_type"`。

### 5.2 处理方式

在两个 M6 include **之后**重新 include 本包自己的两个文件，把 M20 版本重新注册回去：

```xml
<xacro:include filename="$(find m6_ccs_description)/xacro/ros2_control/marvin_hardware.xacro"/>
<xacro:include filename="$(find m6_ccs_description)/xacro/ros2_control/ft_sensor_systems.xacro"/>
<xacro:if value="${has_external_ee}">
    <xacro:include filename="$(find m6_ccs_description)/xacro/ros2_control/external_ee_systems.xacro"/>
</xacro:if>
<!-- 必须在这之后：夺回 + 提供 -->
<xacro:include filename="$(find m20s_ccs_description)/xacro/eefs.xacro"/>
<xacro:include filename="$(find m20s_ccs_description)/xacro/ros2_control/end_effector_interfaces.xacro"/>
```

这两行的职责**并不相同**，因此都不能删、也不能上移：

- `xacro/eefs.xacro` —— 夺回 `ee_dyn_param` 等 A 组宏。另有个独立原因：单臂时
  `xacro/robot.xacro` 走非 dual 分支、不会引入 eefs，故此处必须显式 include。
- `ros2_control/end_effector_interfaces.xacro` —— **双重职责**：
  1. 非外接驱动（`marvin` / `tianji_can` / mock / gz）时，M6 的 `external_ee_systems` 根本没被引入，
     本行是 `dual_arm_ee_block` / `single_arm_ee_block` / `side_gripper_joint` 的**唯一来源**；
  2. 外接驱动时，它负责夺回被 B 组覆盖的定义。

> 已实测：删掉第 2 行在非外接模式下会报 `unknown macro name: xacro:dual_arm_ee_block`
> —— 因为 M6 的那份只在 `usb` / `usb_can` 时才会进命名空间。所以它不是"可选的夺回"，而是必需的提供者。
>
> 另外 `external_ee_systems` 宏本身也只在条件成立后才存在，其**调用点**必须带同一条件
> （`has_external_ee`）；臂体总线与 mock / gz 改由本包直接调用 M6 的 `ft_sensor_systems`，
> 与原先生效路径等价（M6 `external_ee_systems` 对臂体总线本就只做这一步）。

> **注意**：M6 当前另有两个既有问题（与本包无关，本包未触碰 M6）：
> - M6 的 `external_ee_systems.xacro` 无条件 include 上述缺失的 `side_systems.xacro`，导致
>   **M6 自身在 mock/gz/real 下均无法生成** `ros2_control`（`xacro` 直接报缺文件）。
> - `marvin_pro_description` 的 real 模式因未定义 `left_drive_type` 而失败（M6 的
>   `end_effector_interfaces.xacro` 需要它）。本包与 M6 均定义了该属性，不受影响。

仿真硬件插件见 [`robot_common_launch`](../../common/robot_common_launch/xacro/ros2_control/simulation_hardware.xacro)（`simulation_hardware_plugin`：mock / gz / isaac）。

URDF 层复用：`eef_type_resolver.xacro`、`eefs.xacro`。launch 入参（`device_ip`、`left_ee_drive_type` 等）仍在各机器人顶层 `robot.xacro` 声明，经宏 params 传入共享块。
