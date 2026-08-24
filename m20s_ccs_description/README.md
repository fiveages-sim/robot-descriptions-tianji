# Tianji Marvin M20S-CCS Description

Marvin M20S CCS 双臂 URDF / ros2_control 描述包。模型在 **`xacro/`**。OCS2 规划 URDF 由 `xacro/robot.xacro` 生成。末端类型与挂载由本包 `eef_type_resolver.xacro` / `eefs.xacro` 定义。当前仅仿真 / mock，无 Marvin 真机驱动。

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
| Mock 控制 | `demo.launch.py …`（默认 `hardware:=mock_components`） |

![Marvin M20S CCS](../.images/m20s_ccs.png)

**不对称末端示例**

```bash
ros2 launch robot_common_launch manipulator.launch.py robot:=m20s_ccs \
  left_type:=rg75 right_type:=linkerhand_o7 collider:=simple
```

`type:=left` / `right` 只选单臂拓扑，不是末端键。

## 2. 末端执行器

类型映射：`xacro/eef_type_resolver.xacro`。几何挂载：`xacro/eefs.xacro`。当前仅仿真 / mock。

| Type key | 产品 |
|----------|------|
| `rg75` / `rg75_heavy` | Jodell RG75 |
| `erg32` | Jodell ERG32 |
| `ag2f90`, `ag2f90_d` | ChangingTek AG2F90-D |
| `ag2f90_c`, `ag2f90_c_soft` | ChangingTek AG2F90-C |
| `ag2f120s`, `ag2f120s_d` | ChangingTek AG2F120S |
| `linkerhand_o6` / `linkerhand_l6` / `linkerhand_o7` | LinkerHand |

## 3. 平台

双臂默认按 M6-CCS 侧挂：`arm_base` 上左右臂 `xyz = (0, ±0.062, 0)`，`rpy = (∓π/2, 0, 0)`。安装位姿由 `m6_ccs_description` 的 `DualArmMount` 参数覆盖。

`collider:=simple` 时各连杆用包围盒拟合的圆柱碰撞体；`convex` 用 glb mesh。OCS2 `task.info` 已开启双臂自碰撞（`arm_base` 与左右 `link3/5/7`）。
