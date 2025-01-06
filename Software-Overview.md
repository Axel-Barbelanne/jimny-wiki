
# Software Overview

This page explains the structure of the GitLab repository, and discusses the software used in the project

[[#Repository Overview]]
[[#Arduino Code]]
	[[#Sterfboard]]
	[[#Steerbok]]
	[[#AxelBrake]]
[[#Jetson Code]]
	[[#ROS Code]]
		[[#Scripts]]
		[[#cuberos]]
		[[#mpc_driving_controller]]
		[[#mode_switch]]
		[[#to_vehicle]]
		[[#data_capture]]



## Repository Overview

`arduino_sketches` /
	All Arduino code for the 3 modules, any other Arduino-related tools 

`documents` /
	Any documents like the diary, the project proposal, etc.

`jimny_ws` /
	ROS 2 workspace, includes almost all software ran on the Jetson
	`src` /
		Location of all ROS 2 packages

`parts` /
	`CAD` /
		All CAD files, including assemblies
	`PCBs` /
		All PCB-related files (KiCAD files)

`utils` /
	Any other useful code including startup and visualization scripts


## Arduino Code

The following subsections describe the Arduino code (`.ino`) of each module in a little more detail
#### Sterfboard

The Sterfboard
#### Steerbok

The Steerbok is mainly a PID controller controlling the steering, with feedback from the angular encoder.

Data is received via CAN from the SterfBoard (UART from the Jetson) and feedback and computation results are sent back to the SterfBoard via CAN.

#### AxelBrake

The AxelBrake is mainly a PID controller controlling the acceleration and braking, with feedback from the OrangeCube/GPS to get the current GPS velocity.

Data is received via CAN from the SterfBoard (UART from the Jetson) and feedback and computation results are sent back to the SterfBoard via CAN.

**Improvements**:

Whilst having significantly different characteristics, the braking and acceleration are currently on the same PID controller. We noticed this was an important issue at the end, since braking needs a much stronger signal than acceleration. There are two ways to fix this:
- Split the PID into two to have one for acceleration, one for braking with different PID values
- Post-process the PID output for acceleration/braking to take these different characteristics into account




## Jetson Code

#### Scripts

Several scripts are added into the `jimny/utils/`,  `jimny/jimny_ws/` or within specific ROS packages. These are either to configure parameters on the Jetson, or to accelerate repetitive tasks.

`jimny/utils/` includes scripts not directly related to ROS, like a startup script for pulling from git (including updating submodules), and a script for setting up the visualization via SSH.

`jimny/jimny_ws/` include scripts related to ROS, including a `clean.sh` script to clean the workspace and a `start.sh` script to source and build.

Important scripts in packages will be noted below.

#### ROS Code

Each package has  a `README.md` file in its GitLab repository, please check this out for more information on the package.
##### `to_vehicle`

This package takes care of the communication between the Jetson and the SterfBoard, passing on any necessary information like commands and emergency state and receiving feedback data from the modules.

Messages are currently sent via UART due to the lack of CAN transceiver. However, an implementation of the CAN messages has already been done.

A heartbeat is published to ensure a good connection, which will be checked on arrival by the SterfBoard. Along with the "H" (for heartbeat), a signal flag is sent which can be one of:
- 0: Disarmed
- 1: Armed, valid signal
- 2: Armed, invalid signal

This package is also home to the main launch file, `do_jimny.launch.py` which will run the following nodes:
- mpc_driving_controller controller.launch.py
- cuberos cuberos_node
- mode_switch switch
- to_vehicle to_vehicle

##### `cuberos`

The cuberos package provides the interface between the Jetson and the OrangeCube. All relevant information is collected via the mavlink library, and published in separate topics. These topics are the following:

- `/gps`: GPS data, after passing through EKF
- `/gps_raw`: pre-processed GPS data
- `/rc_inputs`: remote control input data
- `/imu_attitude`: IMU data
- `/heartbeat`: heartbeat from OrangeCube

##### `mpc_driving_controller`

Below: Screenshot showing in *black* the global trajectory to be followed, in *blue* the segment of the trajectory fed to the MPC to be tracked, in *green* the MPC prediction steps, in *red* the past and current vehicle states
![[mpc_path.png|400]]

Below: Screenshot showing the MPC simulated on a trajectory recorded during a game drive in Dinokeng game reserve
![[dk_mpc_path.png|400]]

This is currently the most complex package, and includes the ROS2 implementation of two MPC controllers for a vehicle based on a bicycle model.

Most of the information regarding this package can be found in the README of its repository, particularly for the setup and configuration. Also note that the code is based on Vijay Govindarajan's project: 
https://github.com/MPC-Berkeley/genesis_path_follower?tab=MIT-1-ov-file.

Nevertheless, a brief overview will be presented here.

Most importantly, the package consists of two operation modes: simulation and real-world. 
The package consists of five separate nodes:

- `controller_node`:
	Node implementing the control loop, and generating the control commands
- `state_pub_node`:
	Node publishing the vehicle state from GPS, IMU and steering angle data
- `waypoint_generator_node`:
	Node providing a service to generate waypoints for the vehicle to follow, from a GPS trajectory. The vehicle already takes into account the vehicle's target velocity, path cruvature and MPC horizon. 
- `traj_plotter_node`:
	Node plotting the global GPS trajectory and the vehicle's path tracking behaviour
- `vehicle_simulator_node`:
	Node simulating the vehicle dynamics. The simulator uses a linear tire model, more precise than the MPC's

The lower-level code implementing the MPC class as well as the GPS Reference Trajectory class can be found in the `mpc_driving_controller/` directory.

The repository also stores the definitions for the custom messages and the custom service in the `msg/` and `srv/` directories respectively. More can be read on these in the README file.

There are also several useful scripts provided in the `utils/` directory. These should be called from the `jimny_ws` directory. These include:
- `update_py_env.sh`
	Script to easily update all the Shebang lines in the python files (since this is a C++ package for custom msg/srv), if running this from a new device with virtual python environments
	Simply call it from the `jimny_ws` directory after entering your chosen environment
- `change_waypoint_file.sh`:
	Script facilitating the change in waypoint file, for both simulation and real-world modes. Simply call with as argument the name of the file
- `generate_waypoints.py`
	Generator script for very simple test trajectories for debugging
- `process_guru.py`
	Conversion from the Guru Maps app CSV format, which we  used for recording larger trajectories


All the most important parameters that should be tuned have been relocated to the launch files in the `launch` directory for simplicity. There are a few parameters that could be tuned if needed outside of this, see the end of the README for more information on this.

The first functional MPC controller was based on cartesian coordinates, and is the one we used for most of the testing. Nevertheless, with the help of Clovis Schmitt, we worked on developing a second MPC based on Frenet coordinates. There are numerous reasons for which Frenet coordinates can be useful for this, but mainly it serves as a more reliable mechanism to make sure we stay on the desired road. This is implemented as a boolean parameter in the launch file. You may also notice other Frenet parameters that are not actually used in the Frenet MPC. These are remnants from an older non-functional Frenet MPC, but whose parameters could be useful in implementation down the line.

A final feature we will discuss here is the velocity tracking we implemented. Another parameter named `TRACK_VELOCITY` in the launch file ensures the MPC will track the velocity recorded in the waypoints (provided it is actually recorded of course). This allows the MPC to have a time-varying tracking velocity, for instance to go faster on larger roads and slower on narrower ones. This is accompaned by the `TRACK_SCALE` parameter, which will make the MPC track the pre-recorded velocity at a fixed ratio (for instance at half the velocity if 0.5 is set) 

##### `mode_switch`

The mode_switch package contains a node for safe mode switching for the system. Depending on this mode, mode_switch also publishes the Ackermann commands containing desired states from the RC input or the MPC output.

The node also keeps track of if there have been any errors with the signal, like for instance a lost connection with the cube, or a failed MPC computation. This is then attributed to a `signal_error` variable which will be reflected in the heartbeat code described in [[#`to_vehicle`|to_vehicle]].

##### `data_capture`

This package was made to record data, including:
- Left and right ZED camera images
- Vehicle states
	- velocity
	- yaw rate
	- steering angle
	- pose
- Vehicle GPS position

More information can be found in the README.md file in the repository


