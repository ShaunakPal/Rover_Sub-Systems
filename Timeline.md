- 2023-9-06: Study ros2 humble, exploring behaviours due to QoS , docker, github :2023-9-08
- 2023-9-09: CAT-1 :2023-9-16
- 2023-9-16: Drive :2023-9-23
- 2023-9-24: Science :2023-9-30
- 2023-10-1: FPV+IP cam :2023-10-3
- 2023-10-4: Node Management | Gazebo setup and simulation:2023-10-8
- 2023-10-9: App Dev | Testing and Debugging :2023-10-22
- 2023-10-25: End
### Drive (2023/9/16 - 2023/9/23):
- Custom message building 
	- Message name: DrivePwm
		- topic: /Drive/motor_pwm
		- message per motor
			- speed (int)
			- direction (int)
	- Message name: DriveVel
		- topic: /Drive/cmd_vel
		- message
			- v (linear velocity) (double)
			- w (angular velocity) (double)
	- Message name: Imu.msg
		- topic: /Base/imu_raw
		- Location: sensor_msgs/Imu Message
	- Message name: Odometry.msg
		- topic: /Drive/odom
		- location: nav_msgs/Odometry.msg
	- Message name: NavSatFix.msg
		- topic: /Base/gps_raw
		- location: sensor_msgs/NavSatFix.msg

- joystick mapping(yaml file) and publisher node
- motor control node {array of 8 integers: speed and direction for each motor}
- orocos:
	- **KDL solvers** - to compute anything from forward position kinematics, to inverse dynamics
	- The Bayesian Filtering Library - (Extended) Kalman Filters, Particle Filters (or Sequential Monte Carlo methods), etc. These algorithms can, for example, be run on top of the Realtime Services, or be used for estimation in Kinematics & Dynamics applications.
	
	- **Motor Control Component**:
	    - Orocos RTT Component: `motor_control_component`
	    - ROS2 Node: `motor_control_node`
	    - Topics:
	        - `/motor_pwm` (Publish by `motor_control_node`)
	        - `/Drive/cmd_vel` (Subscribe by `motor_control_node`)
	
	- **Sensor Interface Components**:
	    - Orocos RTT Components: `camera_component`, `lidar_component`, `imu_component`, etc.
	    - ROS2 Nodes: `camera_node`, `lidar_node`, `imu_node`, etc.
	    - Topics:
	        - `/rover/camera_data`, `/rover/lidar_data`, `/rover/IMU_raw`, etc. (Published by corresponding ROS2 nodes)
	
	- **Control Logic Component**:
	    - Orocos RTT Component: `control_logic_component`
	    - ROS2 Node: `control_logic_node`
	    - Topics:
	        - `/rover/high_level_commands` (Publish by `control_logic_node`)
	        - `/rover/control_status` (Subscribe by `control_logic_node`)
- odometry publisher
- configure microros agent
- telemetry node
### Science (2023/9/24 - 2023/9/30):
- Custom message building
	- Message Name: ScienceOut.msg
		- topic: /Science/cmd_vel
		- message 
			- int32 drill_dir
			- int32 drill_speed 
			- int32 drill_distance
			- int32 auger_dir
			- int32 auger_speed
			- int32 carousel_dir
			- int32 carousel_speed
			- int32 reagent_direction
			- int32 reagent_speed
			- int32 npk_servo
			- int32 micro_dir
			- int32 micro_speed
	- Message Name: ScienceIn.msg
		- topic: /Science/science_data
		- message
			- float32 temp
			- float32 humidity
			- float32 pressure
			- char N
			- char P
			- char K
			- char soiltemp
			- char soilmoist
			- char soilph
			- float32 uva
			- float32 uvb
			- float32 uvindex
			- int32 gas_sensor

- joystick mapping(yaml file) and publisher node
- cmd_vel publishing node
- telemetry node
- Data Management and Visualisation
	- sensor subscription node(saves to .csv files)
	- matplotlib visualisation program (.csv to graphs)
### FPV+IP cam (2023/10/1 - 2023/10/3):
- Custom message building
	- Message name: FPVservo.msg
		- topic: /Base/`no.`/cmd_vel
		- message
			- int32 pan
			- int32 tilt
	- Message name: IPcam.msg
		- topic: /Base/IPout
		- message
			- int32 motor_dir
			- int32 motor_speed
- joystick mapping(yaml file) and publisher node
- PCA driver ros2_node
- telemetry node
- Live stream node
### Node Management (2023/10/4 -2023/10/8):
- Lifecycle/Managed Node - 
	- service client node - node with 2 service clients, one to change the state of lifecycle publisher, and one to ask for its current state.service client to request lifecycle node’s transition
	- callee script — script to use service client node and request for state transitions.
	- Implemented in nodes which require more control.


- Health Monitoring Node
	- topic: /diagnostics`
	- subscription: to every node
	- package - diagnostic_aggregator
	- msg:  [diagnostic_msgs](https://docs.ros2.org/galactic/api/diagnostic_msgs/index-msg.html)/msg/DiagnosticStatus`

- Fast DDS monitor - https://youtu.be/OYibnUnMIlc?feature=shared
- ros2 Tracing setup - TraceAlyzer

## APP dev:
- Using Electron.js as the main framework. 
- Exe file be created.
- To communicate with ROS-
	- rclnodejs exists
	- ros2 web bridge

