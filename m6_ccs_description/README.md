# Tianji Marvin M6-S CCS Description

This package contains the description files for Rokae AR5 Manipulator.

## 1. Build

```bash
cd ~/ros2_ws
colcon build --packages-up-to m6_ccs_description --symlink-install
```

## 2. Visualize the robot

* Left Arm
    ```bash
    source ~/ros2_ws/install/setup.bash
    ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs
    ```
  ![Marvin M6 CCS](../.images/m6_ccs.png)
* Right Arm
    ```bash
    source ~/ros2_ws/install/setup.bash
    ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=right
    ```

* Dual Arm
    ```bash
    source ~/ros2_ws/install/setup.bash
    ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=dual
    ```


## 3. OCS2 Demo

### 3.1 Official OCS2 Mobile Manipulator Demo

* Left Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator_ocs2.launch.py robot_name:=m6_ccs
  ```
* Right Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator_ocs2.launch.py robot_name:=m6_ccs type:=right
  ```

### 3.2 OCS2 Arm Controller Demo

* Mock Components
  ```bash
  # Left Arm
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs 
  ```
    ```bash
  # Right Arm
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs type:=right
  ```
    ```bash
  # Dual Arm
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs type:=dual
  ```