# Tianji Marvin M6S Lite CCS Description

更短行程的 Marvin CCS 双臂（节长 0.264 m，力矩上限低于 M6-S）。运动学来自 Gento Luna CAD。
模型仅在 **`xacro/`**。挂载复用 `m6_ccs_description` 的 `DualArmMount`。第一期仅 `tcp`，不含末端执行器。

```bash
cd ~/ros2_ws
colcon build --packages-up-to m6s_lite_ccs_description --symlink-install
source install/setup.bash
```

## Visualize

```bash
ros2 launch robot_common_launch manipulator.launch.py robot:=m6s_lite_ccs collider:=simple
```

- `type:=left` / `type:=right`：单臂
- `collider:=simple` / `convex`

## OCS2 / Control

```bash
ros2 launch robot_common_launch manipulator_ocs2.launch.py robot_name:=m6s_lite_ccs
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6s_lite_ccs
```

默认硬件 `mock_components`。
