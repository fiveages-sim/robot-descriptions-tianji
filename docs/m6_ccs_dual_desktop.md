# Marvin M6-CCS Dual-Arm Desktop

## 1. Build
```bash
cd ~/ros2_ws
colcon build --packages-up-to m6_ccs_description --symlink-install
```

## 2. Visualize

* Only Dual Arm

```bash
source ~/ros2_ws/install/setup.bash
ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=dual_desktop collider:=simple
```

* Dual Arm with AG2F90C Gripper

```bash
source ~/ros2_ws/install/setup.bash
ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=dual_desktop_90c collider:=simple
```

## 3. OCS2 Arm Controller
### 3.1 Mock Components

* Only Dual Arm

```bash
source ~/ros2_ws/install/setup.bash
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs type:=dual_desktop
```

* Dual Arm with AG2F90C Gripper

```bash
source ~/ros2_ws/install/setup.bash
ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs type:=dual_desktop_90c
```
