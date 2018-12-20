**Status:** Maintenance (expect bug fixes and minor updates)


NEWS
====

**2018 July 9**

Hi everyone! We wanted to let you know why we haven't pushed changes into this repo for a while: we're working on roboschool2, a
new codebase with different priorities, which we hope will further accelerate robotics research. We continue to recommend the
use of roboschool1 Hopper, Ant, Humanoid and Flagrun for evaluation and testing of algorithms. If you have fixes to make
installation easier we'll be happy to merge it. We'll have more to share about roboschool2 in a while.


**2017 July 17, Version 1.1**

* All envs version bumped to “-v1", due to stronger stuck joint punishment, that improves odds of getting a good policy.
* Flagrun-v1 is much more likely to develop a symmetric gait,
* FlagrunHarder-v1 has new "repeat-underlearned" learning schedule, that allows it to be trained to stand up, walk and turn without falling. 
* Atlas robot model, modified (empty links removed, overly powerful feet weakaned).
* All -v1 envs are shipped with better zoo policies, compared to May versions.
* Keyboard-controlled humanoid included.


Roboschool
==========

Release blog post is here:

https://blog.openai.com/roboschool/


Roboschool is a long-term project to create simulations useful for research. The roadmap is as follows:

1. Replicate Gym MuJoCo environments.
2. Take a step away from trajectory-centric fragile MuJoCo tasks.
3. Explore multiplayer games.
4. Create tasks with camera RGB image and joints in a tuple.
5. Teach robots to follow commands, including verbal commands.


Some wiki pages:

[Contributing New Environments](https://github.com/openai/roboschool/wiki/Contributing-New-Environments)

[Help Wanted](https://github.com/openai/roboschool/wiki/Help-Wanted)



Environments List
=================

The list of Roboschool environments is as follows:

- RoboschoolInvertedPendulum-v0
- RoboschoolInvertedPendulumSwingup-v0
- RoboschoolInvertedDoublePendulum-v0
- RoboschoolReacher-v0
- RoboschoolHopper-v0
- RoboschoolWalker2d-v0
- RoboschoolHalfCheetah-v0
- RoboschoolAnt-v0
- RoboschoolHumanoid-v0
- RoboschoolHumanoidFlagrun-v0
- RoboschoolHumanoidFlagrunHarder-v0
- RoboschoolPong-v0

To obtain this list: `import roboschool, gym; print("\n".join(['- ' + spec.id for spec in gym.envs.registry.all() if spec.id.startswith('Roboschool')]))`.


Basic prerequisites
===================
Roboschool is compatible and tested with python3 (3.5 and 3.6), osx and linux. You may be able to compile it with python2.7 (see Installation from source),
but that may require non-trivial amount of work. 

Installation
============

If you are running Ubuntu or Debian Linux, or OS X, the easiest path to install roboschool is via pip (:
```bash
pip install roboschool
```
Note: in a headless machine (e.g. docker container) you may need to install graphics libraries; this can be achieved via `apt-get install libgl1-mesa-dev`

If you are running some other Linux/Unix distro, or want the latest and the greatest code, or want to tweak the compiler optimization options, read on...

Installation from source
========================

Prerequisites
-------------
First, make sure you are installing from a github repo (not a source package on pypi). That is, clone this repo and cd into cloned folder:
```bash
git clone https://github.com/openai/roboschool && cd roboschool
```

The system-level dependencies of roboschool are qt5 (with opengl), boost-python3 (or boost-python if you are compiling with python2), and assimp. Also, 
some version of graphics libraries is required. 
Qt5 and assimp are rather straightforward to install:

- Ubuntu / Debian: 

    ```bash
    sudo apt-get install qtbase5 libqt5opengl5-dev libassimp-dev
    ```

- OSX:
    
    ```bash
    brew install qt assimp
    ```

Next, we'll need boost-python3. On osx `brew install boost-python3` is usually sufficient, however, on linux it is not always available as a system-level package. 
Do we despair? Of course not! We install it from source! There is a script `install_boost.sh` that should do most of the heavy lifting - note that it will need sudo
to install boost-python3 after compilation is done. 

Next, need a custom version of bullet physics engine. In both osx and linux its installation is a little involved, fortunately, there is a 
helper script `install_bullet.sh` that should do it for you. 
Finally, we also need to set up some environment variables (so that pkg-config knows where has the software been installed) - this can be done via sourcing `exports.sh` script

To summarize, all the prerequisites can be installed as follows:
- Ubuntu / Debian: 

    ```bash
    sudo apt-get install qtbase5 libqt5opengl5-dev libassimp-dev
    sudo ./install_boost.sh
    ./install_bullet.sh
    . exports.sh
    ```

- OSX:
    
    ```bash
    . exports.sh
    brew install qt assimp boost-python3
    ./install_bullet.sh
    . exports.sh
    ```
To check if that installation is successful, run `pkg-config --cflags Qt5OpenGL assimp bullet` - you should see something resembling compiler options and not 
an error message. Now we are ready to compile the bullet project itself. 

Compile and install
-------------------
The compiler options are configured in the [Makefile](roboschool/cpp-household/Makefile). Feel free to tinker with them or leave those as is. To
compile the project code, and then install it as a python package, use the following:
```bash
cd roboschool/cpp-household && make -j4 && cd ../.. && pip install -e .
```

A simple check if resulting installation is valid:
```python
import roboschool
import gym

env = gym.make('RoboschoolAnt-v1')
while True:
    env.step(env.action_space.sample())
    env.render()
```
You can also check the installation running a pretrained agent from the agent zoo, for instance:
```bash
python agent_zoo/RoboschoolHumanoidFlagrun_v0_2017may.py
```

Troubleshooting
---------------
A lot of the issues during installation from source are due to missing / incorrect PKG_CONFIG_PATH variable.
If the command `pkg-config --cflags Qt5OpenGL assimp bullet` shows an error, you can try manually finding missing `*.pc` files (for instance, for if the `pkg-config` complains about assimp, run `find / -name "assimp.pc"` - this is a bit bruteforce, but it works :)) and then adding folder with that files to PKG_CONFIG_PATH. 

Sometime distros of linux may complain about generated code being not platform-independent, and ask you to recompile something with `-fPIC` option (this was seen on older versions of CentOS). In that case, try removing `-march=native` compilation option in the Makefile. 

Agent Zoo
=========

We have provided a number of pre-trained agents in the `agent_zoo` directory.

To see a humanoid run towards a random varying target:

```bash
python $ROBOSCHOOL_PATH/agent_zoo/RoboschoolHumanoidFlagrun_v0_2017may.py
```

To see three agents in a race:

```bash
python $ROBOSCHOOL_PATH/agent_zoo/demo_race2.py
```
