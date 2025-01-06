

This page contains the information to get the Jimny running and to produce our obtained results. If the required hardware and software is already mounted and working, please skip straight to [Start-up](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Getting-Started#start-up). If none of the hardware is mounted, please go through the relevant sections in [Technical Documentation](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation).


Contents:

[Pre-Requisites](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Getting-Started#pre-requisites)

[Software Setup](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Getting-Started#software-setup)

  [Cloning the repository](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Getting-Started#cloning-the-repository)

  [Setting up the SSH connection with visualization](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Getting-Started#setting-up-the-ssh-connection-with-visualization)

[Start-up](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Getting-Started#start-up)

[Shut-Down](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Getting-Started#shut-down)

[Modes](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Getting-Started#modes)

[Commands and Controls](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Getting-Started#commands-and-controls)

  [HereLink Controller](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Getting-Started#herelink-controller)

  [Buttons](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Getting-Started#buttons)

  [Pedals](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Getting-Started#pedals)

<br>

---

## Pre-Requisites

**General pre-requisites**
Laptop with:
- Arduino IDE
- Git
- VS Code along with the following extensions:
	- Remote Development pack
	- Python
	- C/C++
	- ROS

**Useful**:
- CAD software (for analyzing/modifying any of the CAD files, we used Fusion 360/SolidWorks)
- PCB Design (for analyzing/modifying any PCB's, we used KiCAD)
- XLaunch/VcXsrv, or other X server for displays through SSH


The Jetson AGX Orin with the Jimny kit is already equipped with all the necessary software for running any of the Jetson-related software provided. The following is merely in case another Jetson should be used, or if any of the code needs to be ran on a laptop.

- Ubuntu 22.04 (Is ran on Jetpack 6.0)
- ROS 2 Humble
- ROS dependencies
	- See [Software Setup](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Getting-Started#software-setup)
- TMUX
- python3-rosdep

## Software Setup

#### Cloning the repository

Navigate to desired location and clone the repository:

```
cd ~/workspaces
git clone --recurse-submodules git@gitlab.com:tut-robotics/spots/agv/jimny.git
```

Install ROS dependencies

```
sudo rosdep init 
rosdep update
```


#### Setting up the SSH connection with visualization

Assuming you are connecting over ethernet, since wi-fi has the disadvantages of being limited in distance and is much slower.

The Jetson should already be on the 192.168.1.x network at address 192.168.1.123. Your laptop however, even after connecting the ethernet cable may not have this connection. Therefore, after connecting the cable, enter the following on the laptop:

First check the ethernet interface name with `ip a`; it should be something like `eth0` or `eno1`
Next add the static IP and bring it up (these are linux commands for other OS please check online):

```
sudo ip addr add 192.168.1.<chosen address>/24 dev <interface name>
sudo ip link set <interface name> up
```

Select the chosen address you want, between 0-255 and not 123

Now try to ping the Jetson from a terminal after powering it on:
`ping 192.168.1.123`

If this doesn't work, you may need to power on the Jetson with a monitor, keyboard and mouse. The password is available on the Jimny Documentation Google docs document on the drive . You should then follow the instructions above to add the static IP of 192.168.1.123.

The connection should then work, which you can test by pinging again and entering by ssh:
`ssh liljim@192.168.1.123`, followed by the password.

Open VSCode, Ctrl + Shift + P, "Remote-SSH: Open SSH configuration file...", "...\.ssh\.config"

Now paste the following at the end 

```
Host jimny-eth
	HostName 192.168.1.123
	User liljim
	ForwardX11 yes
	ForwardX11Trusted yes
	ForwardAgent yes
	Compression yes
	Protocol 2
	ServerAliveInterval 60
	ServerAliveCountMax 3
	XAuthLocation /usr/bin/xauth
```

Most of these after ForwardAgent are probably not necessary, but this works

Update the visualization script: 
Open VSCode, Ctrl + Shift + P,  "enter Remote-SSH: Connect to Host...", select "jimny-eth"
You should now enter the Jetson via SSH after entering the password.
Update the ~/workspaces/jimny/utils/remote_display.sh with the text editor you like/with VS Code
- Simply update the address with the one of your laptop you chose above


You can now test the SSH connection + visualization as follows:

Run XLaunch on the laptop
Open VSCode, Ctrl + Shift + P,  "enter Remote-SSH: Connect to Host...", select "jimny-eth", open a terminal

```
source ~/workspaces/jimny/utils/remote_display.sh
xclock
```

Now after a few seconds a clock should appear on your laptop (which is run on the Jetson)


---

## Start-up

The start-up consists of 5 steps:

First check the emergency button is not pushed in and the cable connections are secure
   
1. Power on the Jetson
   
	Connect the USB-C cable below to a power source. We were typically using a power bank, though an outlet on an inverter with the battery would work just fine too.
   

<img src="https://github.com/Axel-Barbelanne/jimny-wiki/blob/main/images/jetson_power.svg" width="600">

2. Connect the battery
   
	Connect the crocodile to XT30 connector cable on the battery side (positive crocodile connector first). 
	Connect the XT30 connectors for the battery.

        <img src="https://github.com/Axel-Barbelanne/jimny-wiki/blob/main/images/battery_plug.svg" width="600">
   
3. Turn on the HereLink controller

	Long press the power button, then select the QGroundControl app.

	A green progress bar should start loading at the top. If not, double check the HereLink is on by checking its fan (above you if you are seated in the front).

4. Run the software

	Run XLaunch on the host computer for visualization (default settings are fine)
	
	Connect to SSH via ethernet (or wifi if performing static tests):
	Open VS Code
	Ctrl + Shift + P, "enter Remote-SSH: Connect to Host...", select the Jimny (jimny-eth)
	``
	```
	cd ~/workspaces/jimny/jimny_ws               # Navigate to the directory
	source ../utils/remote_display.sh            # Set up visualization
	source start.sh                              # Build and source the workspace
	tmux                                         # Start a TMUX terminal
	ros2 launch to_vehicle do_jimny.launch.py    # Launch the ROS launch file
	```
	
	Detach from the TMUX terminal by pressing Ctrl + B, then D. You can later re-attach to the terminal if you need with:
	`tmux attach -t <terminal number, starts at 0>`

5. Turn on the car engine with the key


---

## Shut-Down

Similar to the start-up in reverse order:

1. Turn off the car engine with the key
   
2. Stop the ROS launch file:

	`tmux attach -t <terminal number, starts at 0>`
	Note that you can enter `tmux ls` to see the active terminal numbers
	
	You can then press Ctrl + C to stop the running file
	Finally enter `exit` to quit the TMUX terminal


Note: You can leave the Jimny idle as it is now for a short while between tests to avoid having to restart everything every time. 

3. Turn off the HereLink controller

	Long press the power button

4. Unplug the battery with the XT30 connectors
   
5. Unplug the Jetson with the USB-C cable
   

---

## Modes


The Jimny consists of 3 ( + emergency) modes. These are Manual, Remote and Autonomous.

Note that the HereLink modes have conflicting name so we will always be writing them in capitals to distinguish them (e.g. MANUAL, ACRO, etc.)

1. Manual: 
	User can control the vehicle safely with the pedals and steering wheel
	
	- Stepper motor gets no power, allowing the user to steer
	- There is no acceleration
	- There is no braking

2. Remote:
	User can control the vehicle via the HereLink controller.
	
	-  Stepper gets power via the 48V input
	-  Joysticks direct desired velocity and steering angle

3. Autonomous
	The car is controlled autonomously
	
	-  Stepper gets power via the 48V input
	-  MPC directs desired velocity and steering angle

-  Emergency mode
	The car enters a safety mode to ensure it stops 
	
	-  Stepper motor gets no power, allowing the user to steer
	-  There is no acceleration
	-  There is 100% braking applied to bring the car to a stop 
	  


---

## Commands and Controls


The commands on the Jimny will mainly come from three places: the HereLink controller, the buttons and the pedals.

#### HereLink Controller


The controller is the main source of commands at this stage.

After the start-up procedure described [above](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Getting-Started#start-up), you will notice two toggle menus at the top-right: arming status and HereLink mode.

The arming status (red on image below) can be in *Armed* or *Disarmed* states. If on Disarm, the corresponding [Modes](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Getting-Started#modes) is Manual; if on Armed, it corresponds to the HereLink mode

The HereLink mode can be one of many, but only three modes are supported: *MANUAL*, *ACRO*, *LOITER*. Again, be careful with these names since they conflict with the ones [above](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Getting-Started#modes). This is for simplicity since other HereLink modes sometimes didn't respond, but should be fixed down the line.
- *MANUAL*: Corresponds to the Remote mode
- *ACRO*: Corresponds to the Autonomous mode
- *LOITER*: Corresponds to a spare mode which can be used for excitation commands in testing (eg. for system ID)

<img src="https://github.com/Axel-Barbelanne/jimny-wiki/blob/main/images/herelink_screen.svg" width="500">

As a shortcut for the arming/disarming of the vehicle, the user can press the far right button on the HereLink controller (see red arrow on image below).

Additionally, the A, B, C and D buttons can be configured for an easy mode change in the settings option. For now A is set to *MANUAL*, B to *ACRO* and C to *LOITER* (perhaps B and C are inverted).

Finally, if in *MANUAL* HereLink mode, i.e. in remote mode, you can use the joysticks:
- Left joystick:  steering angle
- Right joystick: desired velocity (forward = faster)


<img src="https://github.com/Axel-Barbelanne/jimny-wiki/blob/main/images/controller.svg" width="400">

#### Buttons

The buttons provide several controls available to the user from within the car, without the remote control.

First is the emergency mode, which will immediately enter the emergency mode described [above](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Getting-Started#modes). Make sure this button is not pushed in when starting anything, else the Jimny will be stuck in emergency mode.

<img src="https://github.com/Axel-Barbelanne/jimny-wiki/blob/main/images/emergency.jpg" width="200">

Additionally, three other buttons are available to the user. Only the left is currently used, which serves to manually force manual mode (i.e. user control) on the vehicle. Other buttons can be configured at a later stage by adapting the Sterfboard's Arduino code.

<img src="https://github.com/Axel-Barbelanne/jimny-wiki/blob/main/images/buttons.jpg" width="400">

#### Pedals

Finally, in operation the gas pedal and brake pedal serve as inputs too. In manual mode, these pedals act as one would expect, accelerating and braking the car. In any other mode, any press on the brake pedal will immediately override an accelerating signal that is sent as a safety measure.