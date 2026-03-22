# Arch_ROS
=================================================================================================
                               ROS 2 <---> GAZEBO ARCHITECTURE FLOW
=================================================================================================

       [ROS 2 DDS NETWORK]                 [THE TRANSLATOR]                  [GAZEBO TRANSPORT]
                                                  |
+------------------------------+                  |                  +--------------------------+
|        USER / TELEOP         |                  |                  |    GAZEBO SIMULATION     |
|  (Keyboard/Joystick script)  |                  |                  |       (gazebo-1)         |
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
| to calculate 3D transforms   |                  |
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