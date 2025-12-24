# Tianji Marvin M6-S CCS Description

This package contains the description files for Rokae AR5 Manipulator.

## 1. Build

```bash
cd ~/ros2_ws
colcon build --packages-up-to m6_ccs_description --symlink-install
```

## 2. Visualize the robot

### 2.1 Basic Arm Configuration

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
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=dual collider:=simple
  ```

### 2.2 Arm Configuration for scene

* USB HIL task with AG2F90-C Gripper
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=usb-90c
  ```
* Dual Arm on Desktop
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=dual_desktop
  ```
* Dual Arm on Desktop with AG2F90-C Gripper
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=dual_desktop_90c
  ```

### 2.3 Arm configuration for humannoid

* Dual Arm with ChangingTek AG2F90-C or AG2F90-D
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=dual_ag2f90_c collider:=simple
  ```
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=dual_ag2f90_d collider:=simple
  ```
* Dual Arm with Jodell RG75
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=m6_ccs type:=dual_rg75 collider:=simple
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
* Mock Component With End Effector
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs type:=dual_ag2f90_c
  ```
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs type:=dual_ag2f90_d
  ```
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs type:=dual_rg75
  ```
   ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs type:=rg75
  ```

### 3.3 USB HIL task

#### 3.3.1 Simulation

* Mock Component
  ```bash
  # Right Arm
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs type:=usb-90c
  ```

## 4. Real Robot Deploy

### 4.1 Build Hardware Interfaces

```bash
cd ~/ros2_ws
colcon build --packages-up-to marvin_ros2_control --symlink-install
```

### 4.2 Launch Control

#### 4.2.1 Launch without Gripper
* Left Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs hardware:=real
  ```
* Right Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs hardware:=real type:=right
  ```
* Dual Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs hardware:=real type:=dual
  ```

#### 4.2.2 With ChangingTek AG2F90-C Gripper
* Dual Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=m6_ccs hardware:=real type:=dual_ag2f90_c
  ```