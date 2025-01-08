

This page gives a brief overview of the Jimny project, along with its current state, and an overview of the underlying architecture.

Contents: <br>
[Project Purpose](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Jimny-Project-Overview#project-purpose) <br>
[Project Features](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Jimny-Project-Overview#project-features) <br>
[Current State and Future Objectives](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Jimny-Project-Overview#current-state-and-future-objectives) <br>
[Architecture Overview](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Jimny-Project-Overview#architecture-overview) <br>

<br>
<br>

<img src="https://github.com/Axel-Barbelanne/jimny-wiki/blob/main/images/matt_axel_jimny.JPG" alt="" width="600">

<br>
<br>

--- 

## Project Purpose

The SPOTS Jimny project is a mission that stands to significantly increase fence patrol efficiency across game reserves.

By converting a Suzuki Jimny into an autonomous ground vehicle (AGV), SPOTS seeks to facilitate the checking of the often tremendously long fence perimeters, which needs to be done over and over again. The Jimny also allows these patrols to be done regardless of the weather, as currently these generally need to be interrupted in case of rain or storm. In addition to that, the AGV would minimize the risks of putting rangers in dangerous situations by themselves along the fenceline, as well as lower fuel costs through driving efficiency.

Many additional advantages and features can be thought of down the line for the Jimny. For more information on the Jimny project's role for conservation, check out the Dinokeng magazine article [here](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Appendix#other-links).


---

## Project Features

The project features several useful features to allow the Jimny to operate as desired.

- GPS with RTK enhancement - To allow for centimeter accuracy 
- ZED Stereo Camera - Relay image
- AI Detection of obstacle avoidance
- 3 Variable modes:
	- Manual
	- Remote
	- Autonomous
- Emergency mode for safety


----

## Current State and Future Objectives

Below is an idea of the major stages of the project:

1.  Statically functioning hardware
	1. All modules working with the Jimny on lifts 
2.  Remote control functioning with hardware
	1. Jimny controllable from the HereLink controller
3.  Autonomous mode functioning in simulation
	1. Using pre-recorded trajectory
4.  Autonomous mode functioning with hardware
	1. Autonomous driving functions without human interference on small test trajectories
5.  Autonomous mode functioning safely on unobstructed roads
	1. Telemetry functioning over a sufficiently large range
6.  Object detection and response functioning onboard
	1. Autonomous driving functioning without human interference on roads
7.  Functional Global Planner
	1. Global planner algorithm to decide path to drive

Currently, the project is nearing stage 4, with some key points that should be improved in prior stages.

Importantly, all modules were thoroughly tested with the Jimny on lifts, along with all emergency responses and user controls. Nevertheless, several components are prototyped in PLA and should be exchanged for more durable materials. More information on this can be found in [Improvements and Suggestions](Improvements-and-Suggestions). 

The remote control interface was also tested, yet less extensively due to time constraints. Several test drives were done in the garden, which worked well except for one important drawback: the braking servo was found to be strong enough in practice. Therefore, for now the user needs to assist with the braking pedal to slow down the vehicle, which is necessary as the vehicle moves at a non-zero velocity by default (creep). Besides this, acceleration, braking and steering led to the expected changes in the vehicle's hardware.

Several versions of the autonomous driving algorithm were made and tested in simulation. Each was a modified version of an MPC controller, including one with cartesian coordinates, Frenet coordinates, and obstacle avoidance. The simulation was implemented using a more accurate model than what the MPC used (see [Software Overview](Software-Overview)). The autonomous driving was tested on many simple paths, from lines to circles to curves. Later, tests evolved to recordings of game drives in Dinokeng Game Reserve, which after careful tuning we were able to follow reliably in the general case.

Due to the braking issue discussed above, we never reliably ran the autonomous driving over trajectories in the garden without us assisting with the braking. We were however able to make the system work onboard, providing a plot of the planned trajectory, steering the wheel and accelerating the vehicle. There are hence several tweaks to be made, along with extensive testing and most likely tuning of the autonomous driving algorithms and module code before stage four can be considered completed.

Down the line, there are still several stages to be reached before the vehicle can really be called autonomous.

Stage five consists of the adaptation to the autonomous driving to real roads of the game reserve. Though everything was designed with these roads in mind, testing in these conditions will probably reveal modifications to be made and parameters to be tuned. Modifications may include for instance increasing the reliance on the camera rather than GPS waypoints in autonomous driving for improved reliability and adaptability. Along with this, stage five will also require improvements in telemetry in order to ensure reliable long-distance operation.

Stage six implements object detection and avoidance in practice. A lot of work was already done on this by Alexandre Clin Deffarges, a past intern at SPOTS. His work included a fine-tuned YOLO model to detect obstacles and classify them as human, animal, vehicle or object. Part of his work, along with Clovis Schmitt, was also in implementing this in the MPC algorithm. There is still much testing to be done in simulation, then onboard before validating the obstacle detection.

Stage seven, the final stage *for now* consists of the implementation of a global planner, most likely registering paths between nodes. This would allow for the vehicle to decide on its own paths to navigate, provided a destination, from its current location.


---


## Architecture Overview


The following diagram showcases the main nodes and communication systems in the project hardware. 


<img src="https://github.com/Axel-Barbelanne/jimny-wiki/blob/main/images/jimny_arch_light.svg" alt="" width="600">

<br>
<br>


Below is a very brief description of each node in the system:

**Buttons and Controls**
Buttons: Easy to reach buttons for emergency, mode switch, or other purposes
HereLink Controller: Controls for remote control mode, including mode switch, arming of vehicle, acceleration, braking, steering

CubePilot: Interfaces with the HereLink controller and the GPS, providing the data over USB to the Jetson

Jetson: Central brain of the system, runs most of the project's software, including autonomous driving algorithm, object detection, signal generation

**Modules**
Sterfboard: Controls the power supplied to the other modules, serves as the interface between the Jetson and the other modules
Steerbok: Controls the steering with a PID, based on feedback from a steering sensor and a target signal 
Axelbrake: Controls the acceleration and braking signals

**Interfacing hardware**
Stepper motor: Rotates the wheel based on the Steerbok's signal
Angle sensor: Sends the actual steering value to the Steerbok
Servo: Pulls the brake based on the braking signal 