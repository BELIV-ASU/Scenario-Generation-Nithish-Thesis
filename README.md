# Scenario-Generation-Nithish-Thesis
This repository includes the steps involved in creating scenarios using CARLA Scenario Runner

## Package Installation
This repository assumes you have the following packages installed and built on your local computer. If not, install and build the packages using the links provided.  
1. [CARLA](https://carla.readthedocs.io/en/latest/start_quickstart/#b-package-installation) - Package version 0.9.15
   - This documentation provides steps to install CARLA simulator package version.
   - Note: For CARLA [client library](https://carla.readthedocs.io/en/latest/start_quickstart/#install-client-library) there are three options available, namely: .egg file, .whl file, **downloadable python package**. Downloadable package is straight forward and suggested.
2. [Scenario Runner](https://github.com/carla-simulator/scenario_runner) - Package version 0.9.15  
   - This documentation provides instructions to install scenario runner package which is used to create scenarios.
3. [Carla Autoware bridge](https://github.com/rohanNkhaire/carla_autoware_bridge)  
   - This repo provides a bridge between Carla simulator(0.9.15) and Autoware.Universe

## Scenario Creation and Sensor Data Collection
1. Open a terminal and move the directory where the CARLA package is installed and enter the following command to launch the CARLA simulator. This will launch and open the CARLA simulator with its default Town 10 map.  
```./CarlaUE4.sh```

2. Next launch the CARLA-ROS bridge by entering the following command. Note the CARLA-ROS bridge package is placed inside the carla_autoware workspace, and this workspace contains additional packages required for communication between CARLA and Autoware. However, for the purposes of creating scenarios and testing the proposed C-V2I framework the output from the packages other than ros-bridge are not utilized.
   ```
   ros2 launch carla_autoware_launch carla_autoware.launch.xml \
   objects_definition_file:=</your_object_definition_file.json>
   ```
   - The object definition file mentioned in the above command is not the same as the object configuration file used by the Scenario Runner package. This object definition file contains the pose and other configurations of various sensors attached to the ego and to the CARLA world. It also contains the model of the ego vehicle used and its initial pose in the CARLA world.
   - The object definition file used in this implementation is found [here]().

3. The carla-ros bridge converts all the sensor data from the CARLA simulator into ROS2 format which could be recorded using ros2 bag record command. However, the output rate of the sensors will vary depending on the computing power available on the system. The following command is used to downsample the rate of the infrastructure camera output to 5 FPS and the on-board LiDAR output rate to 10 FPS and thus fix the output rate similar to what we get from physical sensors.
   ```
   ros2 launch sensor_rate_downsampler sensor_rate_downsampler.launch.py
   ```
4. Now the scenario can be loaded using the following command. This will load the scenario in the CARLA and wait until the ego vehicle attains a particular
predefined speed. Once the ego vehicle crosses this threshold speed all the other vehicles in the scenario are triggered based on the provided sequence in the
scenario definition file. This command should be launched only after activating the conda environment which contains the required modules installed to run the
scenario runner.
   ```
   Python3 scenario_runner.py –scenario \ 
   CustomWaypointFollowerScenario2_1 --output
   ```
5. The following command launches the Carla waypoint publisher node which queries the ego vehicle’s current location from the CARLA simulator and provides the waypoints of the trajectory based on the given goal pose.
   ```
   ros2 launch carla_waypoint_publisher carla_waypoint_publisher.launch.py
   ```
6. Next vehicle control using waypoints node can be activated which subscribes to the waypoints published by the carla waypoint publisher node. This node controls the vehicle's velocity and steering and navigates the vehicle towards the goal pose.
   ```
   ros2 run vehicle_control_using_waypoints vehicle_control_using_waypoints
   ```
7. To start recording the required message topics the following command is used.The name of the ros2 bag can be changed as needed. These recorded messages are sufficient to test the proposed C-V2I framework.
   ```
   ros2 bag record -o "your ros2 bag name" \ 
   /carla/camera_infrastructure/rate_downsampled/image \
   /sensing/lidar/concatenated/rate_downsampled/pointcloud \
   /localization/kinematic_state /tf /tf_static \
   /perception/object_recognition/detection/carla/autoware_objects
   ```
8. Now that the ros2 bag recording has started, the ego vehicle can be navigated to the goal pose. As mentioned earlier, once the ego vehicle attains a threshold
velocity the other vehicles are triggered, and the scenario starts to work. The following command publishes the goal pose of the ego vehicle, which is
subscribed by the Carla waypoint publisher node, computes the waypoints and publishes the waypoints which is subscribed by the vehicle control using
waypoints node for controlling the ego vehicle. The goal pose will vary based on the scenarios.
   ```
   ros2 topic pub --once /carla/ego_vehicle/goal geometry_msgs/msg/PoseStamped
   "{header: {stamp: {sec: $(date +%s), nanosec: $(( $(date +%s%N) % 1000000000
   ))}, frame_id: 'my_frame'}, pose: {position: {x: -67.0, y: -28.18, z: 0.03},
   orientation: {x: 0.0, y: 0.0, z: 0.0, w: 1.0}}}"
   ```
   



