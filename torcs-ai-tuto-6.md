title: Building a Simple TORCS AI Agent (6/11): Recovering
date: 2018-12-11 15:00
category: Games
authors: Didier Gohourou
summary: Getting back on track after getting off track or getting stuck.
slug: torcs-ai-tuto-6
lang: en


<p id="introduction"></p>
# Introduction

Those who have been coding all along, <del>not the copy-pasters</del>, if you 
made some mistakes in the values or operations while coding, the car sometimes 
ended up in the wall and got _stuck_... Yeah I can imagine the frustration...
Even if you "flawlessly" implemented everything, we are still running 
alone on the track. Because we plan to build a agent good enough to race 
against other opponents, we can think of what will happen if an opponent hits
us and we go off-course... We will end up being _stuck_!

But **NO MORE** ! In this chapter, we will implement a recovery strategy to get out 
of an accident and out of being stuck.

<p id="basic_recovery"></p>
# A Basic Recovery Strategy

Basically if our car is stuck, we recover, else, we drive normally.
We will consider that our car is stuck, if the angle of the car relative to the 
track is greater than a certain value during a certain period of time. 
We will use 30 degrees as our angle limit, and 100 the number of simulation steps
up to which we will consider the car stuck.
Since the simulation time is 0.02 seconds, the `Drive(...)` method is 
executed every 0.02 seconds. This means that our car needs to form an angle 
with an absolute value greater than 30 degrees relatively to the track, 
for 2 seconds (or more) before we trigger the recovering process.

Let's start by writing a method that tells us if our car is stuck or not. 
The implementation steps are quite simple:

1. Initialize a counter to 0
2. If the absolute value of the angle between the car and the track is smaller 
than a certain value (30 degree), reset the counter to 0 and return false 
(Meaning "not stuck")
3. If the angle is smaller than a certain limit (100), increase the counter and 
return false (for "not stuck"). Otherwise return true (for "stuck").


```cpp


```


As previously explained, this method will be used in our driving strategy. 

```cpp

```


Below is a illustration to help us get our head wrapped around a situation of 
getting stuck.

![Getting Stuck 1]({filename}#)

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

![Getting Stuck 2]({filename}#)

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

![Look inside criteria]({filename}#)

