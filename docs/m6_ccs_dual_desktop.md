# Marvin M6-CCS Dual-Arm Desktop

Dual-arm **booth / demo stand** layout for m6_ccs. Use **`variant:=desktop`** (m6_ccs-specific platform variant), not `type:=desktop`.

End effectors are selected independently via `type` / `left_type` / `right_type` (see [m6_ccs_description/README.md](../m6_ccs_description/README.md)).

## 1. Build

```bash
cd ~/ros2_ws
colcon build --packages-up-to m6_ccs_description --symlink-install
```

## 2. Visualize

* Dual arm on desktop (no gripper)

```bash
source ~/ros2_ws/install/setup.bash
ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs variant:=desktop collider:=simple
```

* Desktop + AG2F90-C gripper

```bash
source ~/ros2_ws/install/setup.bash
ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs variant:=desktop type:=ag2f90_c collider:=simple
```

* Desktop + AG2F90-C (soft pad)

```bash
source ~/ros2_ws/install/setup.bash
ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs variant:=desktop type:=ag2f90_c_soft collider:=simple
```

## 3. OCS2 Arm Controller

### 3.1 Mock Components

* Desktop only

```bash
source ~/ros2_ws/install/setup.bash
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs variant:=desktop
```

* Desktop + AG2F90-C

```bash
source ~/ros2_ws/install/setup.bash
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs variant:=desktop type:=ag2f90_c
```

* Desktop + AG2F90-C (soft pad)

```bash
source ~/ros2_ws/install/setup.bash
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs variant:=desktop type:=ag2f90_c_soft
```

### 3.2 Isaac Sim

* Desktop only

```bash
source ~/ros2_ws/install/setup.bash
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs variant:=desktop hardware:=isaac
```

* Desktop + AG2F90-C

```bash
source ~/ros2_ws/install/setup.bash
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs variant:=desktop type:=ag2f90_c hardware:=isaac
```

* Desktop + AG2F90-C (soft pad)

```bash
source ~/ros2_ws/install/setup.bash
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs variant:=desktop type:=ag2f90_c_soft hardware:=isaac
```
