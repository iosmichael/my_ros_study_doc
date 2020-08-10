# ROS 2 Study Guide

**ROS Terminology** (Filesystem Structure):

-  *Packages*: Nodes, ROS-dependent library, configuration files, organizing files
- *Metapackages*
- *Package Manifests*: Manifests ```package.xml``` provide metadata about a package
  - name, version, description
  - license information, dependencies
- *Repositories*: a collection of packages share a common VCS system
- *Message (msg) types*: define the data structures for messages sent in ROS ```my_package/msg/MyMessageType.msg```
- *Service (srv) types*: define the request and response data structures for services in ROS ```my_package/srv/MyServiceType.msg```

**ROS Terminology** (Computation Graph Structure):

- *Nodes*: processes that perform computation, written use `rospy` or `roscpp`

- *Master*: provides name registration, look up to the rest of the computation graph

- *Parameter Server*: stores shared key in a central location

- *Messages*: a data structure for communication between nodes

- *Topics*: routed via a transport system with publish/subscribe semantics. A topic is a strongly typed message bus, each bus has a name, and anyone can connect to the bus to send or receive messages as long as they are the right type

- *Services*: request and reply is done via services, which are defined by a pair of message structures (one for request and one for reply)

-  *Bags*: format for saving and playing back ROS message data. Important mechanism for storing data, such as sensor data, that can be difficult to collect but is necessary for developing and testing algorithms.

  

![http://ros.org/images/wiki/ROS_basic_concepts.png](http://ros.org/images/wiki/ROS_basic_concepts.png)



## Run ROS2

`Docker Image: iosmichael/ros2-foxy:1.0`

Set Environment Variable by

```bash
source ~/ros2_foxy/install/setup.bash
source /opt/ros/foxy/setup.bash
export ROS_DOMAIN_ID=120 # 120 is just a random number here, can be anything from 0 to 232
```

Check ROS Environment Variable by

```bash
printenv | grep -i ROS
```



## GUI Application with Docker:

Our goal is to connect the docker container as a new **X client** to our current **X server**. First, we need to open up the container ability to access our X server. This is a very good article giving a brief [introduction of X client-server architecture in Linux.](https://medium.com/mindorks/x-server-client-what-the-hell-305bd0dc857f) It is also worth noting that some of the instructions below might not be essential and even redundant for some system setting, but it is the most comprehensive and safe settings I can search on the Internet. 

```bash
sudo usermod -aG docker $USER
sudo chmod 666 /var/run/docker.sock # allow socket connection
sudo xhost +local:docker # permits the root user on the local machine to connect to X windows display.
```

[The two or three things we need to connect with X server is](https://unix.stackexchange.com/questions/118811/why-cant-i-run-gui-apps-from-root-no-protocol-specified)

1. X client needs to know where is X server (`$DISPLAY`)
2. X client needs have access to X server (`$XAUTHORITY`)
3. X client needs to be able to establish back and forth connection with X server (`$XSOCK`)

```bash
XSOCK=/tmp/.X11-unix/
XAUTH=/tmp/.docker.xauth
sudo touch $XAUTH
```

Once we setup the above steps, we can then run our docker container with the following:

```bash
docker run -it --network=host --security-opt apparmor=unconfined -v $XSOCK:$XSOCK:rw -v $XAUTH:$XAUTH:rw --device /dev/dri --privileged -e DISPLAY=$DISPLAY -e XAUTHORITY=$XAUTH iosmichael/ros2:1.1
```

The command options are:

1. `-it` is to enable interactive mode `-i` and set up a pseudo-TTY upon starting the container (optional) `-t`
2. `--network=host` host networking, the container is not launched with an isolated network stack. Instead, all networking interfaces of the host are directly accessible inside the container.
3. `--security-opt` sometimes X client messages can be blocked by [docker profile](https://cloud.google.com/container-optimized-os/docs/how-to/secure-apparmor), [setting **apparmor=**`unconfined`](https://github.com/moby/moby/issues/38442#issuecomment-450571773) gives superuser permission when forwarding X message.
4. `-v` is to share files between host machine and docker containers. Here we need to link both the X socket file and X authority file to the docker container `-v $XSOCK:$XSOCK:rw -v $XAUTH:$XAUTH:rw`
5. `--device /dev/dri` allows our container to access the physical graphic cards from our local computer for graphing purposes
6. `--privileged` (optional) just to grant higher access to the usage of the physical graphic cards
7. `-e` is to share environment variable with the container, here we want to notify the container where is the X server `$DISPLAY`(":0") and what form of access it requires `$XAUTHORITY`
8. `iosmichael/ros2:1.1` is the image that has all the **ros2** demo installed and built. 

[The above instructions is a modified version of a docker run file for Qt docker application.](https://github.com/thewtex/docker-opengl-mesa/blob/master/run.sh)

### Accessing into Docker Container

To access the container, we just created, we need to first find the container id:

```bash
docker container list # to obtain container id
```

then we need to access the container based on the **id**

```bash
docker exec -it <container-id> bash
```

To ensure your GUI application inside Docker works properly, we need to pass two tests:

1. Connection from X client to X server (called X forwarding)
2. GUI application's ability to access GL support (can the program communicate with the **graphic interface**?)

### Test X server Connection

**apt-get install** **-qqy** `x11-apps`, this will give you simple apps like `xeyes` and `xterm`. Run **xeyes** with,

```bash
root@iosmichael-AERO-15XV8/ xeyes
```

If X server is setup correctly, we should see a Eye being graphed on our display

<img src="/home/iosmichael/Documents/ros/docs/imgs/Xeyes.png" alt="w" style="zoom: 33%;" />

If you experience with connection error to X server at $DISPLAY, you need to double check the environment variables `$DISPLAY` and `$XAUTHORITY` .

### Test GL Support for GUI Application

**apt install** `x-window-system`and `mesa-utils`, this will give you command to test GUI support like `glxgears` and `glxinfo`

[If you see errors below, when run `glxgears`](https://github.com/jessfraz/dockerfiles/issues/253#issuecomment-337473051). [I also run **apt install** `-y calibre` from a post suggestion, but not sure if it matters in this case](https://bugreports.qt.io/browse/QTBUG-70447)

```bash
root@iosmichael-AERO-15XV8: glxgears 
libGL error: No matching fbConfigs or visuals found
libGL error: failed to load driver: swrast
X Error of failed request:  GLXBadContext
  Major opcode of failed request:  155 (GLX)
  Minor opcode of failed request:  6 (X_GLXIsDirect)
  Serial number of failed request:  49
  Current serial number in output stream:  48
```

This is probably because GL is not supported by the right driver. To solve this, you need to 

1. link the physical driver device to the docker
2. [install the right library dependencies for your graphic cards](https://github.com/jessfraz/dockerfiles/issues/253#issuecomment-373043685). for me, its `apt install libnvidia-gl-390`

once this is done, you should be able to see `glxgears` correctly running in your X server

<img src="/home/iosmichael/Documents/ros/docs/imgs/gears.png" style="zoom: 25%;" />

And `glxinfo` should look like this:

```bash
OpenGL ES profile version string: OpenGL ES 3.2 NVIDIA 390.138
OpenGL ES profile shading language version string: OpenGL ES GLSL ES 3.20
OpenGL ES profile extensions:
    GL_ANDROID_extension_pack_es31a, GL_EXT_base_instance, 
    GL_EXT_blend_func_extended, GL_EXT_blend_minmax, GL_EXT_buffer_storage,
    ...
```



### Example 1. Action Client and Server

Run **examples_rclcpp_minimal_action_client** and **examples_rclcpp_minimal_action_server**

Terminal 1 (Server Node):

```bash
mkdir -p ~/workspace/example_action_client_server
cd ~/workspace/example_action_client_server
# source the environment
. ~/ros2_foxy/install/setup.bash
# run the server side
RMW_IMPLEMENTATION=rmw_cyclonedds_cpp ros2 run examples_rclcpp_minimal_action_server action_server_not_composable
```

Terminal 2 (Client Node):

```bash
cd ~/workspace/example_action_client_server
# source the environment
. ~/ros2_foxy/install/setup.bash
# run the server side
RMW_IMPLEMENTATION=rmw_cyclonedds_cpp ros2 run examples_rclcpp_minimal_action_client action_client_not_composable
```

Result:

![action_server_client](/home/iosmichael/Documents/ros/docs/imgs/action_server_client.png)

## Example 2 [Build from Workspace](https://colcon.readthedocs.io/en/released/user/quick-start.html)

Unfortunately, native CMake-Tools from VS Code doesn't work with ROS2 project. To build ROS2, you need to use **colcon** compilation procedure. Before you do so, you need to first set up the environment configuration. (you can add these command lines to `~/.bashrc`, so you don't have to setup environment variables every time you open a new terminal window)

```bash
source /opt/ros/foxy/setup.bash
export ROS_DOMAIN_ID="a random number from 0-232"
```

1. Click on VS Code **"Files -> Add Folder to Workspace"** and select the path to **"examples/rclcpp/actions/minimal_action_client"**

2. A typical ROS 2 package will have

   1. "CMakeLists.txt", configuration file that uses "ament_cmake" to compile ROS 2 dependencies

      ```cmake
      cmake_minimum_required(VERSION 3.5)
      project(examples_rclcpp_minimal_action_server)
      
      # Default to C++14
      if(NOT CMAKE_CXX_STANDARD)
        set(CMAKE_CXX_STANDARD 14)
      endif()
      
      if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        add_compile_options(-Wall -Wextra -Wpedantic)
      endif()
      
      find_package(ament_cmake REQUIRED)
      find_package(example_interfaces REQUIRED)
      find_package(rclcpp REQUIRED)
      find_package(rclcpp_action REQUIRED)
      
      add_executable(action_server_member_functions member_functions.cpp)
      ament_target_dependencies(action_server_member_functions
        "rclcpp"
        "rclcpp_action"
        "example_interfaces")
        
      add_executable(action_server_not_composable not_composable.cpp)
      ament_target_dependencies(action_server_not_composable
        "rclcpp"
        "rclcpp_action"
        "example_interfaces")
      
      if(BUILD_TESTING)
        find_package(ament_lint_auto REQUIRED)
        ament_lint_auto_find_test_dependencies()
      endif()
      
      install(TARGETS
        action_server_not_composable
        action_server_member_functions
        DESTINATION lib/${PROJECT_NAME})
      
      ament_package()
      ```

   2. "package.xml", configuration file for package metadata, includes description of required dependencies

      ```xml
      <?xml version="1.0"?>
      <?xml-model href="http://download.ros.org/schema/package_format2.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
      <package format="2">
        <name>examples_rclcpp_minimal_action_server</name>
        <version>0.9.2</version>
        <description>Minimal action server examples</description>
        <maintainer email="jacob@openrobotics.org">Jacob Perron</maintainer>
        <license>Apache License 2.0</license>
        <author email="jacob@openrobotics.org">Jacob Perron</author>
      
        <buildtool_depend>ament_cmake</buildtool_depend>
      
        <depend>example_interfaces</depend>
        <depend>rclcpp</depend>
        <depend>rclcpp_action</depend>
      
        <test_depend>ament_lint_auto</test_depend>
        <test_depend>ament_lint_common</test_depend>
      
        <export>
          <build_type>ament_cmake</build_type>
        </export>
      </package>
      
      ```

   3. To compile the project, run 

      ```bash
      colcon build # to build the package at the workspace
      
      . install/local_setup.bash # to setup environment variable for the package
      
      # run executables
      ./build/examplles_rclcpp_minimal_action_server/action_server_member_functions
      ```

   



Other link to explore

1. [How to setup ros with python3](https://medium.com/@beta_b0t/how-to-setup-ros-with-python-3-44a69ca36674)
2. 