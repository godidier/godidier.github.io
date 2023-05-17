title: Create Your First TORCS Racing AI Bot - Part 03: Aerodynamics and Stability Controls
date: 2018-12-09 10:00
category: Tutorials
authors: Didier Gohourou
summary: Incorporating aerodynamics and stability for better performance.
slug: torcs-ai-tuto-03
lang: en


This is the fourth chapter of a tutorial for building an AI agent for the racing 
game TORCS. In this chapter, we will learn to break, change gears and drive
faster.

* [Table of Contents]({filename}torcs-ai-tuto-00.md#table_of_contents)

# Aerodynamics

So far, we omitted many variables to simplify computations. Now we will include 
an additional concept that plays a major role on our car performance: aerodynamics. 

Aerodynamics can be defined as the study of the motion of air on a solid body. 
The figure below illustrate different aerodynamic forces applied on a car at 
speed: the drag, the lift and the downforce. There are also some other ones like
the yaw moment (not represented here). We will take into account the downforce 
and the drag in our calculations.

![Aerodynamic Forces]({static}/images/2019-01/aerodynamics.png)

Going in depth in explanation of aerodynamic phenomenon might be a bit out 
of scope of this tutorial. <del>This chapter is actually a bit lenghty right.</del> 
But if you are interested, some good resources for our topic are: 

* [Race car Aerodynamics basics and design](https://www.buildyourownracecar.com/race-car-aerodynamics-basics-and-design/)
* [Aerodynamics](https://www.nas.nasa.gov/About/Education/Racecar/aerodynamics.html)

We will revisit the speed limit in turns and the braking mechanism with 
aerodynamics.

## Accessing the default car setup 

Do you remember the directory where we listed the available cars? Well, within 
each car directory there are various files among which an xml one that defines 
our car's properties, including the setup. In our case, the xml properties file 
is at `/usr/local/share/games/torcs/cars/car1-trb1/car1-trb1.xml`. 
These setup are used by default for our AI agent car.

We will discuss more about managing car setup in coming chapters. For now, we need 
to get some setup from the file, relative to aerodynamics. 

One function we will use to access numerical values in the default setup file is 
`GfParmGetNum`. The sections of interest in `car1-trb1.xml` is the following: 

```xml
	<section name="Aerodynamics">
		<attnum name="Cx" min="0.20" max="2.0" val="0.35"/>
		<attnum name="front area" unit="m2" min="1.0" max="3.0" val="1.92"/>
		<attnum name="front Clift" min="0.0" max="1.5" val="0.69"/>
		<attnum name="rear Clift" min="0.0" max="1.5" val="0.7"/>
	</section>
	
	<section name="Front Wing">
		<attnum name="area" unit="m2" val="0.25"/>
		<attnum name="angle" unit="deg" min="0" max="12" val="6"/>
		<attnum name="xpos" unit="m" val="2.20"/>
		<attnum name="zpos" unit="m" val="0.04"/>
	</section>
	
	<section name="Rear Wing">
		<attnum name="area" unit="m2" min="0" max="1.0" val="0.7"/>
		<attnum name="angle" unit="deg" min="0" max="18" val="14"/>
		<attnum name="xpos" unit="m" min="-2.5" max="-1.0" val="-2.01"/>
		<attnum name="zpos" unit="m" min="0.1" max="1.5" val="0.99"/>
	</section>
```

## Aerodynamics in the speed limitation

Let's include the downforce in the equation we previously used to find the speed 
limit in turns. 

$$
\begin{align*}
\underbrace{\frac{m \cdot v^2}{r}}_{\text{Centrifugal force}}& = 
\underbrace{m \cdot g \cdot \mu}_{\text{Friction force}} + 
\underbrace{C_a \cdot v^2 \cdot \mu}_{\text{Aerodynamic downforce}} \\
v^2& = \frac{g \cdot \mu \cdot r}{1 - r \cdot C_a \cdot \mu / m} \\
v& = \sqrt{\frac{g \cdot \mu \cdot r}{1 - min(1, \frac{r \cdot C_a \cdot \mu}{m})}}
\end{align*}
$$

Where $C_a$ is the downforce coefficient. The downforce coefficient can be 
considered a fixed value computed using the density of the air, the area, shape 
and angle of attack of the car's wing and the ride height which is the distance 
between the ground and the car's bottom. Two things actually contribute to 
the downforce: the car wings which act like an inverted airplane wing to keep
the car on the ground, and the ground effect which is cause by the flow of air 
between the car and the ground. The downforce is a performance advantage on 
cars with wings and a specially designed chasis.

When solving the equation for $v$ there is a risk of getting a negative number 
under the square root (a complex number), in case $r \cdot C_a \cdot \mu / m$ 
becomes greater than 1. To avoid that we use the minimum function to put 
a restriction.

Let's start the implementation with the method that initialize the downforce 
coefficient.

```cpp
/**
 * Initialize the aerodynamic coefficient CA of the car.
 */
void CarController::InitCa()
{
	const float air_density = 1.23;
	float rear_wing_area = GfParmGetNum(car->_carHandle, SECT_REARWING, 
					PRM_WINGAREA, (char*) NULL, 0.0);
	float rear_wing_angle = GfParmGetNum(car->_carHandle, SECT_REARWING, 
					PRM_WINGANGLE, (char*) NULL, 0.0);
	float front_ground_effect = GfParmGetNum(car->_carHandle, SECT_AERODYNAMICS, 
					PRM_FCL, (char*) NULL, 0.0);
	float rear_ground_effect = GfParmGetNum(car->_carHandle, SECT_AERODYNAMICS, 
					PRM_RCL, (char*) NULL, 0.0);

	float front_right_height = GfParmGetNum(car->_carHandle, SECT_FRNTRGTWHEEL, 
					PRM_RIDEHEIGHT, (char*) NULL, 0.20);
	float front_left_height = GfParmGetNum(car->_carHandle, SECT_FRNTLFTWHEEL, 
					PRM_RIDEHEIGHT, (char*) NULL, 0.20);
	float rear_right_height = GfParmGetNum(car->_carHandle, SECT_REARRGTWHEEL, 
					PRM_RIDEHEIGHT, (char*) NULL, 0.20);
	float rear_left_height = GfParmGetNum(car->_carHandle, SECT_REARLFTWHEEL, 
					PRM_RIDEHEIGHT, (char*) NULL, 0.20);

	float wing_ca = air_density * rear_wing_area * sin(rear_wing_angle);
	float Cl = front_ground_effect + rear_ground_effect;

	float ride_height = front_right_height + front_left_height 
			+ rear_right_height + rear_left_height;
	float h = 2.0 * exp( -3.0 * pow(ride_height * 1.5, 4));
	
	CA = h * Cl + 4.0 * wing_ca;

}
```

We can now change the last line of `GetAllowedSpeed(tTrackSeg* segment)` from

```cpp
		return sqrt(mu * G * r);
```

to 

```cpp
		return sqrt((mu * G * r) / (1.0 - MIN(1.0, r * CA * mu / full_car_mass)));
``` 

In order to take into account the aerodynamic downforce when we compute the 
maximum allowed speed in turns.
We modify the `NewRace(...)` method to initialize the mass of the car and the 
the downforce coefficient by calling `InitCa()`.

```cpp
void CarController::NewRace(tCarElt* car, tSituation* s)
{
	this->car = car;
	car_mass = GfParmGetNum(car->_carHandle, SECT_CAR, PRM_MASS, NULL, 1000.0);
	InitCa();
}
```

We also have to update the car mass. We do it in the `CarController::Drive(...)` 
function, just after the `memset(...)` line. 

```cpp
//...
    memset((void *)&car->ctrl, 0, sizeof(tCarCtrl)); // reset the values

	full_car_mass = car_mass + car->_fuel;
	float car_angle = CurrentCarAngle(situation);
//...
```

Now let's add the declaration of the new class members in _carcontroller.h_.

```cpp
	private:
		//...
		void InitCa();

		//...

		float car_mass;
		float full_car_mass;
		float CA;

		//...
```

The graph below show a comparison of the maximum speed that can be reach 
in turns in relation to the radius of the turn,
between the old and the new `GetAllowedSpeed(tTrackSeg* segment)` method.

![Comparison]({static}/#)

### Test drive 

lap time

## Aerodynamics in braking

We will make use of the aerodynamic downforce and the drag in our calculations. 
This will allow us to break later and smoothen the braking pattern when driving. 

All right, the next formula maybe intimidating but, take a deep breath and stay 
with me <del>please</del>... Let's rewrite the energy conservation equation 
we used to compute the braking distance, by including the downforce and the 
drag.



$$
\underbrace{\frac{m \cdot v_1^2}{2}}_{\text{Current kinetic energy}} - 
\underbrace{\frac{m \cdot v_2^2}{2}}_{\text{Desired kinetic energy}} = 
\underbrace{m \cdot g \cdot \mu \cdot s + C_a \cdot \int_{s_2}^{s_1} v^2(s) ds \cdot \mu }_{\text{Energy burned by brake}} +
\underbrace{C_w \cdot \int_{s_1}^{s_2} v^2(s) ds}_{\text{air resistance energy (drag)}}
$$

...Ah, that is enough math for today... So here is the braking distance we get when solving the equation.

$$
s = - \frac{ln((c + v_2^2 \cdot d) / (c + v_1^2 \cdot d))}{2 \cdot d}
$$

With $c = g \cdot \mu$ and $d = (C_a \cdot \mu + C_w) / m$: 


$C_w$ is the drag coefficient. The drag coefficient is computed using ...

Here is the implementation of the initialization of $C_w$

```cpp
/**
 * Initialize the aerodynamic drag coefficient Cw
 */
void CarController::InitCw()
{
	float Cx = GfParmGetNum(car->_carHandle, SECT_AERODYNAMICS, 
					PRM_CX, (char*) NULL, 0.0);
	float front_area = GfParmGetNum(car->_carHandle, SECT_AERODYNAMICS, 
					PRM_FRNTAREA, (char*) NULL, 0.0);
	CW = 0.645 * Cx * front_area;
}
```

Now we change the following line from `GetBrake(...)` 

```cpp
			float brake_distance = 
					(pow(speed, 2) - pow(allowed_speed, 2)) / (2.0 * mu * G);
```

to 

```cpp
			float brake_distance = 
					- log((c + pow(allowed_speed, 2) * d) / (c + pow(speed, 2) * d)) / (2 * d); 

```

We declare the new class members in _carcontroller.h_

```cpp
	private:
		//...
		void InitCw();

		//...
		float CW;
```

and we call `InitCw(...)` at the end of `NewRace(...)`.

```cpp
void CarController::NewRace(tCarElt* car, tSituation* s)
{
	//...
	InitCw();
}
```
[Show performance improvement with a plot]
for integral plotting check
* https://matplotlib.org/gallery/showcase/integral.html
* https://matplotlib.org/api/_as_gen/matplotlib.pyplot.plot.html

### Test drive 

Lap time


# Stability Controls

During the test drive you may have notice the car wheels slide at some points 
(leaving marks on the track). This is due to accelerating and braking too hard. 
We will use some stability control approaches to alleviate this phenomenon.

Stability controls are mechanisms that improve the stability of a car while driving.
There are different stability control systems among which the Antilock Brake System
(ABS) and Traction Control (TCL) that we will implement to assist our AI agent 
driving.

## Antilock Brake System (ABS)

Wheel lock is caused by ...
Anti wheel locking system we will implement will...

implementation 

```cpp
/**
 * Brakes ABS filter
 */
float CarController::FilterABS(float brake)
{
	if (car->_speed_x < ABS_MIN_SPEED)
		return brake;
	float slip = 0.0;
	int i;
	for (i = 0; i < 4; i++){
		slip += car->_wheelSpinVel(i) * car->_wheelRadius(i) / car->_speed_x;
	}
	slip = slip / 4.0;
	if (slip < ABS_SLIP){
		brake = brake * slip;
	}
	return brake;
}
```

We apply the filter by adding the following line at the end of `GetBrake(...)`
just above the return statement,

```cpp
float CarController::GetBrake()
{
	// ...
	brake = FilterABS(brake);
	return brake;
}
```

We also add the new constant to _carcontroller.cpp_

```cpp
const float CarController::ABS_SLIP = 0.9; // [-] range [0.95..0.3]
const float CarController::ABS_MIN_SPEED = 3.0; // [m/s]
```

and we update _carcontroller.h_ with the new class members. 

```cpp
// ...
	public:
		// ...
		static const float ABS_SLIP;
		static const float ABS_MIN_SPEED;

	private:
		// ...
		float FilterABS(float brake);

// ...
```


## Traction Control (TCL)

Wheel slips when ...

Implementation ... 
used a pointer to function 

```cpp
/**
 * Acceleration TCL filter
 */
float CarController::FilterTCL(float acceleration)
{
	if (car->_speed_x < TCL_MIN_SPEED)
		return acceleration;
	float wheel_speed = (this->*GetDrivenWheelSpeed)();
	float slip = car->_speed_x / wheel_speed;
	if (slip < TCL_SLIP) {
		acceleration = 0.0;
	}
	return acceleration;
}
```

Select spinning wheel... function to point to

```cpp
/**
 * Select the right spinning wheel speed calculation for the TCL filter
 */
void CarController::InitDrivenWheelSpeed()
{
	const char* train_type = GfParmGetStr(car->_carHandle, SECT_DRIVETRAIN, 
					PRM_TYPE, VAL_TRANS_RWD);
	if(strcmp(train_type, VAL_TRANS_RWD) == 0) {
		GetDrivenWheelSpeed = &CarController::GetWheelSpeedRWD;	
	} else if (strcmp(train_type, VAL_TRANS_FWD) == 0){
		GetDrivenWheelSpeed = &CarController::GetWheelSpeedFWD;	
	} else if (strcmp(train_type, VAL_TRANS_4WD) == 0){
		GetDrivenWheelSpeed = &CarController::GetWheelSpeed4WD;
	}
}
```
Computing the wheel speed for different train type

```cpp
/**
 * Get the driven wheel speed for Rear Wheel Drive (RWD)
 */
float CarController::GetWheelSpeedRWD()
{
	return (car->_wheelSpinVel(REAR_RGT) + car->_wheelSpinVel(REAR_LFT)) * 
			car->_wheelRadius(REAR_LFT) / 2.0;
}


/**
 * Get the driven wheel speed for Front Wheel Drive (FWD)
 */
float CarController::GetWheelSpeedFWD()
{
	return (car->_wheelSpinVel(FRNT_RGT) + car->_wheelSpinVel(FRNT_LFT)) * 
			car->_wheelRadius(FRNT_LFT) / 2.0;
}

/**
 * Get the driven wheel speed for Four Wheel Drive (4WD)
 */
float CarController::GetWheelSpeed4WD()
{
	return ( (car->_wheelSpinVel(REAR_RGT) + car->_wheelSpinVel(REAR_LFT)) * 
				car->_wheelRadius(REAR_LFT) +
			(car->_wheelSpinVel(FRNT_RGT) + car->_wheelSpinVel(FRNT_LFT)) * 
				car->_wheelRadius(FRNT_LFT) ) / 4.0;
}
```

To apply the TCL filter add the following line at the end of  `GetAcceleration()`
before the `return` statement.

```cpp
float CarController::GetAcceleration()
{
	// ...
	acceleration = FilterTCL(acceleration);
	return acceleration;
}
```

add `InitDrivenWheelSpeed();` in `NewRace(...)` to set the wheel speed 
calculation pointer function to the right one.

```cpp
void CarController::NewRace(tCarElt* car, tSituation* s)
{
	// ...
	InitDrivenWheelSpeed();
}
```

We define the new constant in _carcontroller.cpp_,

```cpp
const float CarController::TCL_SLIP = 0.9; // [-] range [0.95..0.3]
const float CarController::TCL_MIN_SPEED = 3.0; // [m/s]
```

and add the new class member declaration in _carcontroller.h_.

```cpp
// ...
	public: 
		// ...
		static const float TCL_SLIP;
		static const float TCL_MIN_SPEED;
		
	private:
		// ...
		float FilterTCL(float acceleration);
		float GetWheelSpeedRWD();
		float GetWheelSpeedFWD();
		float GetWheelSpeed4WD();
		void InitDrivenWheelSpeed();

		float (CarController::*GetDrivenWheelSpeed)();
// ...
```

### Test Drive

[Video Link]


* [Previous: Brakes and Gears]({filename}torcs-ai-tuto-02.md)
* [Next: Steering and Trajectory]({filename}torcs-ai-tuto-04.md)