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


# The Pit Class


# The Pit strategy


# The Pit Brake filter


* [Table of Contents]({filename}torcs-ai-tuto-1.md#table_of_contents)
* [Previous: Collision avoidance and Overtaking]({filename}torcs-ai-tuto-7.md)
* [Next: Customizing the car settings and appearance]({filename}torcs-ai-tuto-9.md)