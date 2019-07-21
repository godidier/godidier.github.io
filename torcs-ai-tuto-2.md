title: Building a Simple TORCS AI Agent (2/11): Acceleration
date: 2018-12-08 15:21
category: Games
authors: Didier Gohourou
summary: Making the car accelerate slightly and steering basically.
slug: torcs-ai-tuto-2
lang: en

This is the second chapter of a tutorial for building an AI agent for the racing 
game TORCS. In this chapter, we will generate the base code of the AI agent, 
make it drive slowly, and reorganize the code for easier implementation.

* [Table of Contents]({filename}torcs-ai-tuto-1.md#table_of_contents)
* [Previous: Getting Started]({filename}torcs-ai-tuto-1.md)
* [Next: Brakes and Gears]({filename}torcs-ai-tuto-3.md)

# Generate the AI agent base code

To start building our AI agent, we will use a tool called `robotgen`, to create 
the baseline code structure. robotgen is used as following: 

```
robotgen -n <name> -a <author> -c <car> [-d description] [--gpl]
```

Where `<name>` is the name of the agent we are creating, `<author>` is the name of 
the creator (AKA you) and `<car>` is the name of the car we want our agent to be 
using.  We might also use optional argument to give a description of our agent, 
or add a GPL statement at the beginning of the source code files.

The list of available cars can be obtain as follow:

```
$ ls /usr/local/share/games/torcs/cars/ 
155-DTM car2-trb1 kc-a110 kc-dbs pw-206wrc 
acura-nsx-sz car3-trb1 kc-alfatz2 kc-dino pw-306wrc 
baja-bug car4-trb1 kc-bigh kc-ghibli pw-corollawrc 
buggy car5-trb1 kc-cobra kc-giulietta pw-evoviwrc 
car1-ow1 car6-trb1 kc-coda kc-grifo pw-focuswrc 
car1-stock1 car7-trb1 kc-conrero kc-gt40 pw-imprezawrc 
car1-stock2 car8-trb1 kc-corvette-ttop kc-gto 
car1-trb1 kc-2000gt kc-daytona kc-p4 
car1-trb3 kc-5300gt kc-db4z p406
```
Now let's move to the `$TOCS_BASE` folder, and use the robotgen utility is and 
generate the initial source code of our agent.

```shell
cd $TORCS_BASE 
./robotgen -n "gtr" -a "Godidier Tech Report" -c "car1-trb1"
```

This command will create a folder with the name of your ai agent inside 
`$TORCS_BASE/src/drivers`. Inside that folder (in our case 
`$TORCS_BASE/src/drivers/gtr`), we will have the following files:

* **Makefile** : Instruction for compiling and installing your AI agent
* **gtr.cpp** : Actual code of your AI agent
* **gtr.xml** : Customizable properties for your agent
* **gtr.def**, **gtr.dsp** : Some Visual Studio (2005) project files. 
Not much useful anymore
* **car1-trb1.rbg** : Image file used as paint job for the car
* **logo.rbg** : Image file used as paint job for your driver's name and flag 
and your team's logo
* **car1-trb1.xcf** : GIMP project file to edit your paint job's image files

# Install the AI Agent

Let's build and install the AI agent. For that, we move to the directory of our 
agent, and use  the command `make`.

```shell
cd $TORCS_BASE/src/drivers/gtr
make 
make install 
```
Note: We might need writing permission to the directory where TORCS library are 
installed `/usr/local/lib/torcs/`.

Once done, let's test our agent (I am so excited...). Run the game with a 
double-click on the desktop icon of TORCS or typing the command `torcs` 
in the command line. Select _Race > Practice > Configure Race_. Choose a track: 
_CG Speedway number 1_, and click _Accept_. Click on your AI agent name in the 
_Not Selected_ column on the right and click _(De)Select_ to put it in the 
_Selected_ column. Only one participant is allowed in practice session so if 
there is already another participant in the _Selected_ Box, you have to remove 
it first by clicking it and click the _(De)Select_ menu item, and then choose 
your AI agent like we just describe. Then click _Accept_. After that, 
select the number of laps and display.  We may leave it as is (2 laps). 
Click _Accept_, then click _New Race_ in the _Practice_ menu. 
Wait... Yay our car is on the road!

![Car on road](#)

Hey wait! but it is not doing anything ?!...

....That was the joke folks ! (I am sure it bombed)

Yes our AI agent is just chilling because we haven't gave it any instructions.
But before we pursue, let's learn to remove our AI agent. First quit the game 
and then delete the 'installed' driver folder from /usr/local/lib/torcs/drivers/. 
In our example, we can run: 

```bash
rm -f /usr/local/lib/torcs/drivers/gtr
```

but for future easier uninstalls, add the following lines at the end of the 
_Makefile_ in our AI Agent folder:

```make
uninstall: 
	rm -rf ${libdir}/${MODULEDIR}
```

From now, can comfortably uninstall by typing `make uninstall` from the our 
AI Agent development folder.

# Before We Write Some Code

In this chapter of the series, we implement basic steering and acceleration. 
All in the AI agent main cpp file represented by `gtr.cpp`. Let's check the 
content of the file.

Open the file with your favourite text editor, one with syntax highlighting should 
be enough. The file start with some standard C language header files. Then we have 
some TORCS header files that includes the necessary data structures and functions 
for the implementation of our agent. Note: those file are accessible via 
` $TORCS_BASE/export/include`.

* tgf.h : Generic gaming API utilities
* track.h : Structure and loader of tracks
* car.h : Data structure (and sub structure) representing a car
* raceman.h : Data Structures providing information during a race
* robottools.h : Utility functions to compute values useful to the AI agent (robot)
* robot.h : AI agent (robot) module interface definition. Contains callback functions that operate the robot.

After the headers come the declaration and definition of the function we will edit to build our AI agent.

`initTrack`: Called for every track change. Can be used to deal with some 
initialization before a new race.
```C
static void initTrack(int index, tTrack* track, void *carHandle, void **carParmHandle, tSituation *s); 
```

`newrace`: Used as callback when a new race starts.
```C
static void newrace(int index, tCarElt* car, tSituation *s);
```

`drive`: Used as callback to get the driving instructions.
```C
static void drive(int index, tCarElt* car, tSituation *s);
```

`endrace`: Used as callback when the current race ends.
```C
static void endrace(int index, tCarElt *car, tSituation *s);
```

`shutdown`: Called before the module is unloaded. Can be used to free memory.
```C
static void shutdown(int index);
```

`InitFuncPt`: Module interface initialization. Setup the implemented callback 
function that will operate the AI agent.
```C
static int InitFuncPt(int index, void *pt);
```

`gtr` (AI agent name): Module entry point. Used to initialize the AI agent 
module properties.
```C
extern "C" int gtr(tModInfo *modInfo) 
{ 
... 
}
```

# Let's drive

To drive we set the values of our car control. Those values are updated every 
0.2 second by the method drive. the control values are: 
* `car->ctrl.steer` between -1.0 and 1.0
* `car->ctrl.gear` between -1 and the car max gear
* `car->ctrl.accelCmd` between 0 and 1.0
* `car->ctrl.brakwCmd` between 0 and 1.0

To accelerate we just press the accelerator right ?! (set it to 1.0) ... 
We might as well end up in the grass or the wall a the first turn you encounter! 
So I think we might want some basic steering. For our first drive the plan is 
to accelerate slowly and steer in a way to always stay in the middle of the track. 
My laziness sense tells me that it is time to quote the tutorial shipped with 
the game.

> Before we can start implementing simple steering we need to discuss how the track 
> looks for the robot. The track is partitioned into segments of the following 
> types: left turns, right turns and straight segments. The segments are usually 
> short, so a turn that looks like a big turn or a long straight is most often 
> split into much smaller segments. The segments are organized as linked list in 
> the memory. A straight segment has a width and a length, a turn has a width, a 
> length and a radius, everyting is measured in the middle of the track. 
> All segments connect tangentially to the next and previous segments, so the 
> middle line is smooth. There is much more data available, the structure 
> tTrack is defined in $TORCS\_BASE/export/include/track.h.

The image below from [Wikipedia](https://en.wikipedia.org/wiki/Speed_Dreams#Tracks)
give an illustration of the quote above.

![Track Segment section]({static}/images/2018-12/Speed_Dreams_Track_Subsegments-Sections_animation.gif)

The goal of our simple steering function is to make the car follow the middle of 
track. For that, we steer the front wheel parallel to the track, and we add a 
correction value if we are not in the middle of the track.

To know how much we need to steer the wheel, we need to know three things: 

* The angle formed by the middle line of the starting segment of the track and 
the tangent line to the segment where the car currently is. It is obtained via 
the function `RtTrackSideTgAngleL(&(car->_trPos))` from `robottools.h`.  
* The angle formed by the middle line of the starting segment of the track and 
the **yaw** direction of the car. <del>I am not talking about yawning tho.</del> 
This is obtained via `car->yaw`. A yaw rotation is a movement around the yaw 
axis. The 3 axes of a car, plane have specific names: roll, pitch and yaw.
![Rotational Axis]({static}/images/2018-12/rotational-movements.svg)
* The distance between the car and the middle line of the segment where the car is.
 This is obtained via `car->_trkPos.toMiddle`, which is positive if the car is to 
the left of the middle line and negative if the car is to the rigth of the middle 
line. This value is in meter so to contribute to steering angle, it will be 
normalized by the width of the segment `car->_trkPos.seg->width`.

Now to compute the steering angle, we substract 
`RtTrackSideTgAngleL(&(car->_trPos))` from `car->_yaw`, we make sure to make it 
lies between &pi; and -&pi;, either using `NORM_PI_PI(angle)` from the game API, 
or the `remainder(double x, double y)` function from _math.h_. Next we substract 
`car->_trkPos.toMiddle` (in meters), normalized by `car->_trkPos.seg->width` 
from the obtained value.  We then convert the angle to the steering 
range [-1, 1] by dividing the angle by the maximum steering value of the car: 
`car->_steerLock`.

We also accelerate slowly (by 30%), with the gearbox set to the first gear.
All of this is implemented within the `drive` function that will look like:

```cpp
static void drive (int index, tCarElt* car, tSituation *s)
{ 
    memset((void *)&car->ctrl, 0, sizeof(tCarCtrl)); // reset the values

	float angle = RtTrackSideTgAngleL(&(car->_trkPos)) - car->_yaw;
	angle = remainder( angle, 2*PI); // making sure angle is within -PI and PI
	angle -= car->_trkPos.toMiddle / car->_trkPos.seg->width;

    car->_steerCmd = angle / car->_steerLock;
    car->_gearCmd = 1; // first gear
    car->_accelCmd = 0.3; // 30% acceleration
    car->_brakeCmd = 0.0; // no brakes
}
```

The picture below try to give you a better intuition of what is going on with the 
angles, and the algorithm above.

![Steering Angles]({static}/images/2018-12/steering-angles.svg)

# Test drive 

add video

# Restructuring the code


Now let's prepare for the coming chapters. We will reorganize the code in a more 
modular way. We will create a class that will be responsible for controlling 
our AI agent. Let's call it `CarController` (yeah, very original).

We create a file named _carcontroller.h_ and define the class. The content of 
the class are callback methods, some properties of our AI module, and helper 
methods to compute controls values.  We also put the car and the track as 
members of the class for easier access.


```cpp
#ifndef _CARCONTROLLER_H_
#define _CARCONTROLLER_H_

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>

#include <tgf.h>
#include <track.h>
#include <car.h>
#include <raceman.h>
#include <robottools.h>
#include <robot.h>

class CarController {

	public: 
		CarController();
		void InitTrack(tTrack* track, void* carHandle, void** carParmHandle, 
						tSituation* situation);
		void NewRace(tCarElt* car, tSituation* s);
		void Drive(tSituation* situation);
		void EndRace(tSituation* situation);
		int PitCommand(tSituation* situation);
		char* GetName();
		char* GetDescription();
		
	private:
		float GetAcceleration();
		float GetBrake();
		float GetGear();
		float GetSteering(float car_angle);
		float CurrentCarAngle(tSituation* situation);

		tCarElt* car;
		tTrack* track;

		char* name;
		char* description;

};
```

Create a file called _carcontroller.cpp_ and define and define the methods as 
follows:

```cpp
#include "carcontroller.h"


CarController::CarController()
{
	this->name = strdup("gtr");
	this->description = strdup("");
}

/**
 * Callback before the start of every race.
 */
void CarController::InitTrack(tTrack* track, void* carHandle, void** carParmHandle, 
		tSituation* situation)
{
	this->track = track;

	*carParmHandle = NULL; 
}

/**
 * Called at the start of a new race
 */
void CarController::NewRace(tCarElt* car, tSituation* s)
{
	this->car = car;
}

/**
 * Function that setup the controls values to drive the car during the race.
 */
void CarController::Drive(tSituation* situation)
{
    memset((void *)&car->ctrl, 0, sizeof(tCarCtrl)); // reset the values

	float car_angle = CurrentCarAngle(situation);

    car->_steerCmd = GetSteering(car_angle);
    car->_gearCmd = GetGear(); 
    car->_accelCmd = GetAcceleration(); 
    car->_brakeCmd = GetBrake(); 
}

/**
 * Return the name of the controller, to be used as module name.
 */
char* CarController::GetName()
{
	return this->name;
}


/**
 * Return the description of the controller to be used as the module description
 */
char* CarController::GetDescription()
{
	return this->description;
}

/***
 * End of the current race
 */
void CarController::EndRace(tSituation* situation)
{
	// TODO 
}

/**
 * Command for the pitstop
 */
int CarController::PitCommand(tSituation* situation)
{
	return ROB_PIT_IM; // return immediately
}
		
/**
 * Compute and return the acceleration value to be applied.
 */
float CarController::GetAcceleration()
{
	return 0.3; // 30% acceleration
}

/**
 * Compute and return the braking value to be applied.
 */
float CarController::GetBrake()
{
	return 0.0; // No brakes
}

/**
 * Compute and return the gear to be applied.
 */
float CarController::GetGear()
{
	return 1; // first gear
}

/**
 * Compute and return the steering value.
 */
float CarController::GetSteering(float car_angle)
{
	float steering_angle;
	steering_angle = car_angle - car->_trkPos.toMiddle / car->_trkPos.seg->width;
	return steering_angle / car->_steerLock;
}

/**
 * Computes the current angle of the car relatively to the track
 */
float CarController::CurrentCarAngle(tSituation* situation)
{
	float car_angle = RtTrackSideTgAngleL(&(car->_trkPos)) - car->_yaw;
	car_angle = remainder( car_angle, 2*PI); // ensure it is within -PI and PI
	return car_angle;
}

```

Let's go back to _gtr.cpp_, and update it accordingly. First, we include our 
CarController class header file and define a static variable that will hold 
an instance of our CarController. Then we instanciate the `CarController` within 
the module entry point method `gtr(...)`. You may also notice that the module 
interface initialization function in `InitFuncPt(..)` is missing a callback. 
The `itf->rbPitCmd` is set to `NULL`. Let's set it to `pitcommand`, a callback 
function that we will define shortly after. The remaining of the code is basically 
about replacing the callback functions their counterparts from the `CarController`
class.  Here is the listing of the updated _gtr.cpp_, with some formatting.



```cpp
#include "carcontroller.h"


CarController::CarController()
{
	this->name = strdup("gtr");
	this->description = strdup("");
}

/**
 * Callback before the start of every race.
 */
void CarController::InitTrack(tTrack* track, void* carHandle, void** carParmHandle, 
		tSituation* situation)
{
	this->track = track;

	*carParmHandle = NULL; 
}

/**
 * Called at the start of a new race
 */
void CarController::NewRace(tCarElt* car, tSituation* s)
{
	this->car = car;
}

/**
 * Function that setup the controls values to drive the car during the race.
 */
void CarController::Drive(tSituation* situation)
{
    memset((void *)&car->ctrl, 0, sizeof(tCarCtrl)); // reset the values

	float car_angle = CurrentCarAngle(situation);

    car->_steerCmd = GetSteering(car_angle);
    car->_gearCmd = GetGear(); 
    car->_accelCmd = GetAcceleration(); 
    car->_brakeCmd = GetBrake(); 
}

/**
 * Return the name of the controller, to be used as module name.
 */
char* CarController::GetName()
{
	return this->name;
}


/**
 * Return the description of the controller to be used as the module description
 */
char* CarController::GetDescription()
{
	return this->description;
}

/***
 * End of the current race
 */
void CarController::EndRace(tSituation* situation)
{
	// TODO 
}

/**
 * Command for the pitstop
 */
int CarController::PitCommand(tSituation* situation)
{
	return ROB_PIT_IM; // return immediately
}
		
/**
 * Compute and return the acceleration value to be applied.
 */
float CarController::GetAcceleration()
{
	return 0.3; // 30% acceleration
}

/**
 * Compute and return the braking value to be applied.
 */
float CarController::GetBrake()
{
	return 0.0; // No brakes
}

/**
 * Compute and return the gear to be applied.
 */
float CarController::GetGear()
{
	return 1; // first gear
}

/**
 * Compute and return the steering value.
 */
float CarController::GetSteering(float car_angle)
{
	float steering_angle;
	steering_angle = car_angle - car->_trkPos.toMiddle / car->_trkPos.seg->width;
	return steering_angle / car->_steerLock;
}

/**
 * Computes the current angle of the car relatively to the track
 */
float CarController::CurrentCarAngle(tSituation* situation)
{
	float car_angle = RtTrackSideTgAngleL(&(car->_trkPos)) - car->_yaw;
	car_angle = remainder( car_angle, 2*PI); // ensure it is within -PI and PI
	return car_angle;
}

```

Compile and run it to ensure that everything went smoothly. If not and you are 
on the edge of tearing of your hairs out of your head, please download or clone 
this [repository](#).

That's it for this chapter!... And yes we have a working agent! But let's be 
honest it is kind of lame. So let's make him faster, break, and change gears.


* [Table of Contents]({filename}torcs-ai-tuto-1.md#table_of_contents)
* [Previous: Getting Started]({filename}torcs-ai-tuto-1.md)
* [Next: Brakes and Gears]({filename}torcs-ai-tuto-3.md)



