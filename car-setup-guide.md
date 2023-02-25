title: Car setup guide from racing (simulation) games
date: 2019-07-21 06:00
category: Games
authors: Didier Gohourou
summary: Guides and tips for setting up a racecar, from racing games (simulations)
slug: car-setup-guide
lang: en
status:published

This is a list of car setup guides from video games mainly Asseto Corsa and 
Gran Turismo. 


# Asseto Corsa

## Tyres 

### Pressure (psi)
Raising the tyre pressure will give more precise steering, higher top speed and 
maybe lower the tyre temperatures if you don't overdrive the car. On the other 
hand it will make the tyre slide earlier if you overdo it.

Lowering the tyre pressure will give more traction and grip, but also less 
precision, lower top speed and more heat. 

Tyre pressure is all about compromise.

## Electronics

### ABS
It prevents locking of the wheel of a vehicle during braking. 

A value of 1 equals to a greater intervention of control. 

### MGUK-Delivery
Motor Generator Unit - Kinetic: Deployment Profiles

There are named profiles that define variables rates of MGU-K power output to the 
drive wheels under power.

These profiles try to optimise MGU-K power output during these acceleration phases
of a lap, sometimes sacrificing top speed in the process.

Some profiles also reduce MGU-K output at very low speeds where there may not 
be enough traction available to the drive wheel to manage the available torque.

### MGUK-Recovery

Motor Generator Unit - Kinetic: Recovery Level

Manages how aggressively the MGU-K harvest energy from braking events on the 
interested axle. With 100% being the most aggressive setting and thus harvesting 
the most energy into the battery at a given time.

Thus, management of this setting can affect the handling of the car in a number of 
ways: 

* An higher percentage of energy regeneration in the MGU-K will mean for a greater 
level of retardation upon the interested axle when off throttle (coast) and 
braking, possibly resulting in entry oversteer. Higher regen will also result in 
longer braking distances.
* A lower percentage of energy regeneration will mean less energy is being charged 
into the ERS battery for deployment on power. The offset to this is more precise 
level of braking control via normal brake balance, and shorter braking distances.

### MGUH-Mode
Motor Generator Unit - Heat

This setting controls how the MGUH operates in conjunction with other PU 
components:

Motors: In this mode the MGUH will recover energy from exhaust gases and direct 
this power directly into the MGU-K, thus supplementing overall power output.

Battery: in this mode the MGU-H recovers exhaust gases and diverts this energy 
into the ERS battery to increase SOC.

### Brake Engine
This setting sets ECU within the ICU to retain a small percentage of fuel flow to 
blow onto the diffuser, reducing engine braking from the ICU on coast. This offset
the high level of coast locking on the interested axle that is generated with 
higher MGU-K regen settings.

Lower settings reduce the level of engine braking and thus reduces retardation 
from the drivetrain onto the interested axle under coast. This provides easier 
management of interested axle locking with MGU-K regen and brake balance.

Due to the increase in diffuser exhaust flow, a lower setting will also provide 
additional rear downforce and stability. However, a lower engine brake settings 
will consume more fuel and thus affect fuel consumption over a stint. 

Higher settings allow a more conventional drivetrain linkage and thus more 
retardation to the interested axle from the ICU, this needs to be balanced against 
MGU-K regen settings to provide a comfortable balance for the driver along with 
suitable fuel consumption numbers.


## Aero

### FWING (Front Wing)

The front wing helps the car to gain downforce at the front end.

Raise the value to eliminate understeer at medium/high speeds. 

Lower the value to make the car more stable at medium/high speeds.

You can also gain downforce by making it work closer to the ground.

### REAR\_MOVABLE (Rear Wing)

The rear wing produces downforce at the rear end of the car, but at the expense of 
more drag.

Raise the value to make the car more stable at medium/high speeds and increase 
traction, but will lose top speed.

Lower the value to make the rear end of the car more lively and gain top speed.

## Alignement

### Camber

Raising the value makes the wheel gain negative camber. More negative camber gives 
more turn in and lateral grip but less longitudinal grip while braking or 
accelerating. More camber also raises tyre temperatures.

Lowering the value makes the wheel lose negative camber. Less negative camber 
gives less lateral grip but also more longitudinal grip while braking or 
accelerating. It also lowers tyre temperatures spreading them more equally.

### Toe

A negative value means the wheel, as seen from above looks towards the outside of 
the car. Gives faster turn in, but also instability under braking, higher tyre 
temperatures and less mid turn grip. 

A positive value means the wheel, as seen from above, looks towards the inside of 
the car. Gives slower turn in, but more stability under braking and higher tyre 
temperatures. 

A zero value gives higher top speed and lower tyre temperatures.


## Brakes

### Brake Bias 

Changes the amount of braking power towards the front or the rear of the car. 

More forward bias makes the car more stable but also can block faster the front 
tyres and make the end uncontrollable under braking and turn in. 

More rearwards bias makes the car more agile and willing to rotate in turn in but 
also less stable and easy to spin. 

## Dampers
The damper stops the spring oscillation.

### Bump 
Damper bump creates a force to slow the 
speed of the suspension in compression (bump) when slow movements occur, primarly 
on weight transfers (roll, pitch).

Higher values make the car more reactive giving better feeling and precision, 
stability under braking but also higher tyre temps and wear. 

Lower value makes the car more predictable but slower in reactions, less precision 
and less stability under braking. Lowers tyre temperature and wear.

### FST Bump 
Damper fast bump creates a force to slow the speed 
of the suspension in compression (bump) when fast movements occur, primarily road 
irregularities and kerbs.

Higher values make the suspension absorb less the impact from a kerb but also keep
the spring compression limited which can be useful as you don't want the spring to 
gain much energy and bounce back violently after a bump. 

Makes the front end a bit nervous over kerbs and bumps.  

Lower value makes the suspension capable of absorbing more energy on bumps and 
kerbs but then you need to make sure it won't bounce down again upsetting the car.

### Rebound
Damper rebound creates a force to slow the speed of the suspension in extension 
(rebound) when slow movements occur, primarly on weight transfer (roll, pitch)

Higher values at the front, make the car more stable under acceleration and on 
fast change direction like chicanes.

Lower values eliminates understeer under acceleration but also makes the car more 
sluggish on changing direction.

### FST Rebound
Damper fast rebound creates a force to slow the speed of the suspension in 
extension (rebound) when fast movements occur, primarly road irregularities and 
kerbs. 

Higher values make the suspension extension slower, thus limiting the shock the 
tyres get from the energy of a compressed spring. If the fast rebound value is too 
high then the suspension might not have time to extend again after a kerb and it 
will keep the front end lower to the ground. This can be good for aerodynamics but 
also bad because on the next bump there will be no suspension travel available.

Lower values let the suspension extend more freely, but can also provoke a bouncing
of the tyres and thus worsening the tyre contact on the ground making the front 
end lose grip on bumps and kerbs.

Find the correct compromise.

## Driverain

### Diff Power 
Differential Lock under power.

When a wheel is spinning under power it loses grip. A locked differential will 
transmit more power on the non spinning wheel, thus aiding traction but also 
changing the handling of the car.

Higher values will aid traction until grip is enough but then will provoke both 
driven wheel spinning.

This will give oversteer to the RWD cars and understeer on FWD under power.

Lower values will give less traction, letting the spinning wheel to continue 
spinning. Makes the car more stable under power but also makes you lose time 
and overheat the spinning tyre.

### Diff Coast 
Differential Lock in coast.

When braking or simply coasting the locked differential will try to keep both 
wheels at the same speed. This can change the handling of the car. 

Higher values make for a more stable car under braking but also make the car want 
to turn less (more understeer). 

Lower values make the car less stable under braking and in turn in, but also make 
the car more agile and willing to rotate.

### Diff Preload
Differential preload (Nm)

Keeps the differential locked until the torque difference is equal to the value 
indicated. Then the lock becomes the one in power or coast. 

Zero or low values makes car agile in the initial turn in, but can also make it 
nervous and lose traction if driven abrutly.

Higher values can make the car more stable and give traction but can also overcome
the differential power and coast settings making the car feel very tight and 
resistantto rotate.


## Suspension 

### ARB Front 
Front Anti Roll Bar.

Raise the value to make the front end roll less and gain stability, precision and 
fast turn in. Usually you might lose some grip on the front end, but it's not 
always the case, depending on the suspension geometry.

Lower the value to make the front end roll more. You will have better mid turn grip
but you will lose precision and agililty and a slower turn in.

Keep higher values on tracks with lots of bumps and kerbs to aid stability.

### ARB Rear
Rear Anti Roll Bar.

Raise the value to make the rear end roll less and be more reactive on driver 
inputs. Usually you might lose some grip on the rear end, but it's not always 
the case, depending on the suspension geometry and rear weight distribution. 

Lower the value to make the rear end roll more. You will have better traction but 
you will lose reactivity and agility in turn in.

Keep lower values on tracks with lots of bumps and kerbs to aid stability.

### Wheel rate
This is the spring force, multiplied buy the install ratio of the suspension. 

Raise the value to stiffen the suspension and get a more direct and precise 
handling with fast reactions.

Lower the value to soften the suspension and get a more slow and predictable 
handling.

General wisdoms says the stiffer the suspension the less grip you get, but this is 
not always the case and it depends on suspension geometry and other design choices.

As usually try to find a compromise.

### Height
Modifies the length of the damper rod and subsequently the ride height of the car.

Keep in mind that ride height also varies depending the wheel rate values, the 
tyre pressures, the fuel load and the travel and rigidity of the bumpstop packers.

## Suspension Adv.

### Packer rate

This is usually a rubber spring before the actual end of the suspension range. 
Think of it as an extra spring that start working when the suspension is compressed
enough to hit on it.

Left/Right front packer rate:
Raise the value to limit the suspension travel after it hits the bump stop, make 
the front end more reactive, limit the aero pitch sensitivity effects and stabilise
the car under braking.
Lower the value to permit a bit more suspension travel after it hits the bump stop,
make the front end more predictable and make the aero pitch sensitivity effects 
under braking.

Left/Right rear packer rate:
Raise the value to limit the suspension travel after it hits the bump stop, make 
the rear end more reactive, limit the aero pitch sensitivity effects of the 
diffuser under acceleration.
Lower the value to permit a bit more suspension travel after it hits the bump stop,
gain traction at the rear end and raise the aero pitch sensitivity of the diffuser
while accelerating.

### Travel range
This is the suspension travel before it hits the bumpstop.

With a lower value the suspension has more travel before it eventually hits the 
bumpstop.

With a lower value you hit the bumpstop sooner, thus blocking the suspension travel.

Check the bumpstop rate help to understand what happens when you the suspension 
hits a bumpstop.

