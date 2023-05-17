title: Create Your First TORCS Racing AI Bot - Part 05: Recovering
date: 2018-12-11 15:00
category: Tutorials
authors: Didier Gohourou
summary: Getting back on track after getting off track or getting stuck.
slug: torcs-ai-tuto-05
lang: en


* [Table of Contents]({filename}torcs-ai-tuto-00.md#table_of_contents)

<p id="introduction"></p>
# Introduction

So far, our agent only knows how to drive forward. But if something happen (like a 
crash with another car during a race), and the agent go off-course to hit 
the wall for example, it will still try to move forward and be _stuck_!

In this chapter we will come up with an approach to recognize that our 
agent is off-course and implement a mechanism to recover from such incident.

<p id="basic_recovery"></p>
# A Basic Recovery Strategy

Basically if our car is stuck, we recover, else, we drive normally.
We will consider that our car is stuck, if the angle of the car relative to the 
track is greater than a certain value during a certain period of time. 
We will start with 30 degrees as our angle limit, and 100 the number of 
simulation steps up to which we will consider the car stuck.
Since the simulation time is 0.02 seconds, the `Drive(...)` method is 
executed every 0.02 seconds. This means that our car needs to form an angle 
with an absolute value greater than 30 degrees relatively to the track, 
for 2 seconds (or more) before we trigger the recovering process.

Let's start by defining those new constants in _carcontroller.h_.

```cpp
// ...
    public: 
        // ...
        static const float MAX_UNSTUCK_ANGLE;
        static const float UNSTUCK_TIME_LIMIT;
    
    private: 
        // ...
        int MAX_UNSTUCK_COUNT;

        int stuck_counter;
// ...
```
At the beginning of _carcontroller.cpp_ we initialize `MAX_UNSTUCK_ANGLE` and 
`UNSTUCK_TIME_LIMIT`.

```cpp
// ...
const float Driver::MAX_UNSTUCK_ANGLE = 30.0/180.0*PI;  /* [radians] */
const float Driver::UNSTUCK_TIME_LIMIT = 2.0;           /* [s] */
```

We also initialise `MAX_UNSTUCK_COUNT` and `stuck_counter` in the 
`newRace(...)` method. Note that `MAX_UNSTUCK_COUNT` stays constant after 
this first initialization.

```cpp
void CarController::NewRace(tCarElt* car, tSituation* s)
{
    // ...
    MAX_UNSTUCK_COUNT = int(UNSTUCK_TIME_LIMIT/RCM_MAX_DT_ROBOTS);
    stuck_counter = 0;
    // ...
}
```
WTF is this `RCM_MAX_DT_ROBOTS` thing ? It is a constant defined in 
_$TORCS_BASE/src/interfaces/raceman.h_ that represents the mean robot update
rate that is equal to 0.02 second, So `MAX_UNSTUCK_COUNT` will be 100.

Then in _carcontroller.cpp_ we write a method that tells us if our car is 
stuck or not. The implementation steps are quite simple:

1. Initialize a stuck counter to 0
2. If the absolute value of the angle between the car and the track is smaller 
than a certain value `MAX_UNSTUCK_ANGLE`, reset the counter to 0 and return false 
(Meaning "not stuck")
3. If the angle is smaller than a certain limit (100), increase the counter and 
return false (for "not stuck"). Otherwise return true (for "stuck").


```cpp
/**
 * Check if the car is off course (stuck)
 */
 bool CarController::IsStuck(){
	float angle = RtTrackSideTgAngleL(&(car->_trkPos)) - car->_yaw;
	angle = remainder(angle, 2*PI);
	if(fabs(angle) < MAX_UNSTUCK_ANGLE){
		stuck_counter = 0;
		return false;
	}
	if (stuck_counter < 100){
		stuck_counter++;
		return false;
	} else {
		return true;
	}
}
```

As previously explained, this method will be used in our driving strategy. 
If the car is stuck we recover by counter steering and going back (until
the car is no more stuck).

```cpp
void CarController::Drive(tSituation* situation)
{
    memset((void *)&car->ctrl, 0, sizeof(tCarCtrl)); // reset the values

	full_car_mass = car_mass + car->_fuel;
	float car_angle = CurrentCarAngle(situation);

	if(IsStuck()){
		car->_steerCmd = - car_angle;
		car->_gear = -1; // reverse
		car->_accelCmd = 0.3; // 30% acceleration
		car->_brakeCmd = 0.0; // no brakes
	} else {
		car->_steerCmd = GetSteering(car_angle);
		car->_gearCmd = GetGear(); 
		car->_brakeCmd = GetBrake(); 
		if (car->_brakeCmd == 0.0){
 			car->_accelCmd = GetAcceleration(); 
		} else {
			car->_accelCmd = 0.0;
		}
	}
}

```

[compile and test ? how to check? collision ?]


Below is a illustration to help us get our head wrapped around a situation of 
getting stuck.

![Getting Stuck 1]({static}/images/2019-11/getting-stuck-1.svg)

This approach is simple and get the job done in most case. Yet, there is an 
edge case we have to take into account.

<p id="better_recovery"></p>
# A Better Recovery Strategy

Consider the situation where the car is oversteering in a turn and the angle 
between the track tangent and the car increase. At a certain point the angle 
becomes big enough (> 30 degree), and start incrementing the 'stuck' counter. 
If the 'stuck' counter gets to 100 and the car hits the wall with its back, 
we will try to get unstuck. Since the back of the car is against the wall, and 
the unstuck method is the back up, we will stay stuck! Having a close look at the 
figure below representing that situation, we don't need to get 'unstuck' i.e 
back off is such situation. We can simply drive forward to come back on the track.
This is mainly because the front of our car point toward the middle of the track 
(or looks inside the track). 

![Getting Stuck 2]({static}/images/2019-11/getting-stuck-2.svg)

To detect if the car points to the middle of the track or not, we will use 
the angle formed by the car and the tangent to the track, and the car 
position relatively to the middle of the track. The angle formed by the car 
direction and the tangent of the track is positive when it goes clockwise, and 
negative when it goes counter-clockwise. The relative position of the car to 
the middle of the track is represented by `car->_trkPos.toMiddle`, which is 
positive on the left side of the track and negative on the right side. We can 
conclude that if the 'angle fo the car' times the 'position to the middle' is 
positive, we are looking toward the middle of the track. The figure below 
illustrate the different values used as criteria and the case where the car 
(in red) look outside the track and the car (in blue) looks inside the track.

![Look inside criteria]({static}/images/2019-11/look-inside.svg)

We will just drive forward if `fabs(angle) < MAX_UNSTUCK_ANGLE`. Else, we are 
looking outward and it the `stuck_counter` is already big enough, `IsStuck()` 
returns true. Otherwise If `stuck_counter` is less, we increment it.

To trigger the unstuck mechanism more safely, we will add a condition on the 
speed to be below a certain threshold. 


Let's start the implementation. First, let's use some constants to hold the 
[...] speed and the [...] distance. 

```cpp
    public:
        // ...
        static const float MAX_UNSTUCK_SPEED;
        static const float MIN_UNSTUCK_DIST;
```

We initialize the constants at the top of _carcontroller.cpp_

```cpp
const float Driver::MAX_UNSTUCK_SPEED = 5.0;   /* [m/s] */
const float Driver::MIN_UNSTUCK_DIST = 3.0;    /* [m] */
```

Let's make the modificaitons in `IsStuck(...)` 

```cpp
bool CarController::IsStuck(){
	float angle = RtTrackSideTgAngleL(&(car->_trkPos)) - car->_yaw;
	angle = remainder(angle, 2*PI);
	if(fabs(angle) > MAX_UNSTUCK_ANGLE && 
		car->_speed_x < MAX_UNSTUCK_SPEED && 
		fabs(car->_trkPos.toMiddle) > MIN_UNSTUCK_DIST){
			if(stuck_counter > MAX_UNSTUCK_COUNT && car->_trkPos.toMiddle*angle < 0.0){
				return true;
			} else {
				stuck_counter++; 
				return false;
			}
	} else {
		stuck_counter = 0;
		return false;
	}
}
```

Let's increase the reverse throttle value, in the `Drive` method a bit.

```cpp
//...
    if (IsStuck(car)) {
        car->ctrl.steer = -angle / car->_steerLock;
        car->ctrl.gear = -1; // reverse gear
        car->ctrl.accelCmd = 0.5; // 50% accelerator pedal
        car->ctrl.brakeCmd = 0.0; // no brakes
    } else {
// ...
```

[Download and Test drive]

* [Previous: Steering and Trajectory]({filename}torcs-ai-tuto-04.md)
* [Next: Collision avoidance and Overtaking]({filename}torcs-ai-tuto-06.md)