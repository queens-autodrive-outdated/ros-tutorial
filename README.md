# ROS Tutorial

This is the ROS tutorial that was run internally for the Queen's AutoDrive team.

## What is ROS? Why do we use it?
ROS is considered a *meta* operating system, which provides typical services you would expect from an operating system such as hardware abstraction, device control, and message passing between processes. ROS is a distributed framework of processes (nodes) that allow code to be loosely coupled at runtime. 

ROS makes it easy to coordinate the operation of complex systems, and has lots of useful tooling to help development. 


## ROS Installation
Information about ROS noetic installation can be found [here](http://wiki.ros.org/noetic/Installation).

## ROS Workspace (Catkin)
ROS uses [catkin workspaces](http://wiki.ros.org/catkin/workspaces), which is a folder to build and run packages. 

### Folder Structure
Within the catkin workspace, you'll typically see the following folder structure: 
```
workspace_folder/         -- CATKIN WORKSPACE
  src/                    -- SOURCE SPACE
    CMakeLists.txt        -- The 'toplevel' CMake file
    package_1/            -- Package directories  
      CMakeLists.txt
      package.xml
      ...
    package_n/
      CATKIN_IGNORE       -- Optional empty file to exclude package_n from being processed
      CMakeLists.txt
      package.xml
      ...
  build/                  -- BUILD SPACE
    CATKIN_IGNORE         -- Keeps catkin from walking this directory
  devel/                  -- DEVELOPMENT SPACE (set by CATKIN_DEVEL_PREFIX)
    bin/
    etc/
    include/
    lib/
    share/
    .catkin
    env.bash
    setup.bash
    setup.sh
    ...
  install/                -- INSTALL SPACE (set by CMAKE_INSTALL_PREFIX)
    bin/
    etc/
    include/
    lib/
    share/
    .catkin             
    env.bash
    setup.bash
    setup.sh
    ...
```

#### source
The source (src) folder conatins the source code of the catkin packages. All ROS packages that are going to be used go in here. 

#### build
This is where catkin builds the packages.

#### devel
This stands for development, and is where packages go during testing. This directory should always be a sibling to the build space

#### install
This is where built targets get installed. This is usually the last step after testing has occured. 

### Building the workspace
The `catkin_make` command is used to build the workspace packages. This will populate the files in the `devel` folder. 

After the make command has been done, you have to source the setup:
```
source devel/setup.bash
```

Now, your system has built the ROS packages and is ready for use!



## ROS Packages
Packages are a core idea in ROS, and allow software to be developed in an isolated way. At a minimum, a package has the following files:
- `package.xml` - This defines a catkin package, and provides meta information about the package like the name, authors, dependencies...etc. 
- `CMakeLists.txt` - This is a boilerplate CMake file which uses catkin. 

The `catkin_create_pkg` script is used to create new packages. For example:
```
cd ~/catkin_ws
catkin_create_pkg tutorial1 rospy std_msgs
```
will create a package, tutorial1, which depends on rospy and std_msgs. 

After creating the new package, it can be build. 
```
cd ~/catkin_ws
catkin_make
```
 

## ROS Nodes
ROS nodes are individual executable pieces of code that communicate with other nodes. Nodes can publish information, or subscribe to information where it makes sense. 

An example would be a simple camera and computer vision algorithm. Let's say we were building a simple object detection system. 

In this case, there would be two nodes:
- Camera Node (camera_node.py) - this would be responsible for streaming camera frames to a topic
- Object Detector Node (detector_node.py) - this would be responsible for listening to the incoming camera frames, and performing the detections.

### Building a ROS Node
Let's build it!

### Useful Node Commands
There are some useful commands when interacting with nodes:
- `roscore` - this will run the ROS system.
- `rosrun` - this will run a specified ROS node.
- `rosnode list` will give a list of all of the nodes that are running.

### Running the ROS Node
To run the ROS node, we will need 3 separate terminals.

The first terminal will be used to run roscore:
```
roscore
```

To run the nodes, we're going to use `rosrun`.
The syntax for it is: 
```
rosrun [package_name] [node_name]
```

The second terminal will be used to run camera_node:
```
rosrun tutorial1 camera_node.py
```

The third terminal will be used to run detector_node:
```
rosrun tutorial1 detector_node.py
```

### Visualizing the data
Often times, we want to visualize what the ROS system is doing. RVIZ is a tool built into ROS that allows you to easily setup a GUI for your system. 
```
rosrun rviz rviz
```


## ROS Topics
The next topic is ROS topics (ha) which are basically the channels where information flows between. The concept behind topics is a publish/subscriber architecture (pubsub), which is a commonly used pattern in many software applications. This helps decouple software from eachother. 

### Useful ROS Topic Commands:
- `rostopic list` - This lists all of the topics currently in the system.
- `rostopic echo [topic]` - This shows the data thats being published to a given topic. 
- `rostopic hz [topic]` - This reports the rate at which data is being published to a topic. 

### ROS Messages
The communication that happens on these topics are what we call ROS messages. Each topic will have a specified type for the information being published to the topic. This makes it possible for nodes to serialize the data from the topic. 

There are a lot of built in ROS message types that are useful for most things. Try to use built-in message structures before writing custom ones. 

### Building a custom message type
Often times, custom message types are required for complex systems. For the example of the object detection system, we will need a custom message type to publish the object detections in a way that makes sense. There are a few steps to adding a custom message type:

1. Create a msg folder and a `.msg` file within the package.
```
roscd tutorial1 # navigate to the package
mkdir msg       # create the msg folder
echo "float32 classification" > msg/Detection.msg
```

This created a simple msg structure that has a float for the classification. 

2. Add the following lines in `package.xml`:
```
<build_depend>message_generation</build_depend>
<exec_depend>message_runtime</exec_depend>
```

3. Edit CMakeLists.txt:
There a few places that cmake needs to be updated:
First, we need to add the message_generation dependency to find_package.

Next, we need to export the message_runtime dependency in `catkin_package()`

Next, we need to add our message files in the `add_message_files()` call. 

Lastly, we need to uncomment the `generate_messages()` call so that it gets built. Now we should see our messages get generated when the package is built!

## Launch Files
So far we've only talked about ROS Nodes on an individual basis. We have different nodes, and we can run each of them in its own terminal.

What happens when the system has a lot of nodes, to the point that it's no longer feasible to have a terminal for each node?

This is where launch files come in. A launch file is a way to specify how ROS should launch, including which nodes to spawn, and pass arguments into the nodes. Usually, a package will implement multiple launch files for different purposes. 

## Recording and playing data
The next important piece of information is how to record and playback data in the ROS system. ROS uses something called a bag file to store information. Basically, a bag file is just a recording of specified rostopics. 

To record the system:
```
rosbag record [topic1] [topic2] ...
```

This will record a bag file with the streams from the given topics. 

To playback the recording:
``` 
rosbag play <bagfile>
```

This will start publishing the data from the recording. You can then launch other ROS nodes to interact with the recording.  
