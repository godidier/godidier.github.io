title: Create Your First TORCS Racing AI Bot - Part 02: Brakes and Gears
date: 2018-12-09 15:00
category: Tutorials
authors: Didier Gohourou
summary: Increasing the pace by braking and changing gears
slug: torcs-ai-tuto-02
lang: en
status: published


This is the third chapter of a tutorial for building an AI agent for the racing 
game TORCS. In this chapter, we will learn to break, change gears and drive
faster.

* [Table of Contents]({filename}torcs-ai-tuto-00.md#table_of_contents)

# Accelerating the right way

Maybe you are expecting that because we will learn to brake, we can set the 
accelerator to 1.0 whenever we move forward right?! I am afraid we can't !
We cannot go full speed on every section of a track. If our car is on a straight 
line then we can do whatever we want to the accelerator, meaning  we can just 
set it to 1.0 (<del>Your dream is realized!</del>). But on a turn section, 
there is a limited amount of speed we have to respect. Otherwise, we migth 
**understeer** and go off road.

Understeer?... Here is a quote from 
[Wikipedia](https://en.wikipedia.org/wiki/Understeer_and_oversteer) to the rescue.
> Understeer and oversteer are vehicle dynamics terms used to describe the 
> sensitivity of a vehicle to steering. Oversteer is what occurs when a car turns 
> (steers) by more than the amount commanded by the driver. Conversely, understeer 
> is what occurs when a car steers less than the amount commanded by the driver. 

![Understeer and Oversteer]({static}/images/2018-12/understeer-oversteer.png)

So we need to know what amount of speed we can apply to safely pass a turn.
We will resort to an equation from physics. <del>Stay with me</del>. 

$$ {mv^2 \over r} = {mg\mu} $$

Where: 

* $m$ : is the current car mass (kg)
* $v$ : current speed (m/s)
* $r$ : radius of the turn's segment(m) 
* $\mu$ : friction coefficient
* $g$ : the gravity of earth 9.80665 (m/s^2)

The left part of the equation is the **centrifugal force** and the left side is 
the **centripetal force** created by the friction of the turning wheels of our 
car. 
Here is a quote from an 
[article](https://www.diffen.com/difference/Centrifugal_Force_vs_Centripetal_Force) 
that explain those two forces. 
> Centrifugal force (Latin for "center fleeing") describes the tendency of an 
> object following a curved path to fly outwards, away from the center of the 
> curve. It's not really a force; it results from inertia — the tendency of an 
> object to resist any change in its state of rest or motion. Centripetal force 
> is a real force that counteracts the centrifugal force and prevents the object 
> from "flying out," keeping it moving instead with a uniform speed along a 
> circular path. 

Let's picture what we are talking about.

![Centrifugal and Centripetal forces]({static}/images/2018-12/centrifugal-centripetal.png)

We find the appropriate speed by expressing $v$ from the equation above, which 
yield:

$$ v = \sqrt{g \mu r} $$

This is a simplified version of the real appropriate speed value. We assume that 
all wheels have the same $\mu$, and the weight is equally distributed over all 
wheels. The coefficient $\mu$ with the friction coefficient which normally is 
a function with many parameters includind the relative speed vector between the  
surfaces, the current load, the shape of the patch, the temperature, etc.

With our result, let's create a function that returns the appropriate (allowed) 
speed on a segment of the track. We declare it in _carcontroller.h_ within the 
`CarController` class interface. 

```cpp
float  CarController::GetAllowedSpeed(tTrackSeg* segment);
```

and define it in _carcontroller.cpp_. 

```cpp
/**
 * Compute the allowed speed on a segment.
 * */
float  CarController::GetAllowedSpeed(tTrackSeg* segment)
{
	if (segment->type == TR_STR){
		return FLT_MAX; // Max float number 
	} else {
		float mu = segment->surface->kFriction;
		float r = segment->radius;
		return sqrt(mu * G * r);
	}
}
```
`TR_STR` is a straight segment, there is also `TR_RGT` for a right curve and 
`TR_LFT` for a left curve, from _track.h_.


To prevent the car from going over the allowed speed of a given segment, we need 
to know how much throttle to apply on the acceleretor pedal. In TORCS the 
acceleration throttle value is between 0 and 1, and can be deduced by 
normalizing the car engine's given revolution per minute (rpm) by its rpm redline
i.e the maximum revolution per minute the car's engine can operate at.

$$
accelerator\ throttle = \frac{rpm}{rpm\ redline} 
$$

If we also express the angular speed of the wheel $\omega = \theta / t$ 
with the linear speed $v = \omega \times \theta$ we have $\omega = v / r$
Since the angular speed is also equal to the rpm divided by the gear ratio, 
we have the following relation: 

$$
\begin{align*}
\frac{v}{r}& = \frac{rpm}{gear\ ratio} \\
rpm& = \frac{v \times gear\ ratio}{r} 
\end{align*}
$$

Where 

* $v$ is the speed of the car (m/s)
* $r$ is the radius of the spinning wheel (m)
* $rpm$ ie the engine's revolution per minute (rad/s)
* $gear\ ratio$ the ratio of the angular velocity fo input gear (engine's rpm) 
and the angular velocity output gear (the wheel). 

We obtain the desired accelerator throttle by plugin the expression of $rpm$ 
into the one of the $accelerator\ throttle$. 

$$
accelerator\ throttle = \frac{(v \times gear\ ratio) / r}{rpm\ redline}
$$

We rewrite the `GetAcceleration()` method with the result. It basically says 
if with our current speed we can accelerate to the max without stepping over 
the allowed speed of the segment, we go full throttle. Else, we compute the 
right amount of  of throttle to apply on the accelerator pedal.

```cpp
float CarController::GetAcceleration()
{
	float acceleration = 0.0;
	float allowed_speed = GetAllowedSpeed(car->_trkPos.seg);
	if (allowed_speed > car->_speed_x + ACCELERATION_MARGIN){
		acceleration = 1.0;
	} else {
		float gear_ratio = car->_gearRatio[car->_gear + car->_gearOffset];
		float rpm_redline = car->_enginerpmRedLine;
		float wheel_radius = car->_wheelRadius(REAR_RGT);
		acceleration = (allowed_speed * gear_ratio / wheel_radius) / rpm_redline;
	}
	return acceleration;
}
```
We finish this part by defining the constants G (gravity) and FULL\_THROTTLE 
in _carcontroller.cpp_.

```cpp
const float CarController::G = 9.81; // [m/s^2]
const float CarController::ACCELERATION_MARGIN = 1.0; // [m/s]
```

and we define them in _carcontroller.h_ among the public element of the 
`CarController` class.

```cpp
		static const float G;
		static const float ACCELERATION_MARGIN;
```

# Braking Basics

## Checking the distance

As we previously saw, there is a speed limitation for turn segments. If we 
approach a turn segment at a speed higher than the allowed speed of that turn 
segment, we need to reduce our speed to the given allowed speed, before engaging 
that turn segment. For that, we need to brake. 

We need to know in advance when to apply the brakes. i.e. at what distance from the 
turn we have to start braking. We can compute the braking distance using the 
following equation of energies. 

$$
\underbrace{\frac{m \cdot v_1^2}{2}}_{\text{Current kinetic energy}} - \underbrace{\frac{m \cdot v_2^2}{2}}_{\text{Desired kinetic energy}} = \underbrace{m \cdot g \cdot \mu \cdot s}_{\text{Energy 'burned' by the brakes over distance s}}
$$

Where 

* $m$ is the mass of the car (kg)
* $v_1$ is the current speed of the car (m/s)
* $v_2$ is the allowed speed on the turn segment, or targeted portion of the track
 (m/s)
* $g$ is the gravity constant
* $\mu$ is the friction coefficient 
* $s$ is the distance necessary to brake (m)

We have 

$$
s = \frac{v_1^2 - v_2^2}{2 \cdot g \cdot \mu}
$$

We also need a mechanism to gradually look ahead from the current position of 
the car to a potential segment with an allowed speed lower than the current speed 
of the car, up to a given distance. The lookahead mechanism will start with the 
distance from the car to the end of the current segment. Let's implement a utility 
function to get this initial distance. 

in _carcontroller.cpp_ add

```cpp
/**
 * Measure the distance from the current position of the car 
 * to the end of the segment the car is on.
 */
float CarController::DistanceFromCarToSegmentEnd()
{
	auto car_position = car->_trkPos;
	auto segment = car_position.seg;
	if (segment->type == TR_STR){
		return segment->length - car_position.toStart;
	} else {
		return (segment->arc - car_position.toStart) * segment->radius;
	}
}
```
As you might see, if the segement is a straight, we simply do a substraction else 
the arc of the radius comes into play.

Note that the maximum distance up to which we check for a turn to slow down is 
a special case of the distance formula $s$ where $v_2 = 0$. Meaning in case 
we have to completely stop the car. This yield $s = v_1^2 / (2 \cdot g \cdot \mu)$

The illustration below gives an idea of the 'look ahead' mechanism. 

![Look ahead]({static}/#)

## Braking

We will now implement our breaking mechanism. In case we are on a segment with a 
speed higher than the allowed speed on that segment, we brake. Otherwise we first 
consider a maximum distance up to which we check for turns that requires to 
decrease our speed. Then we gradually 'look ahead' up to that maximum distance 
for potential turns for which we need to slow down. If we find one, we check 
if the required distance we need to brake to match the allowed speed of that 
turn is greater than the distance we currently are (looking ahead) to that turn. 
If so we apply the brake. For every other conditions, we do not brake.


Here is the implementation of basic braking in _carcontroller.cpp_

```cpp
float CarController::GetBrake()
{
	float brake = 0.0;
	tTrackSeg* segment = car->_trkPos.seg;
	float speed = car->_speed_x;
	float allowed_speed = GetAllowedSpeed(segment);
	float mu = segment->surface->kFriction;
	float look_ahead_distance_max = pow(speed, 2) / (2.0 * mu * G);
	float look_ahead_distance = DistanceFromCarToSegmentEnd();

	if (allowed_speed < speed) {
		brake = 1.0;
	}
	segment = segment->next;
	while(look_ahead_distance < look_ahead_distance_max){
		allowed_speed = GetAllowedSpeed(segment);
		if (allowed_speed < speed) {
			float brake_distance = 
					(pow(speed, 2) - pow(allowed_speed, 2)) / (2.0 * mu * G);
			if (brake_distance > look_ahead_distance){
				brake = 1.0;
				break;
			}
		} 
		look_ahead_distance += segment->length;
		segment = segment->next;
	}
	return brake;
}
```

## Updating related methods

Let's make an update in the `Drive` method such that we accelerate only if we 
are not braking.

```cpp
void CarController::Drive(tSituation* situation)
{
    memset((void *)&car->ctrl, 0, sizeof(tCarCtrl)); // reset the values

	float car_angle = CurrentCarAngle(situation);

    car->_steerCmd = GetSteering(car_angle);
    car->_gearCmd = GetGear(); 
    car->_brakeCmd = GetBrake(); 
	if (car->_brakeCmd == 0.0){
    	car->_accelCmd = GetAcceleration(); 
	} else {
		car->_accelCmd = 0.0;
	}
}
```

Let's fix the gear to 4, for a test drive.

```cpp
float CarController::GetGear()
{
	return 4; 
}
```

Now let's declare the newly defined method within the class declaration in 
_carcontroller.h_.

```cpp
	float DistanceFromCarToSegmentEnd();
```

You can compile and do a test drive.


# Shifting Gears

We will implement a simple yet efficient gear shifting mecanism. 

For now we are only moving forward, so if we are in neutral or reverse, 
we shift to the 1st gear. Otherwise we compute the maximum speed allowed 
at the current gear we are at, by multiplying the angular speed of the 
engine by the wheel radius and a 'shift constant' which represent a percentage 
of the rpm red line. If the product is less than the car speed, we shift up. 
Else, we check if we can shift down by first ensuring we are at a gear higher 
than 1. We then compute the maximum speed allowed at the gear lower than the 
current one. If the maximum speed allowed at the lower gear is lower than our 
current speed, we down shift. Note that we added a shift margin to prevent 
oscillating the gear up and down.

Here is the code for the shifting mechanism, implemented in `GetGear()`

```cpp
float CarController::GetGear()
{
	if (car->_gear <= 0)
		return 1.0;
	float gear_ratio_up = car->_gearRatio[car->_gear + car->_gearOffset];
	float omega = car->_enginerpmRedLine / gear_ratio_up;
	float wheel_radius = car->_wheelRadius(REAR_RGT);
	if (omega * wheel_radius * SHIFT < car->_speed_x){
		return car->_gear + 1;
	} else if (car->_gear > 1) {
		float gear_ratio_down = car->_gearRatio[car->_gear + car->_gearOffset - 1];
		omega = car->_enginerpmRedLine / gear_ratio_down;
		if (omega * wheel_radius * SHIFT < car->_speed_x + SHIFT_MARGIN) {
			return car->_gear - 1;
		}
	}
	return car->_gear; 
}
```

Let's define the new constants at the begining of _carcontroller.cpp_
```cpp
const float CarController::SHIFT = 0.9; // [-] (% of rpm red line)
const float CarController::SHIFT_MARGIN = 4.0; // [m/s]
```
And declare them in _carcontroller.h_
```cpp
		static const float SHIFT;
		static const float SHIFT_MARGIN;
```

Let's end the chapter with a test drive of the result.

<iframe width="560" height="315" src="https://www.youtube.com/embed/9mTA9uM4tw0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Next we will take the aerodynamics into account.

* [Previous: Acceleration]({filename}torcs-ai-tuto-01.md)
* [Next: Aerodynamics and Stability Controls]({filename}torcs-ai-tuto-03.md)