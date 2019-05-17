# AS4SR/general_info

This repository was mainly set up for the existence and use of the associated wiki page.



ArduPilot: Instructions to set up and run an autopilot using SITL and Gazebo simulator


The SITL (software in the loop) simulator allows you to run Plane, Copter or Rover without any hardware. It is a build of the autopilot code using an ordinary C++ compiler, giving us a native executable that allows us to test the behaviour of the code without hardware.

To learn more about the ArduPilot project click this link: Working with the ArduPilot Project Code

The following guidlines have been tried and tested in Ubuntu 14.04. (Ubuntu 16.04 still has some compilation issues that we and others are tracking down.) Detailed instructions for setting up this autopilot in Linux can be found in this link: Setting up SITL on Linux

Table of Contents

    Download ArduPilot:
    Download JSBSim (Plane only)
    Install some required packages:
    Add some directories to your search path:
    Start SITL simulator:
            Potential errors and their solutions:
            Once any initial errors are resolved and the sim actually runs...
    Load a mission:
    Playing around with the ArduCopter:
    Gazebo Plugin for Arducopter:
    Connecting with ArduPilot with ROS:
    Summer of Innovation 2017 (AFRL):

Download ArduPilot:

Under the home directory in the terminal, run the following commands once to download the Ardupilot from github repository:

       git clone git://github.com/ArduPilot/ardupilot.git
       cd ardupilot
       git submodule update --init --recursive

Download JSBSim (Plane only)

To fly the fixed wing (Plane) simulator, download the JSBSim flight simulator from github:

       git clone git://github.com/tridge/jsbsim.git
       sudo apt-get install libtool libtool-bin automake autoconf libexpat1-dev

If you get an error message asking for a newer version of JSBSim when you try to run `sim_vehicle.py -w` below, update it by running the following commands:

       cd jsbsim
       git pull
       ./autogen.sh --enable-libraries
       make

Install some required packages:

Run these commands once:

       sudo apt-get install python-matplotlib python-serial python-wxgtk3.0 python-wxtools python-lxml
       sudo apt-get install python-scipy python-opencv ccache gawk git python-pip python-pexpect
       sudo pip2 install future pymavlink MAVProxy

If you get an error saying 'pip' is not installed, install it using: sudo apt-get install pip

Alternately, upgrade it via: `sudo pip2 install --upgrade pip`
Add some directories to your search path:

Open to edit the .bashrc file. Under home directory on the terminal run the following command:

       gedit .bashrc

Add the following lines to the end of your ”.bashrc” in the home directory:

       export PATH=$PATH:$HOME/jsbsim/src
       export PATH=$PATH:$HOME/ardupilot/Tools/autotest
       export PATH=/usr/lib/ccache:$PATH

Reload your PATH by using the “dot” command in a terminal:

       . ~/.bashrc

Start SITL simulator:

Start the simulator for the first time to load the right default parameters for the vehicle :

       cd ~/ardupilot/ArduPlane
       sim_vehicle.py -w

Potential errors and their solutions:

If you get an error to do with mavgen or mavlink when trying this, run the following then try again:

       sudo pip install --upgrade pymavlink
       sudo pip install --upgrade MAVProxy

If you get an error to do with 'from future import' not finding future, try: (ref: link, link)

       sudo pip install importlib
       sudo pip install unittest2
       sudo pip install argparse
       sudo pip2 install importlib
       sudo pip2 install unittest2
       sudo pip2 install argparse
       sudo apt-get install python-future

If you get an error with mavproxy 'missing parentheses in call to print', then you're having python 3 (vs python 2) issues. Edit the file to include the parentheses at line 944, 945, 980, and change line 12 from "Queue" to "queue", then try again:

       sudo gedit /usr/local/bin/mavproxy.py

If you get an error with loading MAVProxy, try installing the library locally, rather than system-wide: (link):

       pip install MAVProxy

and add the following to your ".bashrc" file:

       export PATH=$PATH:$HOME/.local/bin
       export PYTHONPATH=$PYTHONPATH:$HOME/.local/bin

Once any initial errors are resolved and the sim actually runs...

After loading the default parameters, kill the simulator using Ctrl-C. After that, start the simulator normally:

       sim_vehicle.py --console --map --aircraft test

Load a mission:

To load a test mission for the vehicle, in the same terminal that is running the simulator, run the following command once (you may need to hit enter to get a 'clean' `MANUAL>` prompt):

       wp load ../Tools/autotest/ArduPlane-Missions/CMAC-toff-loop.txt

For takeoff, run the following commands once under the same terminal:

       arm throttle
       mode auto

(Note that once you type `mode auto` that the prompt should now read `MANUAL> AUTO>`, not just `MANUAL>` anymore.)

The virtual plane should now take-off!! Close the simulator using Ctrl+C in the terminal when done.
Playing around with the ArduCopter:

MAVProxy is commonly used by developers to communicate with SITL. Run SITL with the following commands in a new terminal:

       cd ~/ardupilot/ArduCopter
       sim_vehicle.py -j4 --map --console

Send commands to SITL from the command prompt and observe the results on the map. Watch the altitude increase in console with the following commands. Note that takeoff must start within 15 seconds of arming (`arm throttle` command), or the motors will disarm (ignore the `takeoff 40` command).

       mode guided
       arm throttle
       takeoff 40

To keep constant altitude for circle mode, after we've hit altitude of 40m, try: (link -- this sets the throttle to 1500, you can undo this via `rc 3 1100`)

       rc 3 1500

Change to CIRCLE mode and set the radius to 2000cm and watch the copter circle on the map!!

       mode circle
       param set circle_radius 2000

To land the copter when done, set the mode to RTL:

       mode rtl

A number of common ArduPilot tasks to test the autopilot can be found here, here, and here.
Gazebo Plugin for Arducopter:

If you haven't installed gazebo yet, download it now using this command:

       sudo apt-get install libgazebo7-dev

Get the Gazebo plugin from github and build the workspace.

       git clone https://github.com/SwiftGust/ardupilot_gazebo
       cd ardupilot_gazebo
       mkdir build
       cd build
       cmake ..
       make -j4
       sudo make install

Set the path of Gazebo Models:

       echo 'export GAZEBO_MODEL_PATH=~/ardupilot_gazebo/gazebo_models' >> ~/.bashrc
       source ~/.bashrc

Copy Demo Worlds to Gazebo:

       sudo cp -a ~/ardupilot_gazebo/gazebo_worlds/. /usr/share/gazebo-7/worlds

Launch the ArduPilot SITL simulation in the terminal:

       sim_vehicle.py -v ArduCopter -f gazebo-iris  -m --mav10 --map --console -I0

On another terminal, launch Gazebo with demo 3DR Iris model:

       gazebo --verbose worlds/iris_irlock_demo.world

Done!! You can now send commands to the SITL like before and watch them simulate in Gazebo.
Connecting with ArduPilot with ROS:

Install MAVROS with the following command:

       sudo apt install ros-kinetic-mavros

Install RQT for simpler usage on a desktop computer:

       sudo apt-get install ros-kinetic-rqt ros-kinetic-rqt-common-plugins ros-kinetic-rqt-robot-plugins

Install catkin-tools:

       sudo apt-get install python-catkin-tools

Launch an SITL instance:

       sim_vehicle.py -v ArduCopter --console --map

The next step is to create a new directory for a launch file. In a new terminal, we run these:

       cd ardupilot
       mkdir launch
       cd launch

We then copy mavros default launch file for ardupilot and then open to edit it:

       roscp mavros apm.launch apm.launch
       gedit apm.launch

To connect to SITL we modify the first line to

       <arg name="fcu_url" default="udp://127.0.0.1:14551@14555" />

We save the file and launch it with:

       roslaunch apm.launch

The connection is now complete.

Troubleshooting: Since GeographicLib requires certain datasets (mainly the geoid dataset) so to fulfill certain calculations, these may be needed to be installed manually by the user. If you run into GeographicLib error, run these:

       wget https://raw.githubusercontent.com/mavlink/mavros/master/mavros/scripts/install_geographiclib_datasets.sh
       chmod +x install_geographiclib_datasets.sh
       sudo ./install_geographiclib_datasets.sh

Additionally, we can use RQT to see all the topics that mavros has to create from ardupilot information. In a new terminal, type:

       rqt

We can now see all the topics that mavros has to create from ardupilot information.
Summer of Innovation 2017 (AFRL):

For the summer, a common goal for the AS4SR group was to set up a toolchain that would allow AFRL's Unmanned Systems Autonomy Services (UxAS) software to be used in conjunction with ROS. Subsequently, we developed a ROS node that can automatically receive waypoints from UxAS and perform waypoint navigation in simulated environment. All the code along with documented instructions can be found in this github repository: soi_waypoint_work
Pages 20

    ## Home ##
    ## Personnel, AS4SR Lab ##
    (Some) Robots in ROS plus Gazebo!
    ArduPilot: Instructions to set up and run an autopilot using SITL and Gazebo simulator
    AS4SR Wiki Formatting Guidelines...
    Basic ROS MoveIt! and Gazebo Integration
    Code and IDE things (Eclipse, Java)
    Creating Heightmaps for Gazebo
    Creating robot models in Gazebo (beyond URDF files)
    git use and best practices
    Installing AMDGPU PRO driver for Ubuntu 16.04.3 for VR use
    Instructions for installing ROS and Gazebo!
    IRB Information
    Listing of other UAV flight simulators that use Gazebo and related work
    Miscellaneous links to be sorted...

Clone this wiki locally
