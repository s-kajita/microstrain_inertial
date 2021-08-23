## Description

Interface (driver) software, including ROS node, for inertial sensors compatible with the [Microstrain Communication Library (MSCL)](https://github.com/LORD-MicroStrain/MSCL).

MSCL is developed by [LORD Sensing - Microstrain](http://microstrain.com) in Williston, VT. 

## Different Codebases

This repo is now structured differently as of `2.0.0`.

#### Important Branches
There are three important branches that you may want to checkout:

* [master](https://github.com/LORD-MicroStrain/ROS-MSCL/tree/master) -- Contains the most recent ROS1 changes before the transition to `2.0.0`. Kept for backwards compatibility, but no longer updated or supported
* [main](https://github.com/LORD-MicroStrain/ROS-MSCL/tree/main) -- Contains ROS1 implementation for this node as of `2.0.0`. This version is being actively updated and supported
* [ros2](https://github.com/LORD-MicroStrain/ROS-MSCL/tree/ros2) -- Contains ROS2 implementation for this node as of `2.0.0`. This version is being actively updated and supported

Both the `main` and `ros2` branches share most of their code by using the [ROS-MSCL-Common](https://github.com/LORD-MicroStrain/ROS-MSCL-Common) submodule which is submoduled in this repo at [microstrain_common](./microstrain_common)

#### Different Package Names

Prior to version `2.0.0`, this repo contained the following ROS packages:
* `ros_mscl` -- ROS node that will communicate with the devices
* `mscl_msgs` -- Collection of messages produced by the `ros_mscl` node
* `ros_mscl_cpp_example` -- Simple subscriber written in C++ that will consume a message produced by `ros_mscl`
* `ros_mscl_py_example` -- Simple subscriber written in Python that will consume a message produced by `ros_mscl`

Due to requirements laid out by the ROS maintainers [here](https://www.ros.org/reps/rep-0144.html), as of version `2.0.0`, this repo contains the following ROS packages:
* `microstrain_inertial` -- ROS node that will communicate with the devices
* `microstrain_msgs` -- Collection of messages produces by the `microstrain_inertial` node
* `microstrain_examples` -- Collection of examples that show how to interact with the `microstrain_inertial` node. Currently contains one simple C++ and python subscriber node

## Build Instructions

#### Submoduels
This repo now takes advantage of git submodules in order to share code between ROS versions. When cloning the repo, you should clone with the `--recursive` flag to get all of the submodules.

If you have already cloned the repo, you can checkout the submodules by running `git submodule init && git submodule update --recursive` from the project directory

The [CMakeLists.txt](./microstrain_msgs/CMakeLists.txt) will automatically checkout the submodule if it does not exist, but it will not keep it up to date. In order to keep up to date, every
time you pull changes you should pull with the `--recurse-submodules` flag, or alternatively run `git submodule update --recursive` after you have pulled changes

#### MSCL
MSCL is now installed in the [CMakeLists.txt](./ros_mscl/CMakeLists.txt). The version installed can be changed by passing the flag `-DMSCL_VERSION="62.0.0"`

If you already have MSCL installed and want to use your installed version instead of the one automatically downloaded, you can specify the location by passing the flag `-DMSCL_DIR=/usr/share/c++-mscl`

We do our best to keep ROS-MSCL up-to-date with the latest MSCL changes, but sometimes there is a delay. The currently supported version of MSCL is [v62.0.0](https://github.com/LORD-MicroStrain/MSCL/releases/tag/v62.0.0)

#### Building from source
1. Install ROS and create a workspace: [Installing and Configuring Your ROS Environment](http://wiki.ros.org/ROS/Tutorials/InstallingandConfiguringROSEnvironment)

2. Move the entire ROS-MSCL folder (microstrain_inertial, microstrain_msgs , and microstrain_common for just source) to the your_workspace/src directory.

3. Locate and register the ros_mscl package: `rospack find microstrain_inertial`

4. Build your workspace:
        
        cd ~/your_workspace
        catkin_make
        source ~/your_workspace/devel/setup.bash
   The source command may need to be run in each terminal prior to launching a ROS node.
#### Launch the node and publish data
The following command will launch the driver. Keep in mind each instance needs to be run in a separate terminal.
            
        roslaunch microstrain_inertial microstrain.launch
Optional launch parameters:
- name: namespace the node will publish messages to, default: gx5
- port: serial port name to connect to the device over, default: /dev/ttyACM0
- baudrate: baud rate to open the connection with, default: 115200
- imu_rate: sample rate for IMU data (hz), default: 100
- debug: output debug info? default: false
- diagnostics: output diagnostic info? default: true
    
To check published topics:
        
    rostopic list

**Example**: Connect to and publish data from two devices simultaneously  
In two different terminals:
    
    roslaunch microstrain_inertial microstrain.launch name:=sensor1234

    roslaunch microstrain_inertial microstrain.launch name:=bestSensor port:=/dev/ttyACM1
This will launch two nodes that publish data to different namespaces:
- sensor1234, connected over port: /dev/ttyACM0
- bestSensor, connected over port: /dev/ttyACM1

An example subscriber node can be found here: [ROS-MSCL Examples](./microstrain_examples)  


## Docker Integration

### VSCode

The easiest way to use docker while still using an IDE is to use VSCode as an IDE. Follow the steps below to develop on this repo in a docker container

1. Install the following dependencies:
    1. [VSCode](https://code.visualstudio.com/)
    1. [Docker](https://docs.docker.com/get-docker/)
1. Open VSCode and install the following [plugins](https://code.visualstudio.com/docs/editor/extension-marketplace):
    1. [VSCode Docker plugin](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker)
    1. [VSCode Remote Containers plugin](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
1. Open this directory in a container by following [this guide](https://code.visualstudio.com/docs/remote/containers#_quick-start-open-an-existing-folder-in-a-container)
    1. Due to a bug in the remote container plugin, you will need to refresh the window once it comes up. To do this, type `Ctrl+Shift+p` and type `Reload Window` and hit enter. Note that this will have to be repeated every time the container is rebuilt
1. Once the folder is open in VSCode, you can build the project by running `Ctrl+Shift+B` to trigger a build, or `Ctrl+p` to open quick open, then type `task build` and hit enter
1. You can run the project by following [this guide](https://code.visualstudio.com/docs/editor/debugging)

### Make

If you are comfortable working from the command line, or want to produce runtime images, the [Makefile](./devcontainer/Makefile) in the [.devcontainer](./devcontainer) directory
can be used to build docker images, run a shell inside the docker images and produce a runtime image. Follow the steps below to setup your environment to use the `Makefile`

1. Install the following dependencies:
    1. [Make](https://www.gnu.org/software/make/)
    1. [Docker](https://docs.docker.com/get-docker/)
    1. [qemu-user-static](https://packages.ubuntu.com/bionic/qemu-user-static) (for multiarch builds)
        1. Run the following command to register the qemu binaries with docker: `docker run --rm --privileged multiarch/qemu-user-static:register`

The `Makefile` exposes the following tasks. They can all be run from the `.devcontainer` directory:
* `make build-shell` - Builds the docker image and starts a shell session in the image allowing the user to develop and build the ROS project using common commands such as `catkin_make`
* `make image` - Builds the runtim image that contains only the required dependencies and the ROS node. The resulting image is names `ros-mscl`
* `make clean` - Cleans up after the above two tasks

## License
ROS-MSCL is released under the MIT License - see the `LICENSE` file in the source distribution.

Copyright (c)  2021, Parker Hannifin Corp.

