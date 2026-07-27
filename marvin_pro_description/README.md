# Marvin Pro Description

Tianji Marvin Pro：高立柱固定基座 + 双臂 M6-CCS + 末端（默认 Marvin 夹爪）。

![Marvin Pro](../.images/marvin_pro.png)

复用方式与 `fiveages_w2` 相同：

- 臂：`m6_ccs_description` 的 `M6_CCS` / `DualArmMount`
- 末端：`m6_ccs_description` 的 `eef_type_resolver` + `eefs`（含 `marvin_gripper` / `marvin_gripper45`）
- 底座与双臂安装位姿：本包 `components/base.xacro`

厂商汇总见 [Tianji README](../README.md)。

## Build

```bash
cd ~/ros2_ws
colcon build --packages-up-to marvin_pro_description --symlink-install
source install/setup.bash
```

## Visualize

```bash
# 默认：双臂 + 直板 Marvin 夹爪
ros2 launch robot_common_launch manipulator.launch.py robot:=marvin_pro collider:=simple

# 45° 法兰夹爪
ros2 launch robot_common_launch manipulator.launch.py robot:=marvin_pro type:=marvin_gripper45 collider:=simple

# 也可挂 CCS 其它末端，例如
ros2 launch robot_common_launch manipulator.launch.py robot:=marvin_pro type:=rg75 collider:=simple
ros2 launch robot_common_launch manipulator.launch.py robot:=marvin_pro \
  left_type:=marvin_gripper right_type:=rg75 collider:=simple
```

## Control / OCS2

```bash
ros2 launch ocs2_arm_controller demo.launch.py robot:=marvin_pro
ros2 launch robot_common_launch manipulator_ocs2.launch.py robot_name:=marvin_pro
```

## Type keys

`type` / `left_type` / `right_type` 与 `m6_ccs` 双臂末端键相同（见 `m6_ccs_description` README §2），默认 `marvin_gripper`。
