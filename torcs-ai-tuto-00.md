title: Create Your First TORCS Racing AI Bot - Part 0: Getting Started
date: 2018-12-07 15:00
category: Tutorials
authors: Didier Gohourou
summary: Getting to setup and know TORCS
slug: torcs-ai-tuto-00
lang: en
status: published


This is the beginning of a series on how to build an AI agent for the game 
TORCS (The Open Racing Cars Simulation). TORCS is an open source video game.
You can checkout the Wikipedia page of the game for a 
[History of TORCS](https://en.wikipedia.org/wiki/TORCS) from little AI game 
to Speed Dreams and others.

# Why bother ? 
<p id="why"></p>

Writing an AI for a racing game ? Maybe you already have your own motivation, you 
like racing, you might like computer science an AI, you are an enthusiast 
programmer etc. But above all this series aim to give clear fundamentals on the 
task... Or just maybe you are like me... you have plenty of time to waste! 
Even then, it is a wise place to waste it.

A quick disclaimer: this series is <del>highly</del> Inspired from the robot 
tutorial included in the game. You might also want to check that in case you 
want a different perspective... <del>Don't look at me like that!</del> 
You should be careful though, because there are some differences in the progression 
and the code.

# Pre-requisites
<p id="prerequisites"></p>

This tutorial assumes that you have some basic C++ knowledge, and that you have
some experience with high school mathematics and physics.

Nevertheless, I will do my best to be as clear as possible and give pointers 
for further readings, to those interested in learning more.

On a side note, I strongly encougare you to code along with the tutorial by 
yourself. Sure you can copy-paste the code to go fast, but you will miss 
the mistakes that make you learn and understand concepts far better. 
When you make mistakes or things go sideways, try to re-analyse the code and the 
explanations thoroughly and try again. This way you will come up with a deeper 
understanding of how things works. This will enable you to improve the code 
beyond the tutorial, and easily translate your knowledge elsewhere.

<p id="table_of_contents"></p>

# Content

1.	[Getting started]({filename}torcs-ai-tuto-00.md)
2.	[Acceleration]({filename}torcs-ai-tuto-01.md)
3.	[Brakes and Gears]({filename}torcs-ai-tuto-02.md)
4.	[Aerodynamics and Stability Controls]({filename}torcs-ai-tuto-03.md)
5.	[Steering and Trajectory]({filename}torcs-ai-tuto-04.md)
6.	[Recovering]({filename}torcs-ai-tuto-05.md)
7.	[Collision Avoidance and Overtaking]({filename}torcs-ai-tuto-06.md)
8.	[Pit Stops]({filename}torcs-ai-tuto-07.md)
9.	[Tuning and Customization]({filename}torcs-ai-tuto-08.md)
10.	[Handling Multiple Cars]({filename}torcs-ai-tuto-09.md)
11.	[Wrap Up]({filename}torcs-ai-tuto-10.md)

<p id="installation"></p>

# Installation 

Let's install the game!

TORCS can be installed via binaries but we are here to implement AI agent for the 
game so we need to install via from the sources. The binaries and sources of 
TORCS are available at the 
[Official site of TORCS](http://torcs.sourceforge.net/index.php) within the 
[Download/Installation](http://torcs.sourceforge.net/index.php?name=Sections&op=viewarticle&artid=3) section. 
There are also basic installatioin instruction on that page.

**Note**: We will give instructions mainly for an RPM package oriented Linux 
distribution like Fedora, and Ubuntu as well. Excuse my lazyness but I 
might give instructions for 
more Operation System in the future. Meanwhile one can use the instructions 
available on the [Download/Installation](http://torcs.sourceforge.net/index.php?name=Sections&op=viewarticle&artid=3) page. 

## Requirements

### Hardware

They are pretty low: 

* CPU > 800MHz 
* RAM > 256 MB
* A very basic graphic card

Computers dating from the Pentium3 era can run TORCS. So basically if you are 
reading this on a computer today, you should be able to run TORCS.

### Software

Installing the necessary development package for your (Linux) distribution
is highly recommended. 

On Fedora you might want to type the following in the command line: 
```sh
sudo dnf update
sudo dnf groupinstall "Development Tools" "Development Libraries"
```

On Ubuntu enter the following:
```sh
sudo apt update 
sudo apt install build-essential 
```

The main package to compile the game are:

* Hardware accelerated OpenGL
* [FreeGlut](http://freeglut.sourceforge.net/)
* [PLIB](http://plib.sourceforge.net/) version 1.8.5 or higher
* [OpenAL](http://kcat.strangesoft.net/openal.html)
* [libpng](http://libpng.sourceforge.net/)
* [libogg](http://www.vorbis.com/)

All of the above are usually provided by linux distribution. 
Install them using your package manager (dnf on Fedora, apt on Ubuntu.

Below is a command to install the libraries you might need, including 
additional ones (at least the one I needed).

On Fedora:
```sh
sudo dnf install freeglut.x86_64 freeglut-devel.x86_64 \
    plib.x86_64 plib-devel.x86_64 \
    openal-soft.x86_64  openal-soft-devel.x86_64 \
    freealut.x86_64 freealut-devel.x86_64 \
    libXmu.x86_64 libXmu-devel.x86_64 \
    libXxf86vm.x86_64 libXxf86vm-devel.x86_64 
```

On Ubuntu: 
```sh
sudo apt install freeglut3 freeglut3-dev \
    libplib-dev \
    libopenal1  libopenal-dev  libopenal-data \
    libalut0  libalut-dev \
    libvorbis-dev  libvorbis-dbg \
    libxi6  libxi-dev \
    libxmu6  libxmu-headers  libxmu-dev \
    libxrender1  libxrender-dev \
    libxrandr2  libxrandr-dev \
    libpng16-16  libpng-dev
```

After installing those packages you should be good. If you have any trouble,
check the [Troubleshooing](#troubleshooting) section of this article. 


## Getting the Code

The source code of TORCS is available at the TORCS's Sourceforge project 
[page](https://sourceforge.net/projects/torcs/).
Please select the "all-in-one" version when downloading. This tutorial uses 
TORCS 1.3.7 (all-in-one) package which is available [here](http://sourceforge.net/projects/torcs/files/all-in-one/1.3.7/torcs-1.3.7.tar.bz2/download)

## Install TORCS From the Sources

* Unpack the downloaded file. Carefully choose the folder where you will unpack 
the package. I recommend `/usr/local/src`

```
cd /usr/local/src
tar xvf /path/to/downloaded/torcs-1.3.7.tar.bz2
```

Change the unpacked directory's ownership to your username. e.g. if your username
is _uname_ and the unpacked folder _torcs-1.3.7_, run:

```
sudo chown -R uname torcs-1.3.7
``` 

* Run the following command to compile and install TORCS:

```
cd torcs-1.3.7
./configure 
make 
make install
make datainstall
```

In case you have any trouble with those commands, 
check the [Troubleshooing](#troubleshooting) section of this article. 


## Environment Variables

Because we want to develop AI module for the game, we need to point the game 
librairies to our compiler when will be using `Makefile` for compilation. 
For that we set environment variables. 

```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
export TORCS_BASE=/usr/local/src/torcs/torcs-1.3.7
export MAKE_DEFAULT=$TORCS_BASE/Make-default.mk
```

To avoid setting up those variable before every development session, we can add 
the above lines to your shell's rc file like _~/.bashrc_ if you are using a bash 
terminal. You can run :

```
source ~/.bashrc
```

for the changes to take effect.


Remember if you have any trouble, you can try lookin at the 
[Download/Installation](http://torcs.sourceforge.net/index.php?name=Sections&op=viewarticle&artid=3) page instructions, or the installation instructions of the 
[TORCS Robot Tutorial](http://www.berniw.org/tutorials/robot/tutorial.html), on 
which this tutorial is based, or contact the [torcs-users](torcs-users@lists.sourceforge.net) mailing list.

# Gameplay 

After all this setup hassle, I recommend that you 
play the game. Familiarize yourself with it: the menu, the controls, etc.

You can lauch the game from the terminal. 

```
torcs
```

The video below gives an idea of how to setup a player, get to practice mode, select/de-select a car and drive. In the video I am using the keyboard's D-pad, don't judge me.
<iframe width="560" height="315" src="https://www.youtube.com/embed/-zHo3dQoAk8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Tip: If you are on Linux and interested in playing with a Playstation controller,
look [here](https://gameimps.com/ps3-controller-linux-usb-290) for how to use 
`xboxdrv` to connect your Playsation controller with a better support. 
It's also valid for Xbox controllers (Which are usually well supported).


<p id="troubleshooting"></p>

# Troubleshooting

### Trouble with the required package

**Note:** When running `./configure` if the command fails because of an 
uninstalled library, do a quick search with your package manager to make 
sure we find the correct package name for our distribution, and install it. 

On Fedora:
```sh
dnf search <package-name>
sudo dnf install <correct-package-name>
```

On Ubuntu:
```sh
apt search <package-name> 
sudo apt install <correct-package-name>
```

### Compilation Errors
During my recent compilation of **torcs-1.3.7** (April 2023) I experienced 
some compilation errors. Here is how I solved them: 

The first one was when running the `make` command. I encountered the 
following error: 
```sh 
"/usr/local/src/torcs-1.3.7/src/libs/musicplayer/OpenALMusicPlayer.cpp" 271 line
OpenALMusicPlayer.cpp:164:29: error: invalid conversion from ‘char’ to ‘const char*’ [-fpermissive]
```
To solve it, open the file */usr/local/src/torcs-1.3.7/src/libs/musicplayer/OpenALMusicPlayer.cpp*, go to line 271 and make the following change from:
```cpp
    const char* error = '\0';  
```
to (changing single quote to double quotes)
```cpp
const char* error = "\0";
```

The second bug appeared when running `make install`. The following error
showed up:
```sh
src/drivers/olethros/geometry.cpp
isnan not define. line 396
```
To solve it open the file */usr/local/src/torcs-1.3.7/src/drivers/olethros/geometry.cp*. Go to line 396 and change it from:
```cpp
    if (isnan(r)) {
```
to 
```cpp 
    if (std::isnan(r)) {  
```


### Others Troubles
Try lookin at the 
[Download/Installation](http://torcs.sourceforge.net/index.php?name=Sections&op=viewarticle&artid=3) page instructions, or the installation instructions of the 
[TORCS Robot Tutorial](http://www.berniw.org/tutorials/robot/tutorial.html), on 
which this tutorial is based, or contact the [torcs-users](torcs-users@lists.sourceforge.net) mailing list.

I understand it might be frustrating to experience trouble during the installation but please don't give up it's part of your learning journey. 
Take a break (minutes, hours, days) and try again.

# Moving Forward

Now that the setup is done, let's do what we came here for, start building an 
AI agent!

* [Next: Acceleration]({filename}torcs-ai-tuto-01.md)

