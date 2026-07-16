# GitHub Day Log – June 28, 2026
## MoveIt Commander API Study (Part 2)

Today I continued working through the Python `MoveGroupCommander` API documentation to better understand how MoveIt performs motion planning and manipulation.

### Topics Studied

- Motion planning architecture
- Pose goals versus joint goals
- Path constraints
- Trajectory constraints
- Planning frames
- Planning pipelines
- Planner IDs
- Current robot state
- Current end-effector pose
- Joint values
- Named targets
- Cartesian path generation
- Execution of planned trajectories
- Goal tolerances
- Variable counts
- Motion execution using `go()`

### Important Concepts Learned

### `go()`

The `go()` function is the primary command that executes a planned robot motion after all planning and collision checking have completed.

Typical workflow:

1. Define target
2. Plan motion
3. Check collisions
4. Verify constraints
5. Execute motion

---

### Cartesian Paths

Learned how MoveIt can generate straight-line end-effector motion through multiple waypoints while respecting collision checking and optional path constraints.

---

### Planning Information

Studied how MoveIt exposes information such as:

- Current planner
- Planning pipeline
- Planning frame
- End-effector frame
- Robot state
- Goal tolerances

These functions are valuable for debugging and understanding the planning environment.

---

### Documentation Progress

Continued reviewing the `MoveGroupCommander` API documentation in detail to understand how each function contributes to motion planning and manipulation.

Rather than simply memorizing functions, I am building an understanding of:

- when each function is used
- why it exists
- where it fits into the overall manipulation pipeline

This study will help when implementing future ROS2 MoveIt manipulation projects.

Next study session:
Continue through the remaining MoveGroupCommander API functions before moving into additional MoveIt tutorials.
Day_Log 2026-06-29:
I Completed BeanyBot Robotics Podcast Episode 12.
Interviewed a Ghanaian cocoa cooperative leader.
He confirmed labor shortage remains the primary operational challenge.
I learned pods should ideally be collected within four days after harvest to help preserve bean quality.
I Identified pod breaking as the most labor-intensive post-harvest activity.
I Added pollination to the list of future research topics after it was raised for the first time during the podcast.
I am continuing my requirements gathering for future agricultural robotics research.
Today's public GitHub entry should highlight **your learning progress** while avoiding any BeanyBot implementation details or patent-sensitive information.

You can paste the following into your **Day_Log.md**:

---

# Day_Log.md – June 29, 2026

## Robotics Engineering Progress

Today's focus was strengthening my ROS 2 Python and MoveIt 2 fundamentals.

### Completed

* Successfully created and rebuilt another ROS 2 Python package from scratch.
* Practiced creating and debugging ROS 2 Python nodes.
* Learned to interpret Python traceback messages instead of randomly changing code.
* Corrected multiple syntax, indentation, and constructor errors.
* Successfully executed a ROS 2 timer node printing the local system time every seven seconds.
* Continued reviewing ROS 2 package structure, including:

  * package.xml
  * setup.py
  * setup.cfg
  * resource/
  * test/
* Began transitioning from introductory ROS 2 programming toward MoveIt 2 manipulation.

### Mentor Meeting

Met with my faculty mentor today.

Key guidance:

* Continue focusing on MoveIt 2 manipulation rather than Nav2.
* Begin studying ROS 2 Launch files together with RViz.
* Maintain detailed GitHub engineering documentation throughout the learning process.

I was also honored to be invited to serve as a paid Teaching Assistant for Sonoma State University's first Computer Science robotics course beginning this fall semester.

I am grateful for the opportunity and will continue strengthening my ROS 2, RViz, and MoveIt 2 skills.

### Next Objectives

* Learn ROS 2 Launch files.
* Continue MoveIt 2 manipulation tutorials.
* Practice RViz visualization.
* Continue building a strong public robotics engineering portfolio.

Day_Log 2026-07-01:
group.set_joint_value_target(...)
What does it do?

It tells MoveIt: "I want the robot's joints to end up in this configuration." Notice something important: It does not move the robot. It only tells MoveIt where you want the robot to go.

Think of it as entering a destination into a GPS. Entering the destination does not make your car move. Later you would call:group.plan() or group.go() to actually move.
Example 1: Suppose your robot arm has six joints. Current position: 
Joint1 0°
Joint2 0°
Joint3 0°
Joint4 0°
Joint5 0°
Joint6 0°
You tell move it: group.set_joint_value_target(
    [0.2, -0.5, 0.8, 0.0, 1.0, -0.3]
)
Now MoveIt stores that as the goal. Nothing moves yet. Later, plan = group.plan() MoveIt computes a safe trajectory. Later, group.go() # Now the robot actually moves.
Why is it called "joint value target"? Because every joint has a value. For example: 

Shoulder = 20°

Elbow = 45°

Wrist = -30°
Those are the joint values. The collection of those values becomes the target. The documentation mentions several ways to call it. (arg1
arg2
arg3)
Instead of one fixed input. Python supports function overloading by input type. MoveIt allows several different kinds of targets. A list (most common) like, group.set_joint_value_target(
    [0.5, 1.0, -0.8, 0.3, 0.0, 1.2]
)
It has a A dictionary. Instead of relying on order: {
"shoulder_pan_joint":0.5,
"elbow_joint":1.2
}
you specify the names directly, That makes the code easier to read. A single joint. You can move only one joint. Like: group.set_joint_value_target(
"wrist_joint",
0.5
)
but, Only that joint changes. Everything else stays the same. A Pose. This is the really interesting one. Instead of saying 

Joint1 = ?

Joint2 = ?

Joint3 = ?
you say, Put the gripper HERE. For example:
X = 0.40

Y = 0.15

Z = 0.60
MoveIt then solves the mathematics. This mathematics is called Inverse Kinematics (IK). Suppose you tell the robot "Put your fingertip here." The robot now asks "What shoulder angle, elbow angle, and wrist angle will place my fingertip there?" That is exactly what IK solves. The documentation even says. "Instead, one IK solution will be computed first." That sentence is very important, becaue, MoveIt computes the joint angles for you. How this fits into your robotics work? As I think of my TIAGo manipulator. Instead of manually calculating six or seven joint angles, I would simply tell MoveIt:
Place the gripper 15 cm above the object. MoveIt then: Computes the joint angles (Inverse Kinematics). Plans a collision-free trajectory. Executes the motion. 
The engineering workflowL This is the normal sequence I will see in many MoveIt programs: 
group.set_joint_value_target(target)

plan = group.plan()

group.go(wait=True)

group.stop()
Each step has a distinct purpose: set_joint_value_target() — Defines the desired joint configuration. plan() — Computes a safe trajectory from the current position to that target. go() — Executes the planned motion. stop() — Ensures any residual motion commands are halted after execution. As your mentor has emphasized, A robotics engineer should be able to explain why each line exists. 
For set_joint_value_target(), the "why" is: It defines the goal but does not move the robot. It can accept several kinds of inputs (lists, dictionaries, single joints, or poses), making it flexible for different applications. When given a pose, MoveIt uses Inverse Kinematics to convert that desired end-effector position into the joint angles needed to reach it. It is the first step in almost every MoveIt motion-planning workflow. Understanding that distinction, setting a target versus executing motion, is a hallmark of understanding MoveIt rather than just using it.

## July 2, 2026 – ROS 2 TF2 Fundamentals

Today I completed the ROS 2 Humble **Introduction to TF2** tutorial.

### Completed
- Launched the TF2 TurtleSim demonstration.
- Controlled the leader turtle while observing the follower turtle use TF2 transforms.
- Generated and analyzed the TF frame graph using `view_frames`.
- Used `tf2_echo` to observe live translation and rotation data between coordinate frames.
- Visualized the TF frame tree in RViz2.

### Key Concepts Learned
- Coordinate frames define the position and orientation of objects in ROS 2.
- TF2 continuously publishes transforms between coordinate frames.
- `view_frames` generates a visualization of the TF tree.
- `tf2_echo` displays live transform data between frames.
- RViz2 provides a real-time visualization of TF relationships.

### Reflection
Today's work strengthened my understanding of one of the core building blocks of ROS 2. A solid understanding of TF2 will support future work in robot perception, visualization, motion planning, and manipulation.

**Status:** TF2 tutorial completed successfully.
## MoveIt2 commander API: 

## July 5, 2026 – MoveIt Motion Planning Parameters for Precision Cocoa Harvesting:

Today I continued studying the MoveIt Python API (MoveGroupCommander) to better understand how autonomous manipulators can perform safe, precise agricultural tasks. Rather than simply learning new functions, I focused on how each capability could eventually contribute to the BeanyBot cocoa harvesting workflow.

Topics Studied

Motion Speed Control

Learned how to use:

set_max_velocity_scaling_factor()

to reduce the robot arm's joint velocity.

Rather than always moving at maximum speed, the robot can intentionally slow its approach while nearing a cocoa pod. This should improve positioning accuracy, reduce overshoot, and minimize unintended impacts with branches or neighboring pods.

Motion Acceleration Control

Learned how to use:

set_max_acceleration_scaling_factor()

to limit how aggressively the arm accelerates and decelerates.

Lower acceleration values produce smoother starts and stops, reducing mechanical shock and allowing the manipulator to move more carefully around delicate crops.

Named Joint Configurations

Studied:

set_named_target()

which allows predefined robot poses to be stored and recalled by name.

Potential BeanyBot poses include:

Home
Walk
Detect
Cut Ready
Cut
Drop
Rest
Charge

Using named configurations should simplify future workflow code while improving readability and maintainability.

Motion Planning Attempts

Studied:

set_num_planning_attempts()

MoveIt can attempt multiple motion plans before selecting the best trajectory.

This may become valuable when navigating the arm around cocoa branches, pods, and the robot's own links, increasing the probability of finding smooth collision-free motions.

End Effector Orientation

Studied:

set_orientation_target()

This function controls the orientation of the end effector independently from its position.

For a future harvesting system using a mounted pruning tool, maintaining the correct blade orientation may be critical for approaching the cocoa pod stem at an effective cutting angle.

Path Constraints

Studied:

set_path_constraints()

This allows motion rules to remain active throughout an entire trajectory.

Potential agricultural applications include:

Maintaining a vertical cutting tool
Limiting wrist rotation
Keeping the arm within safe operating angles
Preventing unnecessary twisting while moving through tree canopies
BeanyBot Design Reflection

Today's MoveIt study reinforced that successful agricultural robotics depends on precision rather than speed.

A future cocoa harvesting robot must:

Move smoothly around trees.
Maintain proper tool orientation.
Reduce sudden arm acceleration.
Select collision-free trajectories.
Return to predefined safe poses after each harvesting cycle.

These concepts will likely become important building blocks as I continue progressing from MoveIt tutorials toward Isaac Sim simulations and eventually integrating a harvesting end-effector with the BeanyBot platform.

Hours Invested

Approximately 3–4 hours studying MoveIt motion planning concepts, API documentation, and evaluating their potential application to autonomous cocoa harvesting.
July 6, 2026 – Completing My Study of MoveIt's MoveGroupCommander API

Today I completed my review of the public methods available in MoveIt's MoveGroupCommander Python API. This study helped me better understand how MoveIt plans, executes, and safely manages robotic manipulator motion.

Topics Studied

During today's study I learned about:

shift_pose_target() – Making small incremental adjustments to an end-effector's position or orientation without defining an entirely new target pose.
stop() – Safely interrupting robot motion during execution.
Start states and current robot states for motion planning.
Workspace boundaries and trajectory constraints.
Motion planner selection and planning pipelines.
Position, orientation, and Roll-Pitch-Yaw target definitions.
Velocity and acceleration scaling factors for smooth robot motion.
Key Takeaways

One of the biggest lessons from today's study is that successful robotic manipulation depends on much more than simply commanding a robot to "move."

Motion planning involves:

Choosing an appropriate planner.
Defining the robot's starting state.
Specifying precise position and orientation goals.
Applying motion and trajectory constraints.
Limiting velocity and acceleration for safe operation.
Making fine adjustments when necessary.
Providing mechanisms to safely stop robot motion.

These concepts form the foundation of autonomous manipulation in many robotics applications, including manufacturing, warehouse automation, medical robotics, and agricultural robotics.

Looking Ahead

My next objective is to continue applying these MoveIt concepts while progressing toward Isaac Sim simulation and future autonomous manipulation workflows. Understanding these core APIs provides an important foundation for future robotics development.
## Day_Log July 7 2026: 

This is one of the smartest adapters in MoveIt. Instead of immediately giving up when the robot starts in a collision, it first asks: 

"Can I make a tiny adjustment to the robot's joints so it's no longer colliding?" 

If the answer is yes, planning continues automatically. 

Why is this needed? 

Suppose your robot starts in a slightly bad position. 

For example: 

Robot Arm 

Branch 
 

────────────── 

Arm 
 

Perhaps the wrist is barely touching a branch. 

Or maybe the robot is barely touching itself. 

MoveIt sees: 

Starting State = In Collision 

Normally, planning cannot begin because the robot is already in an invalid configuration. 

Without FixStartStateCollision 

Imagine this sequence: 

Current Robot State 

↓ 

Collision Detected 

↓ 

Planning Failed 

The robot never even starts planning. 

With FixStartStateCollision 

Instead MoveIt tries something smarter. 

It says: 

"What if I move one or more joints just a tiny amount?" 

For example: 

Original 

Branch ──────────── 

Arm 

Tiny adjustment: 

Branch ──────────── 

Arm 

Now the collision disappears. 

Planning can continue normally. 

What does "perturbing the joints" mean? 

The documentation says: 

perturbing the joint values by a small amount. 

"Perturb" simply means: 

Make a very small random change. 

Suppose a shoulder joint is at: 

45° 

MoveIt might try: 

44.8° 

45.2° 

44.6° 

45.1° 

Each attempt is only slightly different Explain>  

This section explains a very clever feature of MoveIt called the Self-Filter. Without it, the robot's own camera would constantly mistake its own arm for an obstacle. 

Real-World Example 

  

Imagine you're holding your hand in front of your face. 

  

You still know your hand belongs to you, not to the environment. 

  

The self-filter gives the robot a similar capability. It allows the robot to ignore its own body when interpreting depth data, so it can focus on external objects that matter for planning. 
## Day_Log Wedensday July 8 2026:
July 8, 2026

Today's focus was completing the official ROS 2 Humble TF2 Static Broadcaster (Python) tutorial.

Accomplishments
Successfully created the learning_tf2_py ROS 2 package.
Corrected the package location by moving it into the workspace src/ directory.
Updated package.xml with the required dependencies, including:
rclpy
tf2_ros
tf2_ros_py
geometry_msgs
turtlesim
python3-numpy
Updated setup.py with the required console_scripts entry:
static_turtle_tf2_broadcaster
Created the static_turtle_tf2_broadcaster.py node from the official ROS 2 Humble tutorial.
Successfully built the package using:
colcon build --packages-select learning_tf2_py
Successfully sourced the workspace.
Ran the static TF2 broadcaster.
Verified the transform using:
ros2 run tf2_ros tf2_echo world mystatic
Results
Static transform published successfully.
Translation, quaternion, rotation matrix, and Euler angles displayed correctly.
No build or runtime errors after package corrections.
Lessons Learned
ROS 2 packages must reside inside the workspace src/ directory.
setup.py and package.xml must be configured correctly before building.
TF2 static broadcasters publish a transform once that remains constant throughout execution.
tf2_echo is an effective tool for verifying transforms.
Git commit was postponed because the current workspace is not initialized as a Git repository.
# Day Log – ROS 2 TF2 Listener & Launch File Investigation

Today I continued working through the ROS 2 Humble TF2 tutorials.

## Completed
- Reviewed the TF2 Listener architecture.
- Created the `turtle_tf2_listener.py` node.
- Updated `setup.py` with the new console script entry.
- Learned how ROS 2 installs Python executables through `console_scripts`.
- Created and configured the `launch` directory.
- Added launch file installation to `setup.py`.
- Successfully rebuilt the package multiple times using `colcon build`.
- Verified package installation and workspace sourcing.

## Debugging Experience
During testing I encountered launch file and TF broadcaster issues that prevented the complete demo from running correctly. The listener node launched successfully, but the static TF broadcasters exited due to launch argument configuration problems. Rather than continuing to patch the tutorial, I decided to pause and begin a fresh lesson with a clean environment.

## Takeaways
Today's session reinforced how important package configuration, launch files, and executable registration are in ROS 2. Even though the final demo did not complete, I gained a much deeper understanding of the ROS 2 package structure and the debugging process.

Tomorrow I'll continue with a fresh tutorial and keep moving toward the larger goal of autonomous agricultural robotics for BeanyBot.
## Day_Log Wednesday July 15 2026:

Isaac Sim 5.1 – Franka Pick-and-Place Tutorial (Part 2)

Today I completed a detailed study of the complete execution flow of NVIDIA Isaac Sim's Franka Pick-and-Place example. Rather than simply running the example, I analyzed every major section of the source code to understand how the simulator, physics engine, and controller work together to execute an autonomous manipulation task.

Topics Studied
Physics simulation initialization using SimulationManager.setup_simulation()
Physics timestep (dt = 1/60) and why 60 Hz is commonly used for robotics simulations
Retrieving the active physics scene with SimulationManager.get_physics_scenes()
CPU versus GPU dynamics and the purpose of set_enabled_gpu_dynamics(False)
Starting the simulation with app_utils.play()
Initializing robot articulations before controller startup using app_utils.update_app(steps=20)
Resetting the Franka Pick-and-Place controller
Understanding the controller state machine
Computing max_test_steps using controller.events_dt
Studying the complete simulation loop using:
simulation_app.update()
controller.forward()
controller.is_done()
app_utils.pause()
Understanding automated testing safeguards using RuntimeError
Proper simulation shutdown using:
app_utils.stop()
simulation_app.close()
Concepts Learned

Today's lesson reinforced several fundamental robotics concepts:

Physics engines advance one frame at a time.
Robot controllers operate as state machines rather than issuing one large motion command.
Every simulation frame updates robot joints, collisions, sensors, and controller outputs.
Initialization order is important to ensure articulations and physics tensors are valid before control begins.
Good robotics software includes timeout protection and graceful shutdown procedures.

## FoxGlove Tutorial One: July 15, 2026: 
Think of this as the **robotics equivalent of a pilot's cockpit**. Instead of flying an airplane, you're monitoring everything your robot is seeing, thinking, and doing—all at the same time.

Let's go through the tutorial in plain English.

---

# What is Foxglove?

Imagine you're driving your car.

Normally I only look out the windshield.

Foxglove is like having all of these at once:

* windshield camera
* backup camera
* dashboard gauges
* GPS
* engine diagnostics
* speedometer
* fuel gauge
* tire pressure monitor

All synchronized together.

That is Foxglove for robots.

Instead of only seeing the robot move in Gazebo or Isaac Sim...

Foxglove lets you see **everything happening inside the robot's brain.**

---

# Step 1

## Open an MCAP File

The instructor loads an **MCAP** file.

Think of an MCAP file as

> A flight recorder.

Exactly like the black box on an airplane.

It records everything that happened.

For example:

* camera images
* LiDAR scans
* GPS
* wheel speed
* robot position
* IMU
* joint angles
* battery
* motor commands

Later Foxglove plays it back.

It is literally replaying the robot's life.

---

# Step 2

## Topics

On the left side is something called **Topics**.

A topic is simply

> one stream of information.

Examples

Camera topic

```
camera/image
```

LiDAR topic

```
scan
```

Robot position

```
odom
```

Velocity

```
cmd_vel
```

Battery

```
battery_state
```

Think of topics like television channels.

Every channel broadcasts different information.

Foxglove lets you tune into whichever channel you want.

---

# Step 3

## Raw Message Panel

The instructor opens a **Raw Message Panel.**

This simply displays the numbers.

Instead of pretty graphics...

it literally shows

```
x = 1.54

y = 0.82

velocity = 0.33

orientation = ...
```

Think of it like opening the robot's diary.

Every measurement is written down.

---

# Step 4

## The Timeline

At the bottom is a time slider.

Imagine watching a football replay.

You can pause.

Go backward.

Go forward.

Everything updates to that exact moment.

Camera.

LiDAR.

Robot pose.

Acceleration.

Everything.

Very useful when something goes wrong.

---

# Step 5

## 3D Panel

This is the coolest part.

The instructor creates a **3D panel.**

Instead of looking at numbers...

Foxglove draws the robot in 3D.

In your screenshot...

You can see

* robot body
* wheels
* LiDAR points

all in one scene.

That is exactly what my first screenshot shows.

---

# Step 6

## URDF

Then he loads the robot's **URDF.**

URDF means

**Unified Robot Description Format**

This is basically

> the robot's blueprint.

It tells Foxglove

"My robot has..."

* four wheels
* one laser
* one camera
* this body
* these joints

Without a URDF...

Foxglove only knows there are numbers.

With a URDF...

Foxglove knows what the robot actually looks like.

---

# Step 7

## LiDAR Points

Then he turns on the LiDAR.

Those green dots are laser measurements.

Your first screenshot shows this perfectly.

Imagine standing in a dark room.

Instead of using your eyes...

you shine a laser in every direction.

Every time the beam hits something...

you place a green dot there.

Thousands of dots create a 3D picture.

That picture is called a

**Point Cloud.**

---

# What are the circles?

Notice the rings around the robot.

Those are LiDAR scans.

The laser spins hundreds of times every second.

Every rotation measures distance.

Foxglove draws every measurement.

That is how robots "see."

---

# Step 8

## Base Link

The instructor says

> "Referenced to the body of my robot."

This is extremely important.

Imagine standing in the middle of your living room.

Everything around you moves as you walk.

From your point of view...

YOU never move.

The world moves around you.

That is exactly what

```
base_link
```

means.

The robot is always the center.

Everything else moves around it.

---

# Step 9

## Image Panel

Now he opens the camera.

Your second screenshot shows this.

Upper left

Camera image

Lower left

LiDAR

Both are synchronized.

That means

when the camera sees a tree...

the LiDAR also sees the same tree.

Exactly the same instant.

This is incredibly useful.

---

# Step 10

## Plot Panel

The graph on the lower right is a Plot.

Instead of looking at pictures...

it graphs numbers over time.

In this example

```
Acceleration X
```

means

How quickly the robot speeds up and slows down.

Imagine driving over a pothole.

The graph suddenly spikes.

That spike tells you

> the robot experienced a bump.

---



# Step 11

## Synchronization

This is probably the biggest feature.

Everything happens together.

Move the timeline...

Everything changes.

Camera.

LiDAR.

Graphs.

Robot.

Messages.

All synchronized.

It is like replaying security cameras from multiple angles at once.

---

# Looking at Your Screenshots

## Screenshot 1

I am looking at

* 3D robot model
* LiDAR point cloud
* IMU sensor data
* robot orientation
* performance statistics

This is mostly for understanding

> Where is my robot?

> What is it sensing?

> Is everything updating correctly?

---

## Screenshot 2

This is a more complete dashboard.

Upper left

Camera

Lower left

3D LiDAR

Upper right

IMU measurements

Lower right

Acceleration graph

This is exactly how many robotics engineers debug real robots.

## What I should remember from Tutorial #1

By the end of this tutorial, I should understand these core ideas:

* **Foxglove is a robotics dashboard** for visualizing and debugging robots.
* An **MCAP file** is like a flight recorder that stores all of a robot's data.
* **Topics** are individual streams of information (camera, LiDAR, battery, odometry, etc.).
* The **3D panel** shows the robot and its environment.
* A **URDF** gives Foxglove the robot's physical model so it can display it correctly.
* **LiDAR point clouds** let you see what the robot is sensing in 3D.
* The **Image panel** displays synchronized camera video.
* The **Plot panel** graphs sensor values over time to reveal things like vibrations or sudden impacts.
* The **timeline** keeps every sensor synchronized, making it easy to replay and diagnose robot behavior.

For my ROS 2 and TIAGo work at Sonoma State, Foxglove will become one of my most valuable tools. As I start integrating navigation, MoveIt, cameras, you'll use Foxglove to verify that all of your sensors, robot state, and AI decisions are working together correctly.

## FoxGlove tutorial (3): Navigate the Foxglove app with confidence:

Here's the tutorial explained in plain English.

---

# What was the purpose of this tutorial?

The first tutorial taught you **what Foxglove can display**.

This tutorial teaches you

> **how to navigate Foxglove.**

It's like learning where the steering wheel, mirrors, dashboard, and controls are before driving a car.

---

# Dashboard

When Foxglove opens, you arrive at the **Dashboard**.

Think of it as the home screen.

From here you can:

* Open a recording
* Connect to a robot
* Upload data
* Open recent work
* Browse examples

It's similar to Microsoft Word opening to the "Recent Documents" page.

---

# Open Local File

If your robot already finished driving...

and you recorded everything...

you simply open the recording.

Example:

```
Today's TIAGo navigation test
```

or

```

```

Foxglove replays it.

---

# Open Connection

This is one of the most important buttons.

Instead of watching yesterday's recording...

you connect directly to a robot that is running **right now.**

Imagine watching TIAGo driving through the lab live.

Every sensor updates instantly.

This is called

**live telemetry.**

---

# Upload File

Suppose you're working with another robotics engineer.

Instead of emailing a huge recording...

you upload it to Foxglove Cloud.

Now everyone can view the exact same recording.

---

# Recent Recordings

Foxglove remembers what you recently opened.

Exactly like Word remembers recent documents.

No need to browse for them again.

---

# Example Datasets

Foxglove gives you sample robots.

This is great for learning.

Even if you don't own a robot...

you can practice.

---

# Documentation

Whenever you forget something...

this takes you to Foxglove's manuals.

You'll use these often.

Even experienced robotics engineers do.

---

# Devices

A device simply means

> A robot.

Suppose BeanyBot owns:

* TIAGo
* Optimus

Foxglove treats each as a separate device.

You can organize them by:

* name
* location
* customer
* farm
* serial number

---

# Recordings

Think of recordings as

> Robot movies.

Every time the robot works...

Foxglove saves a recording.

Later you can search them.

Example

```
July 3
```

```
July 8
```

```
July 15
```

Instead of hundreds of random files...

everything stays organized.

---

# Events

Events are bookmarks.

Suppose your robot got stuck.

Instead of remembering

"around 12 minutes..."

you create an event.

Example:

```
Obstacle detected
```

or

```
Failed grasp
```

or

```
Battery low
```

Now Foxglove jumps directly there.

Very useful.

---

# Timeline

Imagine your robot worked for six months.

The Timeline lets you zoom from

Year

↓

Month

↓

Day

↓

Hour

↓

Minute

↓

Second

until you find the exact recording you need.

---

# Layouts

A layout is simply

> A dashboard arrangement.

Example

Navigation Layout

* LiDAR
* Map
* Camera

Another layout

Manipulator Layout

* Arm joints
* Wrist camera
* Gripper

Another

Battery Layout

* Voltage
* Current
* Temperature

Instead of rebuilding your dashboard every day...

Foxglove remembers it.

---

# Panels

Panels are windows.

Examples:

Camera panel

Shows camera.

Plot panel

Shows graphs.

3D panel

Shows robot.

Raw Message panel

Shows numbers.

Map panel

Shows map.

Think of them as apps on your phone.

---

# Drag and Drop

This is one of Foxglove's best features.

You simply drag

```
Camera
```

into

```
Image Panel
```

Instantly it appears.

Drag

```
Laser Scan
```

into

```
3D Panel
```

Now you see LiDAR.

No programming required.

---

# Topics

Topics are still

information channels.

Every sensor publishes one.

Examples

```
Camera
```

```
LiDAR
```

```
GPS
```

```
Battery
```

```
Robot Arm
```

You drag topics into panels.

---

# Problems Tab

This is like your car's

Check Engine Light.

If something is wrong...

Foxglove tells you.

For example

Missing transform

Missing camera

Bad topic

Broken connection

Always check this first.

---

# Playback Controls

Exactly like YouTube.

Play

Pause

Fast forward

Rewind

Loop

Slow motion

Very useful when debugging.

---

# Playback Speed

Instead of watching a robot drive for ten minutes...

watch it at

2×

or

4×

speed.

Or slow it down.

Perfect for catching bugs.

---

# Playback Range

Suppose your robot fails

between

2:10

and

2:40.

Instead of replaying twenty minutes...

Foxglove only replays

those thirty seconds.

Huge time saver.

---

# Extensions

Extensions are add-ons.

Just like Chrome extensions.

People build tools that add new features.

---

# Settings

Here you change

Dark Mode

Profile

Appearance

Debugging

Extensions

General preferences.

---

# About

This simply tells you

what version of Foxglove you are using.

If something breaks...

support will ask for this.

---



Instead of searching through hours of recordings, you'll be able to jump directly to the important moments and analyze exactly what happened.

---

## What you should remember from Tutorial #2

The key lesson is that **Foxglove is more than a viewer—it's an organized workspace for robotics development**. It helps you:

* Organize robots ("Devices")
* Store and search recordings
* Mark important moments with Events
* Reuse custom dashboards (Layouts)
* Display data using Panels
* Inspect ROS 2 Topics
* Replay recordings at any speed
* Troubleshoot using the Problems tab

For your work with **ROS 2, TIAGo, Isaac Sim, learning these navigation tools will make debugging and analyzing robot behavior much faster as your projects become more complex.
## FoxGlove Studios creating layouts:
Excellent! Tutorial #3 is short, but it introduces one of the most useful features in Foxglove. As your BeanyBot robots become more advanced, **Layouts** will save you hours of work.

Let's explain it in plain English.

---

# What is a Layout?

Imagine you're sitting in the cockpit of an airplane.

One pilot wants to see:

* Altitude
* Speed
* Engine gauges

Another pilot wants:

* Weather radar
* Navigation map
* Fuel system

Instead of moving all the instruments around every day...

they save different **cockpit layouts**.

That is exactly what a **Foxglove Layout** is.

It is simply

> **A saved dashboard.**

---

# What is inside a Layout?

A layout remembers:

* which panels are open
* where the panels are
* what topics each panel displays
* panel sizes
* panel settings

Instead of rebuilding your dashboard every day...

Foxglove remembers everything.

---

# Example

Suppose today you're debugging TIAGo navigation.

Your dashboard might look like:

```
Camera
Map
LiDAR
Robot Model
Velocity
```

Tomorrow you're debugging MoveIt.

Now you want

```
Robot Arm
Joint States
Gripper
Planning Scene
Trajectory
```

Instead of rearranging everything...

you simply load another layout.

---

# Why different teams need different layouts

The instructor says:

> Control engineers and perception engineers need different layouts.

Let's explain.

Imagine BeanyBot has three engineers.

---

## Navigation Engineer

This engineer only cares about driving.

Their layout might include:

* Map
* LiDAR
* GPS
* Robot Position
* Velocity

Nothing else.

---

## AI / Vision Engineer

This engineer doesn't care about wheels.

They want:

* RGB Camera
* Depth Camera
* Cocoa Pod Detector
* Confidence Scores
* Bounding Boxes

---

## Robot Arm Engineer

This engineer only wants:

* Joint Angles
* End Effector
* MoveIt Planning
* Wrist Camera
* Gripper Status

Completely different dashboard.

---

# The Blue Dot

The instructor points out the **blue dot**.

What does it mean?

Imagine Microsoft Word.

You type one letter.

Suddenly:

```
Document*
```

appears.

The * means

"You have unsaved changes."

Foxglove's blue dot means exactly the same thing.

You changed something.

It hasn't been saved yet.

---

# Reverting Changes

Suppose you accidentally move panels around.

Instead of rebuilding everything...

Foxglove lets you restore the last saved layout.

Very handy.

---

# Sharing Layouts

This is one of Foxglove's most powerful features.

Suppose my entire BeanyBot engineering team is debugging the same robot.

I can build the perfect dashboard.

Instead of telling everyone

> Put this panel here...
> Move this over there...
> Resize this...

I simply save it.

Everyone opens the same layout.

Instantly everyone's dashboard looks identical.

---

# Organization Layouts

Foxglove lets me save layouts for

Everyone

or

Only myself.

Think of it like Google Drive.

Personal folder

Only you can see it.

Shared folder

Everyone on your team can use it.


# Creating a New Layout

The instructor demonstrates:

```
Layouts

↓

Manage Layouts

↓

Add

↓

New Layout

↓

Give it a name
```

That's all there is to it.

---

# How I Will Use Layouts?

Once I begin integrating ROS 2, MoveIt, Nav2, Isaac Sim, and real robots, I'll probably create layouts such as:

### TIAGo Navigation

* Map
* LiDAR
* Camera
* Velocity
* Robot Model

---

### TIAGo Manipulation

* MoveIt Planning Scene
* Joint States
* Gripper
* Wrist Camera
* End Effector

---


### Diagnostics

* CPU Usage
* Memory
* Network
* ROS Logs
* Errors

Instead of constantly rearranging panels, you can switch between these layouts with a click.

---

## What I should remember from Tutorial #3

A **Layout** is simply a **saved Foxglove dashboard**. It remembers the arrangement of my panels, the topics being displayed, and your visualization settings. Different robotics tasks—and different engineers—need different layouts, so Foxglove lets me save and switch between them quickly. As my BeanyBot project grows from simulation to real robots, using well-organized layouts will make debugging, testing, and collaboration much more efficient.

## Issac Sim: Understanding the Pick-and-Place State Machine: 


Yesterday 2:03 PM
Explain> Using FrankaPickPlace for Complete Pick-and-Place Code: Line 1: import argparse

Let's go through it one line at a time.

Line 1
import argparse
What does this do?
This line imports Python's built-in argparse module.

argparse allows a program to accept command-line arguments when you start it from the terminal.

Think of it as giving your Python program adjustable settings without changing the code itself.

For example, instead of writing:

DEVICE = "cpu"
you could start the program like this:

python franka_pick_place.py --device gpu
or

python franka_pick_place.py --device cpu
The argparse module reads --device gpu and makes it available to the program.

Why is this useful in Isaac Sim?
Many Isaac Sim examples are designed to run in different ways.

Instead of editing the source code every time, NVIDIA lets you choose options when launching the script.

Examples include:

CPU vs GPU physics

Headless mode (no graphics)

Which robot to load

Which USD world to open

Simulation speed

Debug mode

So the same script can be reused for many situations.

A simple example
import argparse

parser = argparse.ArgumentParser()

parser.add_argument("--name", default="Selwyn")

args = parser.parse_args()

print("Hello", args.name)
If you run:

python hello.py
Output:

Hello Selwyn
If you run:

python hello.py --name Franka
Output:

Hello Franka
Nothing in the code changed—you simply passed a different argument.

In the Franka Pick-and-Place tutorial
Later in the tutorial, you'll likely see something similar to:

parser = argparse.ArgumentParser()
parser.add_argument("--device", default="cpu")
args = parser.parse_args()

DEVICE = args.device
Now you can choose the physics device from the terminal:

python franka_pick_place.py --device gpu
or

python franka_pick_place.py --device cpu
The program automatically uses the device you selected.

Why NVIDIA uses argparse
Instead of editing Python files every time you want a different option, argparse lets you:

Change settings from the terminal.

Keep one script for many configurations.

Make tutorials more flexible.

Test different hardware easily.

This is a standard Python tool used in robotics, machine learning, and scientific computing.

Key takeaway
This line:

import argparse
does not control the robot or simulation directly. It simply loads a Python module that lets the program accept options (like --device gpu) from the command line, making the script flexible without requiring you to edit the code.


Today 11:36 AM
This is one of the most important setup lines in the tutorial because it loads additional functionality into Isaac Sim.

app_utils.enable_extension("isaacsim.robot.experimental.manipulators.examples")
What does this line do?
This line tells Isaac Sim:

"Load and activate the Manipulators Examples extension."

Once this extension is enabled, your program can use the example robot classes included with it, such as the Franka Pick-and-Place example.

Without enabling this extension, those example classes would not be available.

Breaking it down
app_utils
This is the application utility module you imported earlier:

import isaacsim.core.experimental.utils.app as app_utils
It provides helper functions for managing the Isaac Sim application.

.enable_extension()
This function loads and activates an Isaac Sim extension.

Think of an extension as a plug-in that adds new capabilities to Isaac Sim.

"isaacsim.robot.experimental.manipulators.examples"
This is the name (identifier) of the extension to load.

Let's break it apart:

isaacsim
The main Isaac Sim package.

robot
This extension is related to robots.

experimental
It belongs to the experimental API, meaning NVIDIA may change it in future releases.

manipulators
These are robotic manipulators—robot arms that interact with objects.

Examples include:

Franka Emika Panda

UR5

UR10

Kinova arms

A manipulator is designed to reach, grasp, move, and place objects.

examples
This extension contains ready-to-use example code demonstrating manipulator tasks.

Why is this necessary?
Later in the tutorial you'll see something like:

from isaacsim.robot.experimental.manipulators.examples.franka import FrankaPickPlace
That class is provided by the extension you just enabled.

If you didn't enable it first, Python wouldn't be able to use that example because the extension wouldn't be loaded into Isaac Sim.

What does the extension contain?
Among other things, it includes helper classes such as:

FrankaPickPlace

Example controllers

Preconfigured robot scenes

State machines for pick-and-place demonstrations

Instead of writing hundreds of lines of setup code yourself, NVIDIA provides these reusable examples.

An analogy
Imagine Isaac Sim is a smartphone.

By default, the phone doesn't include every app.

Calling:

app_utils.enable_extension(...)
is like installing and opening a new app.

Once installed, you can use everything inside it.

In this case, the "app" is the Manipulators Examples extension.

Where it fits in the program
A typical sequence looks like this:

from isaacsim import SimulationApp

simulation_app = SimulationApp({"headless": False})

import isaacsim.core.experimental.utils.app as app_utils

app_utils.enable_extension(
    "isaacsim.robot.experimental.manipulators.examples"
)

from isaacsim.robot.experimental.manipulators.examples.franka import FrankaPickPlace
Start Isaac Sim.

Import the application utilities.

Enable the extension.

Import classes provided by that extension.

Each step depends on the previous one.

Key takeaway
app_utils.enable_extension("isaacsim.robot.experimental.manipulators.examples")
loads and activates the Manipulators Examples extension in Isaac Sim. This extension provides ready-made examples, including the FrankaPickPlace class and related robot-arm demonstrations. Enabling the extension makes these example classes available so your program can import and use them.


Explain> from isaacsim.core.experimental.objects import DomeLight, GroundPlane

This line imports two objects that help create a basic simulation environment.

from isaacsim.core.experimental.objects import DomeLight, GroundPlane
What does this line do?
It imports two classes:

GroundPlane – Creates the floor that robots stand on.

DomeLight – Creates lighting so the scene is illuminated.

Without these, your robot would either:

Have no floor to stand on, or

Be sitting in a dark scene.

Breaking it down
from
This tells Python:

"Go to this package and import only the classes I need."

isaacsim
The main Isaac Sim package.

core
Contains the core simulation features, including:

Worlds

Robots

Objects

Physics

Sensors

experimental
These are newer APIs that NVIDIA is still developing.

objects
This module contains common objects that can be added to a simulation scene.

Examples include:

Ground planes

Lights

Cubes

Spheres

Tables

Other physical objects

GroundPlane
The GroundPlane class creates a flat floor.

Later in the code, you'll often see:

GroundPlane("/World/ground_plane")
This adds a large horizontal plane to the simulation.

The ground plane:

Gives robots something to stand on.

Allows objects to rest on a surface.

Participates in physics and collision detection.

Prevents objects from falling forever.

Without a GroundPlane
If you spawn a Franka robot or a cube without a ground plane, gravity will pull them downward, and they will continue falling because there is nothing to stop them.

DomeLight
The DomeLight class creates a large light source surrounding the entire scene.

Later you may see:

dome_light = DomeLight("/World/DomeLight")
dome_light.set_intensities(1000)
This provides even illumination from all directions.

A dome light:

Brightens the entire scene.

Makes robots and objects visible.

Helps cameras produce clearer images.

Reduces harsh shadows compared to a single point light.

Why are both needed?
Imagine building a movie set.

First, you install the floor.

Then, you turn on the studio lights.

Only then can the actors perform.

In Isaac Sim:

GroundPlane
        ↓
Robot stands safely

DomeLight
        ↓
Robot becomes visible

Robot + Floor + Light
        ↓
Ready for simulation
Where this fits in the program
A typical setup looks like this:

from isaacsim.core.experimental.objects import DomeLight, GroundPlane

GroundPlane("/World/ground_plane")

dome_light = DomeLight("/World/DomeLight")
dome_light.set_intensities(1000)
After these lines:

A floor exists.

Lighting is enabled.

The Franka robot can be spawned into a properly lit environment.

Real-world analogy
Think of preparing a robotics laboratory.

Before testing a robot, you would:

Build a solid floor. (GroundPlane)

Turn on the overhead lights. (DomeLight)

Bring in the robot. (Franka)

Begin the experiment.

Isaac Sim follows the same logical sequence.

Key takeaway
from isaacsim.core.experimental.objects import DomeLight, GroundPlane
imports two scene object classes:

GroundPlane creates a physical floor that supports robots and other objects during simulation.

DomeLight creates ambient lighting that illuminates the entire scene, making robots, objects, and camera views clearly visible.

Together, they provide the basic environment needed before running a robotics simulation.

## Part 2: Using FrankaPickPlace for Complete Pick-and-Place: 










