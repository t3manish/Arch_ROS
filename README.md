# Architecture: ROS 2 and Gazebo Simulation

This document outlines the system architecture and data flow for running the TurtleBot3 simulation using ROS 2 and Gazebo. 

Because ROS 2 and modern Gazebo use different communication protocols, the system relies on a bridge to pass data between the ROS 2 DDS network and the Gazebo transport layer.

## Architecture Flow Diagram

```text
=================================================================================================
                               ROS 2 <---> GAZEBO ARCHITECTURE FLOW
=================================================================================================

       [ROS 2 DDS NETWORK]                 [THE TRANSLATOR]                  [GAZEBO TRANSPORT]
                                                  |
+------------------------------+                  |                  +--------------------------+
|        USER / TELEOP         |                  |                  |    GAZEBO SIMULATION     |
|         (ROS 2 Node)         |                  |                  |       (gazebo-1)         |
+--------------+---------------+                  |                  |  (Physics & Environment) |
               |                                  |                  +-------------+------------+
               | 1. geometry_msgs/TwistStamped    |                                |
               v          (/cmd_vel)              |                                |
+------------------------------+        +---------v---------+        +-------------v------------+
|                              | -----> | ROS -> GZ Bridge  | -----> | 2. gz.msgs.Twist         |
|                              |        | (parameter_bridge)|        |    (/cmd_vel)            |
|                              |        +-------------------+        +--------------------------+
|                              |                  |                                |
|                              |                  |                  3. Physics Engine Calculates
|                              |                  |                     Movement & Sensors
|                              |                  |                                |
|                              |        +---------v---------+        +-------------v------------+
|                              | <----- | GZ -> ROS Bridge  | <----- | 4. GZ Native Messages:   |
|                              |        | (parameter_bridge)|        | - gz.msgs.Clock          |
|                              |        +-------------------+        | - gz.msgs.Model          |
|                              |                  |                  | - gz.msgs.Odometry       |
|                              |                  |                  | - gz.msgs.Pose_V (TF)    |
|                              |                  |                  | - gz.msgs.IMU            |
|                              |                  |                  | - gz.msgs.LaserScan      |
|     ROS 2 TOPIC NETWORK      |                  |                  +-------------+------------+
|                              |                  |                                |
| 5. ROS 2 Messages:           |                  |                                v
| - rosgraph_msgs/Clock        |                  |                  +--------------------------+
| - sensor_msgs/JointState     |                  |                  |        GAZEBO GUI        |
| - nav_msgs/Odometry          |                  |                  |       (gazebo-2)         |
| - tf2_msgs/TFMessage         |                  |                  |   (Visualizes Physics)   |
| - sensor_msgs/Imu            |                  |                  +--------------------------+
| - sensor_msgs/LaserScan      |                  |
|                              |                  |
+--------------+---------------+                  |
               |                                  |
               | 6. /joint_states                 |
               v                                  |
+------------------------------+                  |
|   ROBOT STATE PUBLISHER      |                  |
|  (robot_state_publisher-5)   |                  |
| Uses turtlebot3_burger.urdf  |                  |
| to publish /tf transforms    |                  |
+--------------+---------------+                  |
               |                                  |
               | 7. /tf (Transform Tree)          |
               v                                  |
+------------------------------+                  |
|            RVIZ 2            |                  |
|           (rviz2-1)          |                  |
| (Visualizes /scan, /odom,    |                  |
|  /tf, and URDF model)        |                  |
+------------------------------+                  |
                                                  |
=================================================================================================
```

## Step-by-Step Data Flow

The architecture operates in a continuous loop of command input and state feedback. 

1. **Command Publication:** A ROS 2 node (such as a teleop keyboard script) publishes a velocity command of type `geometry_msgs/msg/TwistStamped` to the `/cmd_vel` ROS 2 topic.
2. **Command Translation:** The `parameter_bridge` node subscribes to the `/cmd_vel` ROS 2 topic. It converts the ROS 2 message format into Gazebo's native format (`gz.msgs.Twist`) and sends it to the Gazebo simulation.
3. **Physics Execution:** The Gazebo server (`gazebo-1`) receives the `gz.msgs.Twist` command and applies the requested motion to the robot model within the physics engine.
4. **State and Sensor Output:** As the simulated robot moves, the Gazebo physics engine generates updated data for the robot's joints, position, and sensors (Lidar, IMU). Gazebo outputs this data using its native `gz.msgs` formats.
5. **State Translation:** The `parameter_bridge` receives the native Gazebo messages. It converts them back into standard ROS 2 messages (e.g., `sensor_msgs/msg/JointState`, `sensor_msgs/msg/LaserScan`) and publishes them to the ROS 2 DDS network.
6. **Kinematics Processing:** The `robot_state_publisher` node subscribes to the translated `/joint_states` topic. It uses these joint angles alongside the robot's URDF (Unified Robot Description Format) file to calculate the 3D position of every robot link, publishing the result to the `/tf` topic.
7. **Visualization:** The RViz 2 node subscribes to the ROS 2 topics (`/scan`, `/odom`, `/tf`) and renders a 3D visual representation of the sensor data and robot state.

## Core System Nodes

* **`gazebo-1` (Server):** The headless physics engine that computes collisions, dynamics, and sensor data.
* **`gazebo-2` (GUI):** The graphical client that renders the simulated environment.
* **`parameter_bridge`:** The required middleware node that translates topics and message types bidirectionally between ROS 2 and Gazebo.
* **`robot_state_publisher`:** A ROS 2 node that broadcasts the robot's kinematic tree based on the URDF and current joint states.
* **`rviz2`:** The standard ROS 2 visualization tool used to view published topic data.