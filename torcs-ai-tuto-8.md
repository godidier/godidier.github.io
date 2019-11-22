title: Building a Simple TORCS AI Agent (8/11): Pit Stops
date: 2018-12-13 15:00
category: Games
authors: Didier Gohourou
summary: Stopping at the pit to refuel and repair damages
slug: torcs-ai-tuto-8
lang: en


* [Table of Contents]({filename}torcs-ai-tuto-1.md#table_of_contents)
* [Previous: Collision avoidance and Overtaking]({filename}torcs-ai-tuto-7.md)
* [Next: Customizing the car settings and appearance]({filename}torcs-ai-tuto-9.md)

# Introduction

During the race, our car consumes fuel, and is likely to endure damages 
over time. In longer races, we might not be able to drive any further 
if our fuel tank is empty (the fuel value get to 0), or the car damages 
exceed a certain threshold (10000 damages points). To stay in the race, we 
might want to go to the pit stop to refuel and repair the car periodically, 
according to a strategy. In this chapter we will learn how to send our car 
to te pit stop for reparation and refuel. 

But how does the pit stop works in TORCS ? Here are the steps to do a pit 
stop in TORCS. 

1. The AI agent decides to go to the pit. 
2. The AI agent enters the pit and drive into the pit lane with respect to 
the speed limit
3. The AI agent drive to its pit and stups there
4. Once the agent (is about to) stops in its pit, the simulation captures the 
car and ask for how much damage point to repairs and the amount of refuel
5. The TORCS simulation repairs and refuel the car, then release it
6. The car drive to the pit exit in respect to the speed limit
7. The AI agent can continue racing

From the above step we can detect the item to implement for pit stops.

* A strategy to decide if we need a pit stop
* A path to drive into, within and out of the pit lane
* An additional brake filter to stop in the pit
* A callback to tell the simulation (TORCS) how much repairs and fuel we need


Note that in TORCS there is no tyre wear, so no need to worry about our 
tyres, and there un pit available per car so no need to verify if there is 
a pit available before going to pit stop.

We will start by defining the spline class that will help us trace our path 
to the pit stop, then we will define a Pit class where we will implement 
helpers methods to achieve what we defined earlier.


# The Spline

## A spline ? 

In the last section you have seen that we need to find somehow a path into our pit. I decided to show you an implementation which uses "cubic" splines to create the path. Cubic splines are interpolating polynomials between given base points, such that the function itself and the first derivative is smooth. We need a smooth path because our car can not follow a path with "cracks". First I will show you the implementation of the spline algorithm, after that we will prepare the base points. 

## Spline implementation

If you want to know how the following algorithm works, you can look it up here. In a nutshell, you query for a given parameter ("x") the value of the spline function ("y"). The algorithm searches first the interval in which "x" falls, then it applies the interpolation to compute y(x). Now let's have a look on the spline.h file (create it in the bt directory). 

The SplinePoint class holds the data for each base point of the spline. For the interpolation algorithm we need to define also the slope at (x,y). 

The Spline class provides just two methods. The constructor to initialize the number of base points (dim) and a pointer to an array of SplinePoints. 

```cpp
Spline::Spline(int dimension, SplinePoint *spline_points )
{
    this->spline_points = spline_points ;
    this->dimension = dimension;
}
```

The second method finally computes the spline values. Here follows the implementation, put it into spline.cpp. 

First the algorithm searches for the interval in which the parameter z falls. 

```cpp 
float Spline::Evaluate(float z)
{
    int i, a, b;
    float t, a0, a1, a2, a3, h;

    a = 0; b = dimension - 1;
    do {
        i = (a + b) / 2;
        if (spline_points[i].X <= z) a = i; else b = i;
    } while ((a + 1) != b);
    i = a; h = spline_points[i+1].X - spline_points[i].X; t = (z-spline_points[i].X) / h;
    a0 = spline_points[i].Y; a1 = spline_points[i+1].Y - a0; a2 = a1 - h*spline_points[i].s;
    a3 = h * spline_points[i+1].S - a1; a3 -= a2;
    return a0 + (a1 + (a2 + a3*t) * (t-1))*t;
}
```

Finally you need to add spline.cpp to the Makefile. Change 

```
SOURCES     = ${ROBOT}.cpp carcontroller.cpp opponent.cpp spline.cpp
```


# The Pit Class

Eggsplain pitstop process ...

The following picture shows the path to and out of the pit. Like you can see we need the base points p0 to p6 to be able to compute the path. Our x-axis is the middle line along the track, so the slope in the base points is zero (parallel to the track). 

[TODO show pic]

We compute the base points of the spline in the constructor of a new class, put it into a new file and save it as pit.cpp. That we don't need to change the constructor all the time I will present you the final result. We will discuss all the new variables later in detail, at the moment we just focus on the base points. The header file will follow later. 



Let's prepare the header file for the pit class. It contains the definition 
of the functionality to decide if we want to pit (the strategy), to compute 
the pit path (into, within and out of the pit) and some utility functions. 
Put the following code into a new file named _pit.h_. 

```cpp
```

# The Pit strategy


# The Pit Brake filter


* [Table of Contents]({filename}torcs-ai-tuto-1.md#table_of_contents)
* [Previous: Collision avoidance and Overtaking]({filename}torcs-ai-tuto-7.md)
* [Next: Customizing the car settings and appearance]({filename}torcs-ai-tuto-9.md)