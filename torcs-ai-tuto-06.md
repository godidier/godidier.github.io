title: Create Your First TORCS Racing AI Bot - Part 06: Collision avoidance and Overtaking
date: 2018-12-12 15:00
category: Tutorials
authors: Didier Gohourou
summary: Taking into account opponents in a race.
slug: torcs-ai-tuto-06
lang: en



* [Table of Contents]({filename}torcs-ai-tuto-00.md#table_of_contents)

This chapter will describe an approach to implement avoidance and overtaking 
of opponents in a race. There are plenty of ways to do it. Here we will 
present a simple one, meaning you are free to improve it or rethink the 
whole thing.

We start by collecting race information about our opponents, and use that 
information to avoid them and/or overtake them.

# Collecting Opponent's data

Let's create an opponent class that will hold data for a given opponent 
relative to our AI agent (eg. the distance between that opponent and our AI 
agent). For simplicity we will that class will also hold data which depends 
only on a given opponent, and can be computed independently of our AI agent
(eg. the speed of a given opponent). 

Our approach is a bit inefficient because in future chapters, as we will 
learn to run multiple instances of our robot, base on the same code called 
module, some of those variables that are computed independently of our AI 
agent will be recomputed unecessarely by each instance of our robot module.
It will be better to put such calculation in an other class visible by 
all the AI agents of our module. Anyway this is no time to ponder on this 
problem. We didn't even talked about this module thing right?! This is for 
coming chapters. 


Create a header file named _opponent.h_ that will contain the declaration 
of the `Opponent` class we just talked about and of the `Opponents` class 
that is hold a collection of `Opponent`s.

## The Opponent Class

```cpp
#ifndef _OPPONENT_H_
#define _OPPONENT_H_

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

#include "linalg/vector2d.h"
#include "carcontroller.h"

#define OPP_IGNORE  0
#define OPP_FRONT	(1<<0)
#define OPP_BACK	(1<<1)
#define OPP_SIDE	(1<<2)
#define OPP_COLL	(1<<3)

class CarController;

/**
 * Class encapsulating data of our AI agent opponents.
 */
class Opponent {
    public:
        Opponent();

        void SetCar(tCarElt *car) { this->car = car; }
        static void SetTrack(tTrack *track) { Opponent::track = track; }

        static float GetSpeed(tCarElt *car);
        tCarElt *GetCar() { return car; }
        int GetState() { return state; }
        float GetCatchDistance() { return catch_distance; }
        float GetDistance() { return distance; }
        float GetSideDistance() { return side_distance; }
        float GetWidth() { return width; }
        float GetSpeed() { return speed; }

        void Update(tSituation *s, CarController *car_controller);

    private:
        float GetDistanceToSegmentStart();

        tCarElt *car;       /* pointer to the opponents car */
        float distance;     /* approximation of the real distance */
        float speed;        /* speed in direction of the track */
        float catch_distance;    /* distance needed to catch the opponent */
        float width;        /* the cars needed width on the track */
        float side_distance;     /* distance of center of gravity of the cars */
        int state;          /* state bitmask of the opponent */

        /* class variables */
        static tTrack *track;

        /* constants */
        static float FRONT_COLLDIST;
        static float BACK_COLLDIST;
        static float SIDE_COLLDIST;
        static float LENGTH_MARGIN;
        static float WIDTH_MARGIN;
};


/** 
 * Holds an array of Opponents our AI agent is racing against
 **/
class Opponents {
    public:
        Opponents(tSituation *s, CarController *car_controller);
        ~Opponents();

        void Update(tSituation *s, CarController *car_controller);
        Opponent *GetOpponentPtr() { return opponent; }
        int Count() { return count; }

    private:
        Opponent *opponent;
        int count;
};

#endif _OPPONENT_H_
```

The code start by including the necessary header, then define bitmasks 
aliases that will be useful to classify opponents. There is also a 
`CarController` class prototype declaration, because of the cross dependency 
[Of what? be more assertive]. 
After that we can declare the `Opponent` class, that mainly contains values 
and 'getters' of the information we want to hold.
Finally, we declare the `Opponents` class that hold an array of `Opponent`s 
characterized by and `Opponent` pointer (the address of the array), and the 
number of opponents we are dealing with.


Now create a file named _opponent.cpp_ where we will define the symbols we just 
declared.

We start by including the header and define the class variables and constants. 

```cpp
#include "opponent.h"

tTrack* Opponent::track;
float Opponent::FRONT_COLLDIST = 200.0;  /* [m] distance to check for other cars */
float Opponent::BACK_COLLDIST = 50.0;    /* [m] distance to check for other cars */
float Opponent::LENGTH_MARGIN = 2.0;    /* [m] safety margin */
float Opponent::WIDTH_MARGIN = 1.0;      /* [m] safety margin */
```

`FRONT_COLLDIST` and `BACK_COLLDIST` respectively define distances from which 
our AI agent will start checking for collision at the front and at the back.
`LENGTH_MARGIN` and `WIDTH_MARGIN` are safety margins respectively on the 
length and width axis of our AI agent's car as minimal distance we would like 
to maintain to our opponents. You can go further and put those parameters in 
an XML setup file and load them at startup (...more on loading setups in 
coming chapters). 

The following `Opponent::GetSpeed(tCarElt *car)` method computes the speed of 
the car in the direction of the track. We first get the direction of the track 
at the cars location. After that, we compose the speed vector of the car. 
We use `_speed_X` and `_speed_Y` here, the uppercase X and Y means that the 
speed vector is in global coordinates, the lowercase x and y are the speeds 
in the _"instantaneously coincident frame of reference"_, a coordinate system 
that is oriented like the cars body but doesn't move with the car (if it 
would move with the car the `_speed_x` and `_speed_y` would always be zero). 
From the trackangle we compute a direction vector of the track 
(its length is one). Finally we compute the dot product of the speed and the 
direction, which will give us the speed in the direction of the track. 

```cpp
/**
 * Compute speed component parallel to the track 
 **/
float Opponent::GetSpeed(tCarElt *car)
{
    Vector2D speed, direction;
    float track_angle = RtTrackSideTgAngleL(&(car->_trkPos));

    speed.x = car->_speed_X;
    speed.y = car->_speed_Y;
    direction.x = cos(track_angle);
    direction.y = sin(track_angle);
    return speed * direction;
}
```

The next method `Opponent::getDistToSegStart()`  computes the distance of the 
opponent to the segments start. The reason is that `car->_trkPos.toStart` 
contains the length of the segment for straight segments and for turns the 
arc, so we need a conversion to the length.

```cpp
/** 
 * Compute the length to the start of the segment 
 **/
float Opponent::GetDistanceToSegmentStart()
{
    if (car->_trkPos.seg->type == TR_STR) {
        return car->_trkPos.toStart;
    } else {
        return car->_trkPos.toStart * car->_trkPos.seg->radius;
    }
}
```

Now we will have a look at the 
`Opponent::Update(tSituation *s, CarController *car_controller)` 
function that is responsible for the update of the values in the Opponent 
instances.  

First we get a pointer to our AI agent controller, that we name `agent`. 
We set the initial state to ignore the current opponent. We then check if 
the opponent's car is not longer part of the simulation. We do that using 
`RM_CAR_STATE_NO_SIMU` which is a car state flag defined in 
_$TORCS\_BASE/src/interfaces/car.h_, that indicate if a car is simulated 
or not.

Next, we compute the distance of the center of the agent car (our car), to the 
center of the current opponent car. We achieve that by computing the distances 
to the start line and taking the difference. Our calculation is approximating 
the real distance. For a more accurate value, we have to compute it with the 
cars corners (look up car.h). The `if` part is to "normalize" the distance. 
Note that from the position of our agent up to a the half track length, the 
opponent is in front of us (`distance` is positive). Otherwise, the opponent 
is behind (`distance` is negative). A detail is that we can't use 
`_distFromStartLine`, an alias of `race.distFromStartLine` in car.h of the 
opponent, because it is a private field [??]. 

We update the speed with the previously introduced `GetSpeed()` method. Then we 
compute the width of the opponents car on the track (think the car is turned 90 
degrees on the track, then the needed "width" is its length). 

Then, we check if the opponent is in the range we defined as relevant. After 
that, the classification of the opponent starts. In this part we check if it is 
in front of us and slower. If that is the case we compute the distance we need 
to drive to catch the opponent (catchdist, we assume the speeds are constant) 
and set the opponents flag `OPP_FRONT`. Because the "distance" contains the 
value of the cars centers we need to subtract a car length and the safety 
margin. At the end we check if we could collide with to opponent. If yes, 
we set the flag `OPP_COLL`. 

Here we check if the opponent is behind us and faster. We won't use that 
in the tutorial robot, but you will need it if you want to let overlap 
faster opponents. 

This part is responsible to check if the opponent is aside of us. If yes,
we compute the distance (sideways) and set the flag `OPP_SIDE`. 

```cpp
/** 
 * Update the values in Opponent this 
 **/
void Opponent::Update(tSituation *s, CarController *car_controller)
{
    tCarElt *agent = car_controller->GetCar();

    /* init state of opponent to ignore */
    state = OPP_IGNORE;

    /* if the car is out of the simulation ignore it */
    if (car->_state & RM_CAR_STATE_NO_SIMU) {
        return;
    }

    /* updating distance along the middle */
    float oppToStart = car->_trkPos.seg->lgfromstart + GetDistanceToSegmentStart();
    distance = oppToStart - agent->_distFromStartLine;
    if (distance > track->length/2.0) {
        distance -= track->length;
    } else if (distance < -track->length/2.0) {
        distance += track->length;
    }

    /* update speed in track direction */
    speed = Opponent::GetSpeed(car);
    float cosa = speed/sqrt(car->_speed_X*car->_speed_X + car->_speed_Y*car->_speed_Y);
    float alpha = acos(cosa);
    width = car->_dimension_x*sin(alpha) + car->_dimension_y*cosa;
    float SIDECOLLDIST = MIN(car->_dimension_x, agent->_dimension_x);

    /* is opponent in relevant range -50..200 m */
    if (-BACK_COLLDIST < distance && distance < FRONT_COLLDIST) {
        /* is opponent in front and slower */
        if (distance > SIDECOLLDIST && speed < car_controller->GetSpeed()) {
            catch_distance = car_controller->GetSpeed() * distance/ (car_controller->GetSpeed() - speed);
            state |= OPP_FRONT;
            distance -= MAX(car->_dimension_x, agent->_dimension_x);
            distance -= LENGTH_MARGIN;
            float cardist = car->_trkPos.toMiddle - agent->_trkPos.toMiddle;
            side_distance = cardist;
            cardist = fabs(cardist) - fabs(width/2.0) - agent->_dimension_y/2.0;
            if (cardist < WIDTH_MARGIN) state |= OPP_COLL;
        } else if (distance < -SIDECOLLDIST && speed > car_controller->GetSpeed()) {/* is opponent behind and faster */
            catch_distance = car_controller->GetSpeed()*distance/(speed - car_controller->GetSpeed());
            state |= OPP_BACK;
            distance -= MAX(car->_dimension_x, agent->_dimension_x);
            distance -= LENGTH_MARGIN;
        } else if (distance > -SIDECOLLDIST && distance < SIDECOLLDIST) {/* is opponent aside */
            side_distance = car->_trkPos.toMiddle - agent->_trkPos.toMiddle;
            state |= OPP_SIDE;
        }
    }
}
```

Now the implementation of the `Opponents` class methods.

First the constructor. The constructor allocates memory and generates the 
Opponent instances. The destructor deletes the instances and frees the memory. 

```cpp
/** 
 * Initialize the list of opponents 
 **/
Opponents::Opponents(tSituation *s, CarController *car_controller)
{
    opponent = new Opponent[s->_ncars - 1];
    int i, j = 0;
    for (i = 0; i < s->_ncars; i++) {
        if (s->cars[i] != car_controller->GetCar()) {
            opponent[j].SetCar(s->cars[i]);
            j++;
        }
    }
    Opponent::SetTrack(car_controller->GetTrack());
    count = s->_ncars - 1;
}
```

Then the `Opponents::Update`

```cpp
Opponents::~Opponents() 
{
    delete [] opponent;
}
```

## Modifying our AI agent controller

Let's add the missing methods to access some data of our car controller 
class from the opponent class.

```cpp
/**
 *  Updates all the Opponent instances 
 **/
void Opponents::Update(tSituation *s, CarController *car_controller)
{
    int i;
    for (i = 0; i < s->_ncars - 1; i++) {
        opponent[i].Update(s, car_controller);
    }
}
```

## Updating the Makefile

Add the `Opponent` class to the compilation configuration by adding 
'_opponent.cpp_' to the `SOURCES` variables in the _Makefile_

```
SOURCES     = ${ROBOT}.cpp carcontroller.cpp opponent.cpp
```

[Test Drive + Video ?]

# Collision Avoidance

Our agent will handle two cases of collision avoidance: front collision and 
side collision. Diffent commands are applied to handle each one of them. 
In this section we implement two basic approaches to avoid collisions 
according to the situation (the type of collision we want to avoid).

## Avoiding Front Collision 

A front collision is the case where the front part of our AI agent's car hits 
an opponent's car at it's back. We will avoid this type of collision by 
adding an additional filter to our braking mechanism.

First, let's add the necessary declaration to our `CarController` class. 
We include the _opponent.h_ header and add the forward-declaration of 
the `Opponent` and `Opponents` classes. We also add a destructor, 
declare the braking filter method in the public section of the class, and 
an `Opponents` and `Opponent` pointers to hold the list of opponents and 
the current opponent we will analyze at a given time.

```cpp
//... Other includes
#include "opponent.h"

//...
class Opponents;
class Opponent;

class CarController {

    public: 
        // ...
        ~CarController();

    private:
        //...
        float FilterBrakeCollision(float brake);
        //...
        Opponents *opponents;
        Opponent opponent;
}
```

Now to the implementation in _carcontroller.cpp_. We define the destructor.

```cpp
CarController::~CarController(){
	delete opponents;
}
```

Instantiate the `Opponent`(s) classes in `NewRace` method

```cpp
void CarController::NewRace(tCarElt* car, tSituation* s)
{
	opponents = new Opponents(s, this);
	opponent = opponents->GetOpponentPtr();
    //...
}
```

Let's implement the filter method. In a nutshell it iterates through the 
opponents and checks for every opponent which has been tagged with `OPP_COLL` 
if we need to brake to avoid a collision. If yes we brake full and leave 
the method. 

We set up some variables and then we enter the loop to iterate through all 
opponents. The presented solution is not optimal. You could replace the call to 
`GetDistance()` with `GetChatchDist()`, but additional code is necessary to make 
it work. The computation of the brakedistance should be familiar from 
previous chapters. 

```cpp
/** 
 * Filter the brake to avoid a front collision
 **/
float CarController::FilterBrakeCollision(float brake){
	float current_speed_sqr = car->_speed_x * car->_speed_x;
	float mu = car->_trkPos.seg->surface->kFriction;
	float cm = mu * G * full_car_mass;
	float ca = CA * mu * CW;
	int i;
	for (i = 0; opponents->Count(); i++){
		if (opponent[i].GetState() & OPP_COLL ){
			float allowed_speed_sqr = pow(opponent[i].GetSpeed(), 2);
			float brake_distance = full_car_mass * (current_speed_sqr - allowed_speed_sqr) 
				/ (2.0 * (cm + allowed_speed_sqr * ca)); 
				if(brake_distance > opponent[i].GetDistance()){
					return 1.0;
				}
		}
	}
	return brake;
}
```

Now we can call it in the `GetBrake` method.

```cpp
float CarController::GetBrake()
{
    // ...
	brake = FilterABS(FilterBrakeCollision(brake));
	return brake;
}
```

[Test drive with other robots or driver]

## Avoiding Side Collision

In this section we add a collision avoidance mechanism for cars that drive 
aside of our car. The following implementation will add an additional filter 
to the steer value. But it doesn't check if we leave the track. 

We start with the declaration in _carcontroller.h_.

```cpp
class CarController {

	public: 
        // ...
    	static const float SIDE_COLLISION_MARGIN;

    private:
        // ...
		float FilterSteeringCollision(float steering);
        // ...
}
```


We go in _carcontroller.cpp_ for the definitions. 
The `SIDE_COLLISION_MARGIN` constant at the beginning. 

```cpp
const float CarController::SIDE_COLLISION_MARGIN = 2.0; /* [m] */
```

The `FilterSteeringCollision` method do an approximate search of the 
nearest car aside of our agent's one, at first. If there is another car it 
checks if it is inside the `SIDE_COLLISION_MARGIN`. If so, it computes a 
new steer value and returns it. 

Concretely, we iterate through all cars and check those which have been 
marked with the `OPP_SIDE` flag. Among those cars, we select the closest 
one, using `min_side_distance` to keep the minimum distance between our 
car and another one, and `opp` a pointer to hold the closest car flagged
with `OPP_SIDE`. Note: `min_side_distance` is initialized with the maximum 
float value because we search for the minimum. 

If we found an opponent with our search criteria (`opp` is not null) we check 
if it is inside the margin. To keep the method simple we assumed that our car 
is parallel to the track (which is not true most of the time). 

After that, we compute `dist`, the side distance between our car and the 
closest opponent, without the opponent car's width. If `dist` is below the 
side collision margin, we compute `c`, the half of the side collision margin. 
After that, we compute `parallel_steering` which is the amount of steering to 
apply to drive parallel to the opponent we are trying to avoid. It is 
obtained by normalizing the `angle_difference` between our car and the 
opponent's. 

The value `dist / c` is used to weight the sum of the normal `steering` and the 
`parallel_steering`. The result (steering) value will turn away our car 
from the opponent's when we are near (about to hit) him.

The mechanism described here can be improved with moore accurate distances, 
and also checking if the normal steering value already point away of the 
opponent's car, before further computations.

[TODO: an illustration of the situation]

```cpp
/**
 * Filter the steering to avoid a collision from the side
 **/
float CarController::FilterSteeringCollision(float steering){
	int i;
	float side_distance = 0.0, 
		abs_side_distance = 0.0, 
		min_side_distance = FLT_MAX;
	Opponent *opp = NULL;

	/* get the index of the nearest car (o) */
	for (i = 0; i < opponents->Count(); i++) {
		if (opponent[i].GetState() & OPP_SIDE) {
			side_distance = opponent[i].GetSideDistance();
  			abs_side_distance = fabs(side_distance);
			if (abs_side_distance < min_side_distance) {
				min_side_distance = abs_side_distance;
				opp = &opponent[i];
			}
		}
	}
	/* if there is another car handle the situation */
	if (opp != NULL) {
		float dist = min_side_distance - opp->GetWidth();
 		/* near enough */
		if (dist < SIDE_COLLISION_MARGIN) {
			/* compute angle between cars */
			tCarElt *ocar = opp->GetCar();
			float angle_difference = ocar->_yaw - car->_yaw;
			remainder(angle_difference, 2*PI);
			const float c = SIDE_COLLISION_MARGIN/2.0;
			dist = dist - c;
			if (dist < 0.0) dist = 0.0;
			float parallel_steering = angle_difference / car->_steerLock;
			return steering*(dist / c) + 2.0* parallel_steering * (1.0 - dist / c);
		}
		return steering;
	}
	return steering;
}
```

We can now use the filter in the `GetSteering` method. Change the last line to 
the following two.

```cpp
float CarController::GetSteering(float car_angle)
{
    // ...
	float steering = target_angle / car->_steerLock;
	return FilterSteeringCollision(steering);
}
```

[Test drive, video]

# Overtaking

If we race, it is to win! So we have to tell our ai agent how to overtake. 
As usual, the overtaking mechanism we implement here is quite basic. 

If there are opponent cars in front ours (flagged with `OPP_FRONT`), we pick 
the one with the smallest catch up distance, to attempt an overtake.

To overtake an opponent we will add (another) modification to the steering 
value. This is based on modifying the target point used for our trajectory.
Depending on the side we want to pass an opponent, we slightly add an offset 
vector to the target point. This offset is perpendicular to the track tangent. 

[TODO: Illustration]

Let's make the necessary declarations in _carcontroller.h_. 

```cpp 
class CarController {

	public: 
        // ...
   		static const float BORDER_OVERTAKE_MARGIN;
		static const float OVERTAKE_OFFSET_INCREMENT;
    
    private: 
        // ...
   		float GetOvertakeOffset();
        // ...
   		float overtake_offset;
        // ...
}
```

Then in _carcontroller.cpp_ we define the values of the constants at the 
beginning of the file.


`BORDER_OVERTAKE_MARGIN` is the border relative to the 
`FilterTrack(float acceleration)` width. It should avoid that we leave the 
range on the track where `FilterTrack`allows acceleration. 
`OVERTAKE_OFFSET_INCREMENT` is the increment of the offset value. 
We also lower the value of the `WIDTH_DIV` constant.


```cpp
// ...
const float CarController::WIDTH_DIV = 3.0; // [-]
// ...
const float CarController::BORDER_OVERTAKE_MARGIN = 0.5; /* [m] */
const float CarController::OVERTAKE_OFFSET_INCREMENT = 0.1;    /* [m/timestep] */
```

`overtake_offset` is initialize to zero in the `NewRace()` method.

```cpp
void CarController::NewRace(tCarElt* car, tSituation* s)
{
    // ...
	overtake_offset = 0.0;
    // ...
}
```

And here is the implementation of `GetOvertakeOffset()` that compute the 
offset to the target point.

[More explanation ?]
Now we have a look on how we compute the offset of the target point. 

As we discussed earlier, we hold a pointer `opp` to the car with the smallest 
`catch_distance` (the one we will reach first), among the car with the 
`OPP_FRONT` set (in front of us).

In case we found an opponent to overtake (`opp` is not `NULL`), we check 
which side of the track that opponent is, and if there is enough space on the 
other side of the track to overtake.

In case we have not found an opponent to overtake, the offset goes back 
slightly toward zero. In the next section we will add the offset to the 
target point. 

```cpp
/**
 * Compute an offset to the trajectory target point for overtaking.
 * */
float CarController::GetOvertakeOffset(){
	int i;
	float catch_distance, 
		min_catch_distance = FLT_MAX;
	Opponent *opp = NULL;

	for (i = 0; i < opponents->Count(); i++) {
		if (opponent[i].GetState() & OPP_FRONT) {
			catch_distance = opponent[i].GetCatchDistance();
		if (catch_distance < min_catch_distance) {
				min_catch_distance = catch_distance;
				opp = &opponent[i];
			}
		}
	}
	if (opp != NULL) {
		float w = opp->GetCar()->_trkPos.seg->width / WIDTH_DIV - BORDER_OVERTAKE_MARGIN;
		float opp_to_middle = opp->GetCar()->_trkPos.toMiddle;
		if (opp_to_middle > 0.0 && overtake_offset > -w) 
			overtake_offset -= OVERTAKE_OFFSET_INCREMENT;
		else if (overtake_offset < 0.0 && overtake_offset < w) 
			overtake_offset += OVERTAKE_OFFSET_INCREMENT;
	} else {
		if (overtake_offset > OVERTAKE_OFFSET_INCREMENT) 
			overtake_offset -= OVERTAKE_OFFSET_INCREMENT;
		else if (overtake_offset < -OVERTAKE_OFFSET_INCREMENT) 
			overtake_offset += OVERTAKE_OFFSET_INCREMENT;
		else 
			overtake_offset = 0.0;
	}
	return overtake_offset;
}
```


Finally, we need to add the offset to the target point. We will include the 
calculations that directly into the `GetTargetPoint()` method. For that, we 
compute a normalized vector at the target point perpendicular to the track 
tangent. We also multiply it with the `offset` before we adding it to the 
target vector. 

Here are the modifications of `GetTargetPoint()`. The first change is the 
`offset` variable, initialized with GetOvertakeOffset(). 

The `tangent_perpendicular` holds the normalized vector perpendicular to 
the track tangent at the target point. The computation depends on whether 
the target point is on a straight or a turn. 

```cpp
/**
 * Find a target point ahead of the car toward which the wheels will steer
 */
Vector2D CarController::GetTargetPoint()
{
	tTrackSeg* segment = car->_trkPos.seg;
	float look_ahead = LOOK_AHEAD_CONST + car->_speed_x * LOOK_AHEAD_FACTOR;
	float length = DistanceFromCarToSegmentEnd();
	float offset = GetOvertakeOffset();

	while (length < look_ahead) {
		segment = segment->next;
		length += segment->length;	
	}
	length = look_ahead - length + segment->length;
	float target_x = (segment->vertex[TR_SL].x + segment->vertex[TR_SR].x) / 2.0;
	float target_y = (segment->vertex[TR_SL].y + segment->vertex[TR_SR].y) / 2.0;
	Vector2D target(target_x, target_y);
	if ( segment->type == TR_STR) {
		float tangent_perpendicular_x = (segment->vertex[TR_EL].x - segment->vertex[TR_ER].x) / segment->length;
		float tangent_perpendicular_y = (segment->vertex[TR_EL].y - segment->vertex[TR_ER].y) / segment->length;
		Vector2D tangent_perpendicular(tangent_perpendicular_x, tangent_perpendicular_y);
		tangent_perpendicular.Normalize();
		float direction_x = (segment->vertex[TR_EL].x - segment->vertex[TR_SL].x) / segment->length;
		float direction_y = (segment->vertex[TR_EL].y - segment->vertex[TR_SL].y) / segment->length;
		Vector2D direction(direction_x, direction_y);
		return target + direction * length + offset * tangent_perpendicular;
	} else {
		Vector2D center(segment->center.x, segment->center.y);	
		float arc = length / segment->radius;
		float arcsign = (segment->type == TR_RGT) ? -1 : 1;
		arc *= arcsign;
		target =  target.Rotate(center, arc);
		Vector2D tangent_perpendicular = center - target;
		return target + arcsign * offset * tangent_perpendicular;
	}
}
```

[Test drive & video]

[TODO: about camber setting change]

Let me know if you have a different implementation


* [Previous: Recovering]({filename}torcs-ai-tuto-05.md)
* [Next: Pit Stops]({filename}torcs-ai-tuto-07.md)