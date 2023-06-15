title: Create Your First TORCS Racing AI Bot - Part 07: Pit Stops
date: 2018-12-13 15:00
modified: 2019-11-22 21:37
category: Tutorials
authors: Didier Gohourou
summary: Stopping at the pit to refuel and repair damages
slug: torcs-ai-tuto-07
lang: en


* [Table of Contents]({filename}torcs-ai-tuto-00.md#table_of_contents)

# Introduction

During the race, our car consumes fuel, and is likely to endure damages 
over time. In longer races, we might not be able to drive any further 
if our fuel tank is empty (the fuel value get to 0), or the car damages 
exceed a certain threshold (10000 damages points). To stay in the race, we 
might want to go to the pit stop to refuel and repair the car periodically, 
according to a strategy. In this chapter we will learn how to send our car 
to the pit stop for reparation and refuel. 

But how does the pit stop works in TORCS ? Here are the steps to do a pit 
stop in TORCS. 

1. The AI agent decides to go to the pit. 
2. The AI agent enters the pit and drive into the pit lane with respect to 
the speed limit
3. The AI agent drive to its pit and stops there
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

We will start by defining the **spline** class that will help us trace our path 
to the pit stop, then we will define a Pit class where we will implement 
helpers methods to achieve what we defined earlier.


# The Spline

## A spline ? 

We wil start with a method to define a path into our pit. For this task we 
choose to implement "cubic" splines to create the path. Cubic splines are 
interpolating polynomials between given base points, such that the function 
itself and the first derivative are smooth. We need a smooth path because 
our car can not follow a path with "cracks". 

[Image of smooth path vs non smooth path ?]

## Spline implementation

Here we define the algorithm to draw a path into our pit using 'cubic' splines.

In a nutshell, you query for a given parameter $x$ the value of the spline 
function $y$. The algorithm first searches the interval in which $x$ falls, 
then it applies the interpolation to compute $y(x)$. 
[Detail explanation of splines ?
If you want to know how the following algorithm works, you can look it up here.] 

We create a _spline.h_ file in the _gtr_ directory. In this file, we declare 
a `SplinePoint` class to hold data for the base point of the spline, as well 
as the slope we will use for the interpolation algorithm. 

We also declare a `Spline` class with a constructor, a method to evaluate
the spline at a given point and a collection of spline points represented 
by an array and its size.

```cpp
#ifndef _SPLINE_H_
#define _SPLINE_H_

/**
 * A spline point
 **/
class SplinePoint {
    public: 
        float X; // x coordinate
        float Y; // y coordinate
        float S; // slope
};


/**
 * A Spline
 **/
class Spline{
    // TODO: basically a vector 
    public: 
        Spline(int dimension, SplinePoint *spline_points);
        float Evaluate(float z);
    
    private:
        SplinePoint *spline_points;
        int dimension;
};

#endif // _SPLINE_H_
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

[Eggsplain pitstop process ...]

Let's prepare the header file for the pit class. It contains the definition 
of the functionality to decide if we want to pit (the strategy), to compute 
the pit path (into, within and out of the pit) and some utility functions. 
Put the following code into a new file named _pit.h_. 

```cpp
```

The following picture shows the path to and out of the pit. We will have to 
designate base point (p0 to p6) to compute the path. We choose the points 
such thta our x-axis is the middle line along the track, so the slope at 
the base points is zero (parallel to the track). 

![Path to the PIT]({static}/images/2020-02/path-to-pit.svg)

Let's implement the methods of our pit class. We compute the base points of 
the spline in the constructor. 

[Eggsplain the spline point ?]

```cpp
// put constructor here
```


# The Pit strategy


# The Pit Brake filter


* [Previous: Collision avoidance and Overtaking]({filename}torcs-ai-tuto-06.md)
* [Next: Customizing the car settings and appearance]({filename}torcs-ai-tuto-08.md)