# KUKA iiwa Visual Servoing and ArUco Tracking

A ROS 2 and Gazebo Sim project that adds a camera to the KUKA iiwa end-effector and uses visual measurements for camera positioning and target tracking.

[Technical report](Trenti_hw3.pdf) · [ArUco marker 201 reference](ros2_vision/data/Online%20ArUco%20markers%20generator.pdf)

## Implementation

- The simulation was extended with an end-effector camera and selectable sphere and ArUco worlds.
- A blue-sphere detector was implemented using HSV segmentation and OpenCV contours.
- The controller uses ArUco pose estimates for camera positioning and look-at-point control.
- The effort-controlled demonstration combines straight-line trajectory tracking with look-at-point control.

## Run the demos

Use ROS 2 Humble with Gazebo Sim, OpenCV, `cv_bridge`, and `rqt_image_view`. The repository includes the iiwa simulation stack and the ArUco ROS packages.

Clone the repository from the `src` directory of the ROS 2 workspace.

```bash
git clone https://github.com/FedericoTr26/RL_Homework-3.git
```

Return to the workspace root to build and source the workspace.

```bash
colcon build
source install/setup.bash
```

Source `install/setup.bash` from the workspace root in each new terminal.

### Blue-sphere detection

Start the sphere world in one terminal, then launch the detector and the image viewer in separate terminals.

```bash
ros2 launch iiwa_bringup iiwa.launch.py command_interface:="velocity" robot_controller:="velocity_controller" use_aruco:=false
```

```bash
ros2 run ros2_opencv ros2_opencv_node
```

```bash
ros2 run rqt_image_view rqt_image_view
```

Select `/processed_image` to see the contour overlaid on the image. The detector reads `/videocamera`.

### Position the camera relative to the marker

Start Gazebo, the ArUco detector, and the controller in separate terminals.

```bash
ros2 launch iiwa_bringup iiwa.launch.py command_interface:="velocity" robot_controller:="velocity_controller"
```

```bash
ros2 run aruco_ros single --ros-args -r /image:=/videocamera -r /camera_info:=/camera_info -p marker_id:=201 -p marker_size:=0.1 -p reference_frame:=world -p marker_frame:=aruco_marker_frame -p camera_frame:=camera_link_optical
```

```bash
ros2 run ros2_kdl_package ros2_kdl_vision_control
```

[Positioning video](VideoHW3/Positioning.mp4)

### Keep the marker centered

Use the same velocity launch. Start the detector with the camera optical frame as the reference, then start the look-at-point controller.

```bash
ros2 run aruco_ros single --ros-args -r /image:=/videocamera -r /camera_info:=/camera_info -p marker_id:=201 -p marker_size:=0.1 -p reference_frame:=camera_link_optical -p marker_frame:=aruco_marker_frame -p camera_frame:=camera_link_optical
```

```bash
ros2 run ros2_kdl_package ros2_kdl_vision_control --ros-args -p task:="look-at-point"
```

[Look-at-point video](VideoHW3/LookAtPoint.mp4)

### Track a straight trajectory while looking at the marker

Use the effort controller in Gazebo. Run the ArUco detector with the same optical-frame command from the look-at-point demo, then start the effort-controlled node.

```bash
ros2 launch iiwa_bringup iiwa.launch.py command_interface:="effort" robot_controller:="effort_controller"
```

```bash
ros2 run ros2_kdl_package ros2_kdl_vision_control --ros-args -p task:="look-at-point" -p cmd_interface:=effort
```

[Trajectory and visual tracking video](VideoHW3/Punto2b.mp4)

For any ArUco demo, use the image viewer and select `/videocamera`. The recordings use a WebM container with an `.mp4` filename.

## Results

The sphere detector overlays the contour in the simulated camera feed. In the marker demo, the camera re-centers the marker after it is moved. The effort demo shows the arm tracking a straight path while keeping the marker in view. The report also documents the camera roll observed when the marker is displaced laterally.

## Attribution

Developed as a group project for Robotics Lab. The project builds on the bundled [iiwa ROS 2 stack](ros2_iiwa/README.md) and [vision packages](ros2_vision/README.md).
