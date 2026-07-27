# Tianji Robots

天玑（Tianji / Marvin）机械臂描述包：单臂 M6-SRS、双臂 M6-CCS，以及高立柱固定基座的 Marvin Pro。末端执行器由 `m6_ccs_description` 统一解析，亦可被其它机型（如 `fiveages_w2`）复用。

| | | |
|:---:|:---:|:---:|
| <img src=".images/m6_srs.png" alt="Marvin M6-SRS" height="240"> | <img src=".images/m6_ccs.png" alt="Marvin M6-CCS" height="240"> | <img src=".images/marvin_pro.png" alt="Marvin Pro" height="240"> |
| **[M6-SRS](m6_srs_description/)** | **[M6-CCS](m6_ccs_description/)** | **[Marvin Pro](marvin_pro_description/)** |

## Packages

| Package | Launch `robot` | 形态 | 说明 |
|---------|----------------|------|------|
| [m6_srs_description](m6_srs_description/) | `m6_srs` | 单臂 | 左/右臂布局（`type:=left` / `right`） |
| [m6_ccs_description](m6_ccs_description/) | `m6_ccs` | 双臂平台 | 丰富末端；支持 `variant:=desktop` 展台 |
| [marvin_pro_description](marvin_pro_description/) | `marvin_pro` | 高立柱 + 双臂 | 复用 CCS 臂与末端；默认 `marvin_gripper` |

相关夹爪包：[`marvin_gripper_description`](../../common/gripper/marvin_gripper_description/)（直板 / 45° 法兰）。

## Common Commands

### Build

```bash
cd ~/ros2_ws
colcon build --packages-up-to <package> --symlink-install
source ~/ros2_ws/install/setup.bash
```

将 `<package>` 替换为 `m6_srs_description`、`m6_ccs_description` 或 `marvin_pro_description`。

### Visualize

```bash
ros2 launch robot_common_launch manipulator.launch.py robot:=<robot> collider:=simple
```

常用参数：

- `robot:=m6_srs` / `m6_ccs` / `marvin_pro`
- `type:=<ee_key>`：对称末端（CCS / Pro）；SRS 用 `left` / `right` 选侧
- `left_type:=… right_type:=…`：左右不同末端（**不要**再传 `type:=`）
- `collider:=simple`：规划用简化碰撞体（xacro 默认多为 `convex`）

末端键表与真机驱动方式见 [m6_ccs_description README §2](m6_ccs_description/README.md#2-末端执行器)。

### OCS2 / Control

```bash
ros2 launch robot_common_launch manipulator_ocs2.launch.py robot_name:=<robot>
ros2 launch ocs2_arm_controller demo.launch.py robot:=<robot>
```

默认硬件为 `mock_components`；真机加 `hardware:=real` 或 `real_usb`（细节见各包 README）。

## Layout

```text
Tianji/
├── .images/
├── m6_srs_description/
├── m6_ccs_description/
└── marvin_pro_description/
```
