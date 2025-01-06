
# Technical Documentation

This page gives an in-depth understanding of all important components of the Jimny. Software related information will be discussed in [[Software-Overview|Software Overview]].

Contents:
[Modules](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#modules)

  - [Sterfboard](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#sterfboard)

  - [AxelBrake](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#axelbrake)

  - [Steerbok](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#steerbok)


[Jetson](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#jetson)


[OrangeCube](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#orangecube)


[Sensors](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#sensors)

  - [HereLink](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#herelink)

  - [GPS](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#gps)

  - [Camera](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#camera)


[Output Components](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#output-components)

  - [Stepper](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#stepper)

  - [Boost Converter](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#boost-converter)

  - [Stepper Driver](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#stepper-driver)

  - [Angular Encoder](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#angular-encoder)

  - [Acceleration plug](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#acceleration-plug)

  - [Braking servo](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#braking-servo)


[Additional Components](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#additional-components)

  - [Buck Converter](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#buck-converter)

  - [Battery](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#battery)


[Communication Protocols](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#communication-protocols)

  - [CAN](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation#can)



<img src="https://github.com/Axel-Barbelanne/jimny-wiki/blob/main/images/components_sketch.svg" width="600">
<br>
<br>


| No. | Component  | No. | Component        |
| --- | ---------- | --- | ---------------- |
| 1   | Sterfboard | 9   | Emergency Button |
| 2   | Steerbok   | 10  | Buttons          |
| 3   | Axelbrake  | 11  | Stepper Motor    |
| 4   | Jetson     | 12  | Boost Converter  |
| 5   | CubePilot  | 13  | Stepper Driver   |
| 6   | HereLink   | 14  | Angular Encoder  |
| 7   | GPS        | 15  | Buck Converter   |
| 8   | Camera     |     |                  |

---

## Modules


### Sterfboard


![[sterfboard.jpg|400]]

##### Function

The Sterfboard serves two main purposes: it selects the operation mode for the Jimny, and transmits any relevant information to the other modules via CAN protocol. 

The mode selection is based on several criteria, namely:
- Warning flag of the Jetson, which can be one of:
	- Disarmed 
	- Armed, valid signal
	- Armed, invalid signal (error)
- Button state
	- Emergency button
	- Manual override button
- Whether communication has been lost with the Jetson
The decision of whether the modules should be in manual, autonomous, or emergency mode is then carried out by activating the 48V relay accordingly for the stepper motor, and by communicating the decided mode to the other modules.

The communication aspect mainly relays information received by the Jetson through UART. We hoped to do this via CAN from the beginning but were unable to get our hands on a CAN transceiver for the Jetson. The communication is therefore done through a 3 cable connection (Rx, Tx, GND, see [[#Set-up]]) from the Jetson to the Sterfboard. After receiving this information, this module will process it and publish it to the CAN bus which is shared with the two other modules. See more [[#CAN|below]] for more information on the CAN bus. 

For the Sterfboard, power is received via a regulated 9V input on the board from a buck converter, and is delivered to other modules over the same ethernet that implements the CAN bus.

##### Set-up

Like all three modules in this section, the Stefboard was designed from scratch and hence needs to have all its components soldered manually. All the relevant information on the circuitry can be found on the main repository at: Jimny > Parts > PCBs > Jimny_power.

Besides that, the arduino needs to be flashed with the Sterfboard code using Arduino IDE, which can be found at: Jimny > arduino_sketches > sterfBoard.ino.

As for mounting, the Sterfboard is currently simply held by velcro where the original Jimny screen was (see diagram at the top). 

The cables that need to be connected include:
- 48V in/out
- 9V in
- Ethernet (to Steerbok)
- Buttons: GND, 5V, sig
- Emergency button: 5V, sig
- UART cables on the Arduino

##### Additional Information / Debug

The Arduino in this module can only be flashed if it is removed from the board. Be careful to align the pins properly when putting it back in.

It was also found that if no CAN messages were received when they were supposed to, re-flashing the Arduino with the associated code sometimes fixed the issue.

Like all modules, the Arduino used is an outdated mini-B cable. The issue with these is the drivers required that always pose annoying problems when switching between these Arduino's and newer ones. In order to use these, we had to install a driver which we provided in Jimny > arduino_sketches > CH34x_Install_Windows_v3_4.EXE. We also add to toggle a parameter in Arduino IDE, namely: Tools > ATmega328P (Old Bootloader).

Additionally on this module, we had to solder the Rx and Tx pins on the Arduino. This is because we were hoping to be able to simply connect the Arduino via USB to the Jetson, exactly how it worked with the ESP32's. Unfortunately this is not currently allowed, which meant that we had to communicate via UART with the 40 pin header on the Jetson, after the PCB's had been designed. 

Several indicator LED's were added on the PCB for debugging, these are:
- Power LED: Indicates whether power is provided to the Sterboard
- 9V Relay: Indicates whether 9V is provided to other modules
- 48V Relay: Indicates whether 48V is provided to the stepper driver
- CAN: Blinks if a CAN message is received
- Emergency: Shines if the emergency button is pressed
- Button LED's: one for each button, shines if the button is pressed
- Jumper between pins 1 and 2 of J8 header

##### Improvements

The currently soldered connections should be implemented in the PCB if it is re-printed.


### Steerbok

![[steerbok.jpg|400]]

##### Function

The Steerbok module controls, as the name suggests, the steering of the vehicle. The module takes feedback of the current steering angle, from an angular encoder linked to the steering shaft and converges to the desired angle as communicated by the Sterfboard via CAN bus.

The module receives its needed information from the CAN bus as well as its power supply over the ethernet cable. 

This module is also the intermediate device in the CAN bus, between the Sterfboard and Axelbrake, though the order doesn't really matter.

##### Set-up

Like the Sterfboard, the PCB must be soldered and assembled. The relevant files are in Jimny > Parts > PCBs > Jimny_steering.

Again the arduino needs to be flashed with the Steerbok code using Arduino IDE, which can be found at: Jimny > arduino_sketches > Steerbok.ino.

As for mounting, the Steerbok is currently simply held by velcro where the original Jimny screen was (see diagram at the top). 

The cables that need to be connected include:
- Angular encoder (2 pins to connectors, 2 pins to Arduino)
- Stepper driver
- Ethernet 1 (to Sterfboard)
- Ethernet 2 (to AxelBrake)

##### Additional Information / Debug

See [[#Sterfboard]] for information on drivers to flash the Arduino.

The system was originally designed to work with a potentiometer rather than an angular encoder. Unfortunately however, after a lot of testing and driving around the vehicle, the potentiometer ended up loosening up a little and breaking, since it was operating so close to its maximum range. The potentiometer was therefore replaced by an angular encoder, to ensure this would not happen again since the encoder does not have a maximum range. Since this was done quite late,  the wiring needed to be adapted to the PCB, and two cables had to be soldered to the Arduino. 

Several indicator LED's were added on the PCB for debugging, these are:
- Power: indicates that power is received over ethernet
- CAN: blinks when a CAN message is received
- Enable: shines when the stepper is enabled
- Pulse R/L: blink when pulses are sent to the stepper in the respective direction. Note that the blink duration correspond to the speed of the stepper (faster rotation → faster blinks)

##### Improvements

The soldered wires on the Arduino are not ideal, and the PCB should eventually be re-designed to be adapted to the angular encoder rather than the potentiometer.

Additionally, a better mounting system should be designed than the velcro attachment.


### AxelBrake

![[axelbrake.jpg|400]]

##### Function

This module serves to interface with the acceleration and the braking of the Jimny, hence the name (for the record, Matthijs named it not me).

The module receives its needed information from the CAN bus as well as its power supply over the ethernet cable. 

Several relays will ensure the correct and safe signal is output to the acceleration plug, based on the mode, the input acceleration or the desired velocity, and whether the brake is pressed.

The braking servo is connected via its header pins on the board, whose 6V is regulated by an external buck converter (see small PCB just above the board in the image above).

##### Set-up

Like the Sterfboard, the PCB must be soldered and assembled. The relevant files are in Jimny > Parts > PCBs > Jimny_throttle.

Again the arduino needs to be flashed with the Sterfboard code using Arduino IDE, which can be found at: Jimny > arduino_sketches > AxelBrake.ino.

The mounting for this module is a little annoying and consists of fastening it under the steering wheel, against the side of the vehicle. This is pretty hard to reach and might drive you a little crazy at the beginning. Make sure everything is properly fastened and connected before doing this.

The cables that need to be connected include:
- Input/Output acceleration cables
- Braking input
- Buck converter input/Output
- Servo output
- Ethernet (to Steerbok)
- Jumper between pins 1 and 2 of J1 header

##### Additional Information / Debug

See [[#Sterfboard]] for information on drivers to flash the Arduino.

If an issue is seen with this board, it could well be that one of the cables got unsoldered or disconnected during the mounting process. Particularly look at the soldered connections on the buck converter.

Several indicator LED's were added on the PCB for debugging, these are:
- Power: indicates that power is received over ethernet
- Brake: indicates if the brake is engaged
- Free: Indicates if the brake is free/disengaged (debug)
- CAN: blinks when a CAN message is received
- Signal debug: 2 LED's, shine when the respective signal is being sent from non-manual mode to the acceleration output


##### Improvements

The connections, particularly those soldered on the buck converter should be solidified for robustness. Alternatively, a different mounting system could be found.

Additionally, a better mounting system should be designed than the velcro attachment.


---

## Jetson


##### Function

The Jetson AGX Orin serves as the main brain of the system. This is where all the heavy software is run including MPC, obstacle detection, etc. Additionally, it is also the central node that interfaces between the CubePilot, the sensors, and the modules.

##### Set-up

The password for the Jetson can be found in the Documentation in Google Drive. 

Not much needs to be mounted/changed on the hardware side.

The Jetson AGX Orin already has mounted on it a 2TB external hard drive, as well as a camera driver. The Jetpack version is also already on version 6.0.

The UART connection needs to be connected to the Sterfboard. The relevant pins are the 40-pin header are GND, UART1_TX and UART2_TX. In the pinout below, note that pin 1 is the pin on the top right.
The pinout is the following:

![[datasheet_jetson.png]]

##### Improvements

The communication mechanism would be more reliable/consistent if a CAN transceiver could be used so that all the interactions between the Jetson and the modules could be done via CAN. 

A better mount should also be designed so that the Jetson is fixed in place


---

## OrangeCube

![[orangecube.svg|300]]

##### Function

Unlike its typical usage, the OrangeCube for us is simply the interface between the GPS and HereLink, and the rest of the system.

Note that we use the terms OrangeCube and CubePilot intechangeably.

##### Set-up

Not much hardware set up is required here either.

Two cables should be plugged into the GX16 connectors, in addition to the 9V power supply that should be fixed into the screw terminal. These are for the GPS and HereLink and are both labeled. The GPS cable should be connected to the male connector with 4 connected wires, on the left side. The HereLink is the second with 7 connected wires, on the right.

For reference, the wires connected are the following:

GPS:
1. VCC (RED) (5V)
2. GND (BLACK)
3. CANH (BROWN)
4. CANL (ORANGE)
   
HereLink:
1. VCC (9V)
2. GND
3. UART TXD
4. UART GND
5. SBUS out 1
6. SBUS GND
7. -
8. UART RXD (maybe better to swap with 4?)
   

---

## Sensors


### HereLink receiver

##### Function 

The two HereLink antennas and the receiver work together to receive the signal from the HereLink Controller and pass it on to the OrangeCube.

##### Set-up

One of the wires going through the roof has an ending that barely fits through the connector. It's a bit annoying but you can use a tool to force it through.

##### Improvements

The mounting definitely needs to be improved, especially for the antennas which are taped on the GPS mount.

### GPS

![[gps.jpg|350]]

##### Function 

The GPS currently serves as the main localization method for the system, with the actual navigation relying on waypoints defined by GPS coordinates and orientation, as well as optional velocity data.

##### Set-up

The set-up is pretty simple, the GPS clips into its spot and a lid can be screwed on to be fixed. Note that the heated inserts are messed up on the mount and you'll probably only be able to fix two of them inside, but it should be enough to hold the GPS tightly. The mount also allows for the two HereLink antennas to be fixed on the sides at a set angle (around 10 degrees). There is only one cable to plug into a GX16 connector, as seen on the image above.

Note that the mount is made out of ASA in order to be both waterproof and UV-resistant.

You may be asked by the HereLink controller to re-callibrate the compass. If that's the case, you'll just need to take out the GPS and shake / rotate it around until the progress bar finishes loading (might take a little while).

##### Improvements

The mount needs to be re-made in order to fix the lid screwing problem.

##### RTK System

The RTK sharply increases the GPS precision. We already tested its set-up and found around 10cm precision within a few days, which is significantly better than GPS on its own.

We have not, however, set it up on the vehicle; this remains to be done. A mount should be designed for the antenna which should fit on the vehicle roof.

### Camera

![[camera.jpg|300]]

##### Function 

This is a ZED X Stereo Camera, which allows for a really wide field of view. We tried to maximize this with the case, which is declined by 10 degrees. The casing is made of PETG, which is waterproof. The case also includes a plexiglass screen for protection

##### Set-up

The cable simply needs to be plugged into the Jetson. Note that 3 more cameras could potentially be added.

##### Improvements

The case should be slightly modified to allow a watertight fit with the plexiglass. It should also be reprinted in ASA in order to be UV resistant

A second camera could also be added down the line, for instance for rear view.


---

## Output Components


### Stepper

![[steering.jpg|300]]

##### Function 

The stepper, along with its wheel handle, ensure the steering wheel gets turned as desired. The stepper is a NEMA-34 motor. We upgraded to this motor from the NEMA-23 to ensure we had enough torque on the wheel.

Note that by itself, the motor does not have enough torque to fully operate the steering wheel. What makes this system work effectively is the electric power steering which supports the steering at the shaft level.

We fixed springs to the motor to ensure it had enough force pressing down on the wheel, in addition to the weight of the motor (heavier than you'd think).

The end bit that spins the wheel is for now a prototyped part made of PLA. We added insulating tape for grip.

##### Set-up

The mounting system for the stepper is a multi-part system that allows sliding in two axes. The easiest way to understand its mounting is by viewing the assembly files which can be found at Jimny > parts > CAD > steering > steering motor.

After this part, there is simply the end-bit to insert, and after adjustment the springs to add for which we used two pairs of zipties.

##### Improvements

Very importantly the end-bit needs to be reprinted in a more durable material, or even machined to withstand the strong forces. The coupling should also be slightly adapted, since the NEMA-34 we received was not the same as on the spec sheet as it included 2 flat sides on the cylindrical shaft; this is a lot better for grip, but the coupling for this needs to be adapted.

### Boost Converter

##### Function 

The boost converter is a 12V-48V converter which allows us to get an efficient use of the stepper motor.

The 12V input is switched on/off by a relay in the Sterfboard, which decides if the stepper will be drawing current or not.

### Stepper Driver

##### Function 

This is the driver associated with the NEMA-34 stepper motor. The cables are separated into the high-power wires on one end, that interface with the power input and the motor itself, and the low-power cableson the other that interface with the Steerbok.


### Angular Encoder

Below is the mount and transmission system for the angular encoder on the vehicle.
![[assembly.png|400]]
##### Function 

The angular encoder serves to give a feedback to the Steerbok on where the steering angle currently is.

Originally, a potentiometer was designed for this. Its range was however limited and close to the maximum steering angles. We therefore had a problem one time that the potentiometer, which must have slipped slightly, broke due to it reaching a rotating limit. An angular encoder was therefore used to prevent this from happening again and has the added benefit of calibrating  itself back to 0 every time, which means it wouldn't get affected by long-term slipping or shocks.

The mounting system for this is a little complex, and the mount is based on previous work by Hubert Visser and Marnix van der Berg which was originally made to hold a stepper motor. This was adapted by adding a bearing-support system to that the chain linking the steering shaft and the sprocket on the angular encoder shaft is not directly pressing on the encoder itself.

##### Set-up

Exploded view of key parts of the angular encoder mount and transmission system.
![[assembly_exploded.png|400]]

The mounting is a little difficult to do, mostly due to its positioning under the steering wheel.

Again, the best thing to do is to take a look at the assembly in Jimny > parts > CAD > steering > steering feedback.

We found the easiest way to mount was the following: after having all the components assembled together separate from the car, put the chain through the sprocket, about halfway in. Next set up the large printed parts (with the holes) up behind the steering shaft. There is a cable that gets in the way on the left side, just push it out of the way. Fix it with bolts, then slide the chain around the sprocket on the steering shaft. Now you can tighten by screwing the two screws on the bottom of the sliding mounting print, and by adjusting the position of the sprocket on the steering shaft. Make sure the chain remains close to parallel with the sprockets.
##### Improvements

The part of the current assembly connected to the angular encoder is a prototype that was done by hand, as we did not have access to the 3D printers and had limited time. This should therefore be modified in the CAD assembly and reprinted in order to adapt from a potentiometer to an angular encoder.

Additionally, the screws that tighten the mount will come loose a little after a while and need to be retightened. Something could be added to improve the grip here.


## Acceleration plug

##### Explanation

The accelerator plug is part of the [[#AxelBrake]] module and allows for interaction with the acceleration input of the vehicle. It is done by taking the input of the pedal into the AxelBrake, and providing an output into the female plug in the car. 

The acceleration signal consists of two voltages, the second being the double of the first. The first ranges from 0.38V - 2.23V and the second from 0.76V - 4.46V. More can be read on this on the Documentation Google docs in the drive, or on the PCB files (See [[#AxelBrake]]).

## Braking servo

##### Explanation

The braking servo serves to pull the braking pedal with a metal wire. It is equipped with a printed mounting bracket.

##### Improvements

One of the most important improvements of the whole system is to upgrade this servo to one with more power. Currently it does not push in all the way and is not enough to keep the car reliably static.


---

## Additional Components


### Buck Converter

##### Function 

This DC-DC buck converter supplies a regulated 9V from a 12V battery. The 9V is supplied to the SterfBoard and to the OrangeCube. This can be seen in the diagram in [[#Battery]].

##### Improvements

A better mounting system could be designed.

### Battery

Power circuitry for the system
![[power_circuitry.svg]]

##### Function 

The battery serves as the power source for the whole system. 

We were previously using the cigarette lighter in the car, but this was not providing enough power for the whole system, notably including the big stepper motor. If the cigarette lighter is to be used down the line, a fuse must be replaced in order to get it working.

We hence use the large battery at SPOTS which can be equipped with an inverter to power the Jetson. Cables have been made with an XT30 connector for an easy connection.

##### Improvements

The cigarette lighter could be used for power, if accompanied by a bigger battery / maybe alternator and fuses.


---

## Communication Protocols

#### CAN

We chose to use the CAN protocol between the modules for its numerous advantages including reliability, flexibility in adding/removing devices and data prioritization. Additionally, CAN is already widely used in vehicles for communication between components.

Below is an overview of the CAN messages that are used.

| CAN ID | Sender     | Purpose               | Data 1               | Data 2                  |
| ------ | ---------- | --------------------- | -------------------- | ----------------------- |
| 0x01   | Sterfboard | Heartbeat & Data      | Interval             | Mode                    |
| 0x11   | Steerbok   | Hearbeat & Feedback   | Steering Rate target | Steering angle feedback |
| 0x110  | Sterfboard | Desired steering info | Steering angle       | Steering angle velocity |
| 0x12   | AxelBrake  | Heartbeat & Feedback  | Velocity target      | Velocity feedback       |
| 0x120  | Sterfboard | Desired velocity info | Desired velocity     | Desired acceleration    |
| 0x121  | Sterfboard | Real velocity info    | GPS velocity         |                         |
