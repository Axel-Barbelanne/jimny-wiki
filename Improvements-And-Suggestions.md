
Quite a lot of improvements are suggested throughout this wiki, particularly in the [Technical Documentation](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Technical-Documentation) and [Software Overview](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Software-Overview) sections. We will present them here, in a more ordered and hierarchical way. These improvements are of course our suggestions, after working on the project and seeing it from our perspective. To future interns, please feel free to find alternative solutions to the issues presented, or to bring forward any issues not mentioned here.

Contents:
[Critical Issues](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Improvements-And-Suggestions#critical-issues)
[Important Improvements](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Improvements-And-Suggestions#important-improvements)
[Further improvements](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Improvements-And-Suggestions#further-improvements)
[Additional Potential Features](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Improvements-And-Suggestions#additional-potential-features)
[Other suggestions](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Improvements-And-Suggestions#other-suggestions)


### Critical Issues

**Braking servo**:
	The servo currently implemented is not powerful enough to fully push the brake pedal in. A stronger servo should be used, and its mount re-designed.

**Printed steering bit**:
	The steering bit on the stepper shaft is made out of PLA and is already starting to break down. It  should be re-designed slightly to be adapted to the flat edges of the stepper shaft, and produced out of a more robust material.


### Important Improvements

**PID controller for acceleration and braking**:
	A single PID is currently used for both acceleration and braking, despite these being two very disparate systems. These should either be separated into two PID's, or the outputs of the PID post-processed to be applied in two different ways to acceleration and to braking. The first alternative is probably the cleaner solution

**Installation of the RTK antennas**
	This will be a major improvement in terms of GPS accuracy. This will most likely require extra drilled holes through the roof and connectors for the wiring

**Re-design of fragile 3D printed parts**
	Several 3D parts that were designed as mere prototypes in PLA are currently mounted on the vehicle for instance as mount for the angular encoder or for the stepper. These should be replaced by stronger materials to avoid any incidents with them breaking


### Further improvements

**Telemetry improvements**
	There is a lot of potential work to be done on the telemetry side, and this will be necessary to increase the range of the Jimny

**Mounting Improvements**
	A lot of the components are currently just velcro'd on due to time constraints, but this should be improved for things not to move while driving

There are many more future improvements to be made of course, a little more on the future objectives can be seen in [Getting Started](https://github.com/Axel-Barbelanne/jimny-wiki/wiki/Getting-Started).


### Additional Potential Features

- **Mode shift module**:
	Extra module for shifting the modes: parking/reverse/neutral/drive
- **Ignition module**
	Extra module for starting/turning off the car autonomously
	Can also simplify the entire start-up procedure
- **Lighting module/Nagapie mounting**
	Extra module controlling the lighting with the Nagapie (separate SPOTS project) mounted on the car
- **AI footstep detection**
	Implement footstep detection on the Jetson, with an extra camera placed correctly on the Jimny


### Other suggestions

Here's a few suggestions, more on the SPOTS project side of things:

- Update the documentation
	The SPOTS internship is quite short, and we spent a lot of time at the beginning just understanding what had been done be for. Try to minimize this for the next batch!
- Order components as early as possible
	The components sometimes take really long to arrive to Dinokeng. Try and order things as soon as you think you'll need them, and for small components order some extras in case. Also figure out early on what you can work on while you wait
- Contact us if you need any information that's not in the wiki, we're happy to help!
	axel.barbelanne@gmail.com
	mdsteyerberg@gmail.com