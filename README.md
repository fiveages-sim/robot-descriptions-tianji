# Tianji Robots

天机（Tianji / Marvin）机械臂描述包：单臂 M6-SRS、双臂 M6-CCS / M6S Lite CCS / M20S-CCS，以及高立柱固定基座的 Marvin Pro。M6-CCS 的末端解析供 Marvin Pro 与 `fiveages_w2` 复用；M6S Lite 用于 Gento Luna。

| | |
|:---:|:---:|
| <img src=".images/m6_srs.png" alt="Marvin M6-SRS" height="240"> | <img src=".images/m6_ccs.png" alt="Marvin M6-CCS" height="240"> |
| **[M6-SRS](m6_srs_description/)** | **[M6-CCS](m6_ccs_description/)** |
| <img src=".images/m20s_ccs.png" alt="Marvin M20S-CCS" height="240"> | <img src=".images/marvin_pro.png" alt="Marvin Pro" height="240"> |
| **[M20S-CCS](m20s_ccs_description/)** | **[Marvin Pro](marvin_pro_description/)** |

## Packages

| Package | Launch `robot` | 形态 | 说明 |
|---------|----------------|------|------|
| [m6_srs_description](m6_srs_description/) | `m6_srs` | 单臂 | 左/右臂布局（`type:=left` / `right`） |
| [m6_ccs_description](m6_ccs_description/) | `m6_ccs` | 双臂平台 | 丰富末端；支持 `variant:=desktop` 展台；可接 Marvin 真机 |
| [m6s_lite_ccs_description](m6s_lite_ccs_description/) | `m6s_lite_ccs` | 双臂平台 | 更短行程 CCS（节长 0.264 m）；第一期仅 tcp；Gento Luna 使用 |
| [m20s_ccs_description](m20s_ccs_description/) | `m20s_ccs` | 双臂平台 | 更大行程 CCS；自有末端解析与挂载；可接 Marvin 真机 |
| [marvin_pro_description](marvin_pro_description/) | `marvin_pro` | 高立柱 + 双臂 | 复用 **M6-CCS** 臂与末端；默认 `marvin_gripper` |

相关夹爪包（M6-CCS / Marvin Pro）：[`marvin_gripper_description`](../../common/gripper/marvin_gripper_description/)（直板 / 45° 法兰）。

## Common Commands

### Build

```bash
cd ~/ros2_ws
colcon build --packages-up-to <package> --symlink-install
source ~/ros2_ws/install/setup.bash
```

将 `<package>` 替换为 `m6_srs_description`、`m6_ccs_description`、`m6s_lite_ccs_description`、`m20s_ccs_description` 或 `marvin_pro_description`。

### Visualize

```bash
ros2 launch robot_common_launch manipulator.launch.py robot:=<robot> collider:=simple
```

常用参数：

- `robot:=m6_srs` / `m6_ccs` / `m6s_lite_ccs` / `m20s_ccs` / `marvin_pro`
- `type:=<ee_key>`：对称末端（M6-CCS / M20S-CCS / Marvin Pro）
- `type:=left` / `right`：单臂布局（不要同时当末端键用）
- `left_type:=… right_type:=…`：左右不同末端（**不要**再传 `type:=`）
- `variant:=desktop`：仅 `m6_ccs` 展台，与 `type` 正交
- `collider:=simple`：规划用简化碰撞体（xacro 默认多为 `convex`）
- `component.launch.py robot:=marvin_pro type:=body`：仅高立柱底座；`xacro_isaac:=true` 藏头部相机网格（Isaac 导入）

末端键表：

- M6-CCS / Marvin Pro：[m6_ccs_description README §2](m6_ccs_description/README.md#2-末端执行器)（含真机驱动）
- M20S-CCS：[m20s_ccs_description README §2](m20s_ccs_description/README.md#2-末端执行器)（子集，含真机驱动）

### OCS2 / Control

各包普通 `task.info`、`desktop.info`、`single.info` 的手臂参数以 W2 的 `fixed_base.info` 为基准：预测时域 1.0 s，R scaling 为 0.05，每臂对角值为 `[0.5, 0.5, 0.5, 0.5, 0.1, 0.1, 0.1]`，末端过程权重为位置/姿态 `100/50`，终端权重为 `200/100`，关节软限速为 ±1.57 rad/s。M6-CCS 的 `task_tcp.info` 使用 `fixed_base_tcp.info` 参数：时域 0.4 s，R scaling 为 0.001，过程/终端权重分别为 `10/5`、`20/10`，J1–J4 软限速 ±2.07、J5–J7 ±2.57 rad/s。

单臂配置只含 7 个关节，双臂配置只含 14 个关节，保留各模型原有的基座和末端坐标系。单臂自碰撞保持关闭；双臂按本模型基座映射碰撞对。M6-CCS 保留已有的 6/7 轴耦合范围：单臂由读取器自动使用索引 5/6，双臂为 5/6 与 12/13；其余机型不新增该约束。配置不包含腰、头、底盘关节或全身模式约束。

```bash
ros2 launch robot_common_launch manipulator_ocs2.launch.py robot_name:=<robot>
ros2 launch ocs2_arm_controller demo.launch.py robot:=<robot>
```

默认硬件为 `mock_components`。M6-CCS / Marvin Pro / M20S-CCS 真机加 `hardware:=real`；末端驱动方式通过 `left_ee_drive_type` / `right_ee_drive_type` 指定（细节见 [m6_ccs_description README](m6_ccs_description/README.md) 与 [m20s_ccs_description README](m20s_ccs_description/README.md)）。

## Layout

```text
Tianji/
├── .images/
├── m6_srs_description/
├── m6_ccs_description/
├── m6s_lite_ccs_description/
├── m20s_ccs_description/
└── marvin_pro_description/
```
