# ROS-101 Part 1: Foundations

> The guide I wish I had before starting ROS.

---

# Introduction

Welcome to ROS-101.

Less than two months ago, I had never touched ROS. Terms like nodes, topics, TF trees, DDS, QoS, and localization sounded like a completely different language. The first few days were spent wondering why topics weren't appearing, why RViz was empty, and why robots seemed to stop working for no obvious reason.

This guide is written from the perspective of someone who learned ROS from scratch and wants to explain it in the simplest way possible.

I have divided this into 3 parts focusing on various ROS concepts. 

Part 1 focuses on the fundamental concepts that every ROS user should understand before diving into robotics applications.

By the end of this guide, you should understand:

* What ROS is
* Why ROS exists
* ROS 1 vs ROS 2
* Linux and ROS
* Nodes
* Topics
* Services
* Actions
* Parameters
* Launch Files
* Workspaces
* Packages

---

# What is ROS?

ROS stands for **Robot Operating System**.

Despite its name, ROS is **not actually an operating system**.

ROS is a middleware framework that sits between your hardware and your software.

Think of ROS as a communication layer that allows different parts of a robot to talk to each other.

For example, a robot may contain:

* Cameras
* LiDAR sensors
* IMUs
* GPS modules
* Motor drivers
* Navigation systems
* AI algorithms

Each of these components needs to exchange information with the others.

Without ROS, every developer would need to build their own communication system from scratch.

ROS provides a standardized framework so that all these components can work together.

---

# Why Was ROS Created?

Imagine building a robot without ROS.

Your camera would need custom code to communicate with your navigation software.
Your navigation software would need custom code to communicate with your motor controller.
Your motor controller would need custom code to communicate with your sensors.

The system quickly becomes difficult to maintain.


ROS solves this by providing a common framework for communication.

This allows developers to focus on building robots instead of building communication infrastructure.

ROS also provides:

* Standardized message types
* Visualization tools
* Navigation frameworks
* Simulation support
* Hardware abstraction
* Package management

---

# ROS 1 vs ROS 2

ROS 1 was released in 2010 and became the standard framework for robotics research.

However, as robotics systems became larger and more complex, several limitations became apparent.

ROS 2 was developed to address these limitations.

## Major Differences

| Feature              | ROS 1      | ROS 2              |
| -------------------- | ---------- | ------------------ |
| Communication        | TCPROS     | DDS                |
| Multi-Robot Support  | Limited    | Native             |
| Security             | Minimal    | Supported          |
| Real-Time Capability | Poor       | Improved           |
| QoS Support          | No         | Yes                |
| Lifecycle Nodes      | No         | Yes                |
| Long-Term Future     | Deprecated | Active Development |

Today, almost all new robotics projects use ROS 2. All ROS1 versions have reached End-of-Life.

---

# Linux and ROS

Although ROS 2 supports Windows and macOS, Linux remains the preferred platform.

Most ROS development happens on Ubuntu Linux because:

* Better package support
* Better driver support
* Better community support
* Easier debugging
* Better compatibility with robotics hardware

Popular combinations include:

| Ubuntu Version | ROS Distribution |
| -------------- | ---------------- |
| Ubuntu 22.04   | Humble           |
| Ubuntu 24.04   | Jazzy            |

If you are learning ROS for the first time, Ubuntu is strongly recommended.

---

# Installing ROS

You can find standard installation guide on ROS Wiki. Newer verions of ROS2 such as Jazzy and Humble are preferable. 

Before installing ROS, always check your Ubuntu version.

```bash
lsb_release -a
```

The Ubuntu version determines which ROS distribution you should install.

For Ubuntu 24.04:

```text
ROS 2 Jazzy Jalisco
```

is the recommended distribution.

After installation, verify ROS is working:

```bash
ros2 --help
```

---

# Sourcing ROS

One of the most common beginner mistakes is forgetting to source ROS.

Before using ROS commands, source the ROS environment:

```bash
source /opt/ros/jazzy/setup.bash
```

Without sourcing, your terminal will not know where ROS is installed.

You can add this command to your `.bashrc` file if desired.

---

# ROS Architecture

ROS applications are built using several core concepts:

```text
Nodes
Topics
Services
Actions
Parameters
```

Understanding these concepts is essential.

---

# Nodes

Nodes are the fundamental building blocks of every ROS system.

A node is simply an executable process that performs a specific task.

What exactly is a process?

When you run a program, the operating system creates a process.

For example:

firefox

creates a Firefox process.

Similarly:

```bash
ros2 run my_package my_node
```

creates a ROS node process.

Instead of having one giant program responsible for everything on a robot, ROS breaks the robot into many smaller programs, each with a well-defined responsibility.

An easy way to think about a node is as a worker in a factory. Every worker has a specific job, does that job well, and communicates with other workers when necessary.


Examples include:

* Camera Driver
* LiDAR Driver
* Localization Node
* Navigation Node
* Motor Controller Node

These nodes rarely work in isolation. Instead, they exchange information and work together to achieve a larger objective.

Consider a localization pipeline:

```text
LiDAR Sensor
      ↓
LiDAR Node
      ↓
Laser Scan Data
      ↓
Localization Node
      ↓
Robot Pose Estimate
```

The LiDAR node simply publishes scan data.

The localization node subscribes to that data and estimates the robot's position.

This modular design makes systems easier to develop, debug, and maintain.

View running nodes:

```bash
ros2 node list
```

Inspect a node:

```bash
ros2 node info <node_name>
```



---

# Topics

Topics are communication channels used by nodes.

A node can publish information to a topic.

Other nodes can subscribe to that topic.

Example:

```text
Camera Node
      ↓
   /image
      ↓
Object Detection Node
```

The camera node publishes images.

The object detection node subscribes to those images.

This communication is asynchronous.

The publisher does not need to know who is receiving the data.

Useful commands:

```bash
ros2 topic list
```

```bash
ros2 topic echo <topic_name>
```

```bash
ros2 topic info <topic_name>
```
**Common Beginner Questions**

1. Where does a topic actually exist?

This confused me a lot when I first learned ROS.

Topics are not files. Topics are not databases. Topics are not servers.

A topic is simply a communication channel managed by the middleware.

Think of a topic as a radio frequency.

Publishers broadcast. Subscribers listen.

The topic itself does not permanently store data.

Can multiple nodes publish to the same topic?

Technically yes.

However, this is usually avoided because subscribers may not know which publisher produced which message.

Most topics have a single publisher.

2. Can multiple nodes subscribe to the same topic?

Absolutely.

This is one of ROS's greatest strengths.

Example:

Camera
   |
 /image
   |
   ├── Object Detector
   ├── SLAM System
   ├── Logger
   └── Visualization Tool
   
One publisher can support many subscribers simultaneously.

3. What happens if nobody subscribes?

The publisher continues publishing.
The messages simply go nowhere.

4. How fast are topics published?

Depends entirely on the application.

Examples:

Camera Images      30 Hz
LiDAR              10 Hz
IMU               100 Hz
GPS                 1 Hz

Hz means messages per second.
---

# Services

Services provide request-response communication.

Unlike topics, services are used when a node needs an immediate answer.

Example:

```text
Client
   |
Request
   |
Server
   |
Response
```

Think of a service like asking a question and waiting for an answer.

Examples:

* Resetting a sensor
* Requesting a configuration value
* Triggering a calibration process

**Common Beginner Questions**

1. When should I use a Service instead of a Topic?

Use Topics when data is continuously flowing.

Examples:

Camera Images
LiDAR Scans
Odometry

Use Services when you need an answer.

Examples:

Reset Sensor
Calibrate Camera
Load Configuration

A simple rule:

Topics stream.

Services answer.
---

# Actions

Actions are used for long-running tasks.

Unlike services, actions can provide continuous feedback while running.

Examples:

* Navigating to a goal
* Docking
* Undocking
* Executing a trajectory

An action can:

* Start execution
* Provide progress updates
* Be cancelled
* Return a result

Think of actions as a service that takes a long time to finish.

---

# Parameters

Parameters are configuration values associated with a node.

Examples:

```text
Controller Frequency
Maximum Velocity
Sensor Range
Map File Path
```

Instead of modifying source code every time a setting changes, parameters allow behavior to be configured externally.

List parameters:

```bash
ros2 param list
```

Get parameter value:

```bash
ros2 param get <node_name> <parameter_name>
```

Set parameter value:

```bash
ros2 param set <node_name> <parameter_name> <value>
```

---

# Launch Files

Robots often require many nodes to start simultaneously.

Launching every node manually becomes impractical.

Launch files automate this process.

Instead of running:

```bash
Node A
Node B
Node C
Node D
```

individually, a launch file starts everything together.

Launch files are commonly written in Python.

Example:

```bash
ros2 launch <package_name> <launch_file>
```

---

# Packages

Packages are the basic organizational unit in ROS.

A package may contain:

* Source code
* Launch files
* Configuration files
* URDF models
* Documentation

Examples:

```text
navigation_package
camera_driver_package
lidar_driver_package
```

Packages allow code to be reused and shared.

List installed packages:

```bash
ros2 pkg list
```

---

# Workspaces

A workspace is a directory where ROS packages are developed.

Typical structure:

```text
my_workspace/

├── src
├── build
├── install
└── log
```

### src

Contains source code and packages.

### build

Contains compilation outputs.

### install

Contains generated executables and setup files.

### log

Contains build logs.

Build a workspace:

```bash
colcon build
```

After building:

```bash
source install/setup.bash
```

This step is extremely important.

Without sourcing the workspace, ROS cannot find your newly built packages.

---

# Useful Beginner Commands

List nodes:

```bash
ros2 node list
```

List topics:

```bash
ros2 topic list
```

Inspect node:

```bash
ros2 node info <node_name>
```

Inspect topic:

```bash
ros2 topic info <topic_name>
```

Echo topic:

```bash
ros2 topic echo <topic_name>
```

List packages:

```bash
ros2 pkg list
```

List parameters:

```bash
ros2 param list
```

---

# Key Takeaways

Before moving to Part 2, make sure you understand:

* ROS is middleware, not an operating system.
* Nodes are individual processes that perform specific tasks.
* Topics allow continuous communication between nodes.
* Services provide request-response communication.
* Actions handle long-running tasks.
* Parameters configure node behavior.
* Launch files start multiple nodes together.
* Packages organize ROS code.
* Workspaces are used to develop ROS projects.

These concepts form the foundation upon which all ROS systems are built.
