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
ChatGPT Plus





Friday 12:48 AM
Part 3 FrankaPickPlace Code Explanation:  explain> dome_light.set_intensities(1000)

Absolutely! This line is very simple, but it's important because it controls how well you can see your simulation.

dome_light.set_intensities(1000)
What is a DomeLight?
Earlier in the code, you created a dome light:

dome_light = DomeLight("/World/DomeLight")
A DomeLight is like the sky in the real world. Instead of shining from one direction like the Sun, it illuminates the entire scene evenly from all directions.

Think of it like:

🌤️ An overcast sky

💡 A giant photography light tent

🏠 A room with bright ambient lighting

Without a light source, your robot, cube, and ground would appear very dark or even completely black.

What does set_intensities() do?
dome_light.set_intensities(1000)
This changes how bright the dome light is.

Higher number = brighter scene

Lower number = darker scene

In this example:

1000
means the scene is illuminated with an intensity of 1000 units.

Why do we need it?
The Franka robot is about to perform a pick-and-place task.

To see:

the robot arm

the cube

the ground plane

the robot's movements

the simulator needs lighting.

Without this line:

the simulation may be extremely dark

shadows may hide objects

debugging becomes much harder

Real-world analogy
Imagine walking into a warehouse at night.

If the lights are off:

❌ You can't see the robot.

Turn on the overhead lights:

💡 Now you can clearly watch the robot pick up the object.

That's exactly what this command does for Isaac Sim.

What if you used different values?
dome_light.set_intensities(100)
Very dim.

dome_light.set_intensities(1000)
Nice bright lighting (common choice).

dome_light.set_intensities(5000)
Very bright.

dome_light.set_intensities(0)
No light at all.

The simulation would be almost completely dark.

Does this affect the robot?
No.

It only affects the visual appearance of the simulation.

It does not change:

physics

robot motion

collisions

grasping

pick-and-place behavior

The Franka robot performs exactly the same actions whether the light intensity is 100 or 5000—the only difference is how well you can see the scene.

Step-by-step
dome_light = DomeLight("/World/DomeLight")
➡️ Create a dome light.

↓

dome_light.set_intensities(1000)
➡️ Set its brightness to 1000.

↓

The simulator now has good lighting.

↓

You can clearly watch the Franka robot pick up and place the cube.

In plain English
dome_light.set_intensities(1000) tells Isaac Sim to make the virtual lighting bright enough to clearly illuminate the entire scene, making it easy to observe the Franka robot, the cube, and the environment during the pick-and-place simulation.
ChatGPT Plus





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


I have to go attend my Zoom Next Gen Orientation soon. So, what is my LinkedIn post for today's lessons on Issac Sim?

Have a great time at your ARI NextGen Orientation! Here's a concise LinkedIn post that matches the style you've preferred—professional, focused on today's learning, and centered on your robotics progress.

Today I continued working through NVIDIA Isaac Sim and gained a deeper understanding of how a robotics simulation is built before a robot ever begins to move.

Today's lesson focused on the fundamentals behind the Franka Pick-and-Place example, including command-line argument parsing (argparse), launching the simulation with SimulationApp, enabling Isaac Sim extensions, and creating a basic simulation environment with a GroundPlane and DomeLight. Breaking the code down line by line has helped me better understand not just what the tutorial is doing, but why each component is necessary.

Every lesson strengthens my foundation in robotics software engineering as I continue developing skills in ROS 2, Isaac Sim, and autonomous manipulation.

My public GitHub portfolio continues to grow as I document my learning journey and projects.

#IsaacSim #Robotics #Python #ROS2 #AutonomousRobots #Engineering #Learning #SonomaStateUniversity

When you finish your orientation, we'll pick up right where we left off and continue working through the Franka Pick-and-Place tutorial line by line.

This section sets up the **simulation loop** for the Franka robot. It doesn't actually move the robot yet—it prepares variables that control **how long the simulation should run**.

```python
# Main loop: run one pick-place step each physics frame until done
step_count = 0
max_test_steps = sum(controller.events_dt) + 60
```

Let's break it down.

---

### Comment

```python
# Main loop: run one pick-place step each physics frame until done
```

This is telling you what the upcoming loop will do.

In Isaac Sim:

* The physics engine updates the world **60 times every second** (because earlier you set `dt = 1.0 / 60.0`).
* Each update is called a **physics frame**.
* During every frame, the `FrankaPickPlace` controller advances the robot one small step toward completing the pick-and-place task.

Instead of telling the robot:

> "Pick up the cube instantly."

it tells it:

> "Move a tiny bit."

60 times every second until the task is complete.

That produces smooth, realistic robot motion.

---

## Line 1

```python
step_count = 0
```

This creates a counter.

Initially:

```
step_count = 0
```

Every physics frame, the program will increase it.

For example:

Frame 1

```
step_count = 1
```

Frame 2

```
step_count = 2
```

Frame 3

```
step_count = 3
```

...

Eventually

```
step_count = 560
```

or whatever number of frames the task requires.

This counter is commonly used to:

* avoid infinite loops
* measure elapsed simulation time
* stop after a maximum number of frames

---

## Line 2

```python
max_test_steps = sum(controller.events_dt) + 60
```

This line determines the **maximum number of simulation steps** the controller is allowed to run.

Let's go inside it.

---

### controller.events_dt

Earlier you created

```python
controller = FrankaPickPlace()
```

Inside that class is a list called

```python
controller.events_dt
```

It contains the duration of every stage of the pick-and-place operation.

Conceptually it looks something like

```python
[
30,
50,
20,
40,
70,
...
]
```

These numbers represent how many physics frames each event lasts.

For example

| Event               | Frames |
| ------------------- | -----: |
| Move above cube     |     40 |
| Lower arm           |     20 |
| Close gripper       |     15 |
| Lift cube           |     35 |
| Move to destination |     60 |
| Lower cube          |     20 |
| Open gripper        |     15 |

---

### sum()

Python's

```python
sum(...)
```

adds everything together.

Example

```python
events_dt = [40,20,15,35]
```

Then

```python
sum(events_dt)
```

returns

```python
110
```

meaning

> the controller expects about **110 simulation frames** to finish.

---

### Why add 60?

```python
+ 60
```

adds an extra second.

Remember:

```
60 frames
=
1 second
```

because

```
dt = 1/60
```

This gives the controller a safety margin.

Without it:

Suppose the controller expects

```
520
```

frames

but due to tiny timing differences actually finishes at

```
523
```

The simulation would stop too early.

Instead

```
520 + 60
=
580
```

Now the controller has an extra second to complete the task cleanly.

---

## Why not use an infinite loop?

Instead of

```python
while True:
```

they use a maximum number of steps because:

* prevents bugs from running forever
* protects your CPU
* protects Isaac Sim from hanging
* automatically ends tests

This is standard practice in robotics.

Absolutely, Selwyn. Based on today's Isaac Sim lessons, here's a professional **GitHub Day Log** followed by a **LinkedIn post**.

---

# GitHub Day Log – July 20, 2026

## Day XX – Isaac Sim Core API Tutorial: Franka Pick-and-Place State Machine

### Objectives

* Continue learning NVIDIA Isaac Sim Core API tutorials.
* Understand the architecture of the `FrankaPickPlace` controller.
* Learn how a robotics state machine executes a complete pick-and-place task.
* Compare Isaac Sim manipulation with MoveIt 2 motion planning.

---

## Topics Studied

### Main Simulation Loop

Learned how Isaac Sim continuously executes physics updates using:

```python
while simulation_app.is_running():
```

This loop keeps the simulation alive until the application closes.

---

### Updating the Simulation

Studied:

```python
simulation_app.update()
```

This advances the simulator one physics frame by:

* Updating physics
* Rendering graphics
* Processing sensors
* Refreshing the UI

---

### Frame Counter

Learned the purpose of:

```python
step_count += 1
```

This counts simulation frames and is used for:

* timing
* controller execution
* automated testing
* timeout detection

---

### Play State

Studied:

```python
if app_utils.is_playing():
```

The controller only executes while the simulation is actively running.

---

### Controller Completion

Learned:

```python
if not controller.is_done():
```

The controller continues executing until every phase of the pick-and-place operation has completed.

---

### Controller Forward Method

Studied:

```python
controller.forward()
```

This computes the next robot action every physics frame.

The robot does not instantly jump to the goal—it gradually progresses through each phase of the task.

---

### Pick-and-Place State Machine

Studied NVIDIA's seven-state manipulation controller.

1. Move above cube
2. Approach cube
3. Close gripper
4. Lift cube
5. Move to target
6. Open gripper
7. Move away

This demonstrated how complex robot behavior can be decomposed into sequential states.

---

### Customizing Controller Timing

Learned how to modify phase durations using:

```python
events_dt=[80,60,30,60,100,30,30]
```

Changing these values adjusts how long the robot spends in each phase without modifying the controller logic.

---

### Scene Customization

Studied:

```python
controller.setup_scene(...)
```

Configured:

* cube initial position
* cube dimensions
* target placement location

This illustrates parameterized simulation design.

---

### Robotics Insight

One of today's biggest lessons was understanding the distinction between **MoveIt 2** and **Isaac Sim**.

MoveIt 2 is responsible for:

* motion planning
* inverse kinematics
* collision-free trajectories

Isaac Sim provides:

* physics simulation
* contact dynamics
* gravity
* grasp interaction
* object manipulation

Together they form complementary parts of a complete robotics software stack.

---



This software architecture can later be integrated with ROS 2, MoveIt 2, Isaac Sim, and eventually physical robotic hardware.

---

## Hours

Approximate study time:
**2.5–3 hours**

---

# LinkedIn Post

**Today's robotics lesson focused on one of the most fundamental concepts in robotic manipulation: state machines.**

I continued working through NVIDIA Isaac Sim's Core API tutorials and studied how the **Franka Pick-and-Place controller** performs an autonomous manipulation task.

Rather than commanding every joint individually, the controller progresses through a sequence of well-defined phases:

• Move above the object
• Approach
• Grasp
• Lift
• Transport
• Release
• Move away safely

I also learned how these phases can be customized using controller parameters such as `events_dt`, allowing the robot's behavior to be tuned without changing the controller's underlying logic.

Another valuable lesson was understanding the difference between **MoveIt 2** and **Isaac Sim**. MoveIt 2 plans safe, collision-free robot motions, while Isaac Sim provides the physics simulation that models contact, gravity, and object interaction. Together, they form complementary components of a modern robotic manipulation pipeline.


Every tutorial brings another piece of the robotics puzzle together.

#Robotics #IsaacSim #NVIDIA #ROS2 #MoveIt2 #Python #Automation #AgriculturalRobotics #Simulation #BeanyBot #Engineering #STEM
## This code is part of a **state machine**. The variable `self._pick_phase` keeps track of which step of the pick-and-place operation the robot is currently performing.
ChatGPT Plus





This section takes the cube you just created and makes it behave like a real physical object. Then it loads the Franka Panda robotic arm into the scene.

Without these lines:

the cube would not collide with anything,

it would not move when pushed,

and the Franka robot would not exist in the simulation.

Let's go through it line by line.

Closing Parenthesis
)
This finishes creating the cube.

At this point, the cube exists visually, but it is just a shape.

Line 1
GeomPrim(paths=cube_shape.paths, apply_collision_apis=True)
What is GeomPrim?
A GeomPrim (Geometric Primitive) represents the cube's geometric shape.

Think of it as defining:

the cube's surfaces

its dimensions

its visible geometry

paths=cube_shape.paths
Earlier you created:

cube_shape = Cube(...)
This cube lives at

/World/Cube
So

cube_shape.paths
returns something similar to:

/World/Cube
Now GeomPrim knows which object you're modifying.

apply_collision_apis=True
This is the important part.

It tells Isaac Sim:

"This object should participate in collisions."

Without collisions:

the Jetbot would drive through the cube

the Franka gripper would pass through it

gravity might ignore it

With collisions enabled:

robots can push it

the gripper can touch it

the floor can stop it from falling through

Real-world analogy
Imagine a cardboard box.

If collision is disabled:

👻 It behaves like a ghost.

You can walk right through it.

If collision is enabled:

📦 It behaves like a real box.

You bump into it.

Line 2
RigidPrim(paths=cube_shape.paths)
This makes the cube a rigid body.

What is a rigid body?
A rigid body follows the laws of physics.

It can:

fall

slide

roll

collide

be pushed

have mass

Without RigidPrim, the cube would simply sit in space like scenery.

With RigidPrim
The cube now responds to:

gravity

forces

collisions

momentum

Now the Jetbot can push it.

Real-world analogy
Imagine placing a wooden block on the floor.

If you kick it:

➡️ It slides.

That's what RigidPrim makes possible.

Line 3
cube_shape.apply_visual_materials(visual_material)
Earlier you created:

visual_material
which was bright red.

Now you're telling Isaac Sim:

"Paint this cube using that material."

Before:

⬜ White cube

After:

🟥 Red cube

Only the appearance changes.

Its physics remain exactly the same.

What has happened so far?
Your cube now has:

✅ Geometry

✅ Collision

✅ Physics

✅ Red appearance

It is now a complete physical object.

Comment
# Add Franka manipulator at a position the Jetbot will push the cube to
This explains the next step.

The Jetbot's job is:

Push Cube
The Franka's job is:

Receive Cube

↓

Pick it up
This demonstrates cooperation between a mobile robot and a robotic arm.

Line 4
stage_utils.add_reference_to_stage(
You've seen this before.

It means:

"Load another USD file into the Stage."

Previously you loaded:

Ground

Jetbot

Now you're loading:

Franka Panda

Line 5
usd_path=assets_root_path +
"/Isaac/Robots/FrankaRobotics/FrankaPanda/franka.usd",
This points to the Franka Panda robot description.

Inside that USD file are:

seven robot arm joints

the gripper

links

collision models

physics

visual meshes

Instead of building the robot manually, Isaac Sim loads everything automatically.

What is the Franka Panda?
The Franka Panda is a 7-degree-of-freedom robotic manipulator.

It has:

shoulder

elbow

wrist

parallel gripper

It is commonly used for:

pick-and-place

assembly

research

AI robotics

A simplified view looks like this:

      Gripper
         ||
         ||
       __||__
      /
     /
    /
   /
Base
Line 6
path="/World/Franka",
This tells Isaac Sim where to place the robot in the Stage.

The Stage now becomes:

World
│
├── ground
├── Jetbot
├── Cube
├── Franka
└── Materials
Later the program can access the robot by referring to:

"/World/Franka"
Closing Parenthesis
)
The Franka robot is now loaded into the simulation.

What the completed scene looks like
                 Franka Arm
                     🤖
                     │
                     │
                🟥 Cube
                     ▲
                     │
                 Jetbot

=================================
            Ground
The intended workflow is:

The Jetbot drives toward the cube.

The Jetbot pushes the cube across the floor.

The cube moves because it has collision detection and rigid-body physics.

The Franka robot waits where the cube is delivered.

The Franka can then grasp or manipulate the cube.

Execution flow
Cube created
      │
      ▼
Enable collisions
      │
      ▼
Enable rigid-body physics
      │
      ▼
Paint cube red
      │
      ▼
Load Franka robot
      │
      ▼
Scene now contains:

Ground
Jetbot
Cube
Franka
Why this matters for BeanyBot
This example demonstrates a common robotics pattern: one robot transports an object, and another robot manipulates it. For BeanyBot, the same concepts could be adapted to agricultural tasks. A mobile robot might carry harvested cocoa pods to a collection point, while a robotic arm sorts or loads them. The same building blocks—collision-enabled objects, rigid-body physics, and articulated robots loaded from USD files—will be used when you create more advanced farm simulations.


 # Position Franka so the cube will be pushed into its workspace
        franka_xform = XformPrim("/World/Franka")
        franka_xform.set_world_poses(positions=np.array([[0.8, -0.5, 0.0]]))

    # -- End of setup_scene -- #

This is the last part of setup_scene(). The Franka robot has already been loaded into the simulation. Now you're moving it to a specific location so it can interact with the cube after the Jetbot pushes it.

Let's go through it line by line.

Comment
# Position Franka so the cube will be pushed into its workspace
This comment explains the goal.

The Franka robot is not being placed randomly.

It is being positioned so that:

The Jetbot pushes the cube.

The cube arrives near the Franka.

The Franka can reach the cube with its arm.

This creates a simple cooperative robotics scenario.

Line 1
franka_xform = XformPrim("/World/Franka")
What is XformPrim?
XformPrim stands for Transform Primitive.

A transform controls an object's:

Position

Rotation

Scale

Think of it as the robot's "location manager."

Earlier, you loaded the Franka robot:

stage_utils.add_reference_to_stage(...)
That created the robot.

Now:

XformPrim("/World/Franka")
creates an object that lets you move that robot.

"/World/Franka"
This tells Isaac Sim exactly which object you're controlling.

In the Stage:

World
│
├── ground
├── Jetbot
├── Cube
└── Franka
You're saying:

"Give me control over the transform of /World/Franka."

The result is stored in the variable:

franka_xform
Line 2
franka_xform.set_world_poses(
    positions=np.array([[0.8, -0.5, 0.0]])
)
This actually moves the robot.

set_world_poses()
This function sets an object's position in the world coordinate system.

Think of it as saying:

"Move the Franka robot to these coordinates."

positions=np.array(...)
The coordinates are:

X = 0.8
Y = -0.5
Z = 0.0
Let's interpret each one.

X = 0.8
0.8 meters
The Franka is placed 80 cm forward from the world's origin.

Y = -0.5
-0.5 meters
The robot is shifted 50 cm to one side (negative Y direction).

This prevents it from sitting directly in front of the Jetbot and gives it a reachable workspace beside the cube's expected path.

Z = 0.0
0.0 meters
The base of the Franka sits on the ground.

Why move the Franka?
Suppose the robot stayed at the world's origin.

The Jetbot might collide with it before reaching the cube.

Instead, the scene is arranged so that:

the Jetbot has room to drive,

the cube can be pushed safely,

and the Franka can reach the cube without blocking the Jetbot.

Top-down view (simplified)
              Franka
                 🤖
                /
               /
              /

Jetbot ---> 🟥 Cube
The Jetbot pushes the cube toward the Franka's workspace.

Why use NumPy?
np.array([[0.8, -0.5, 0.0]])
Isaac Sim often expects positions as arrays because it can move multiple objects at once.

For a single object, the outer brackets still follow that expected format.

Comment
# -- End of setup_scene -- #
This marks the end of the setup_scene() method.

At this point, the virtual world has been completely assembled.

What exists in the scene now?
The Stage contains:

World
│
├── Ground
├── Jetbot
├── Cube
├── Franka
└── Materials
      └── Red
And each object has been configured:

Ground provides a floor.

Jetbot is ready to drive.

Cube has collision, rigid-body physics, and a red material.

Franka has been positioned where it can reach the cube.

Overall setup flow
Create Stage
      │
      ▼
Add Ground
      │
      ▼
Add Jetbot
      │
      ▼
Create Red Cube
      │
      ▼
Enable Cube Physics
      │
      ▼
Load Franka
      │
      ▼
Move Franka to (0.8, -0.5, 0.0)
      │
      ▼
Scene ready for simulation
Why this matters for BeanyBot
This example demonstrates an important robotics principle: careful placement of robots and objects within a shared workspace. In a future BeanyBot simulation, you might position a mobile robot so it delivers harvested cocoa pods into the working area of a robotic arm or collection station. The same technique—using an XformPrim and set_world_poses()—allows you to place robots, baskets, trees, charging stations, or other equipment precisely where they need to be in your simulated cocoa farm.


Wednesday 3:49 PM
Screenshot 2026-07-22 at 3.40.53 PM.png
Screenshot 2026-07-22 at 3.47.08 PM.png
Screenshot 2026-07-22 at 3.47.15 PM.png
Screenshot 2026-07-22 at 3.47.21 PM.png
Val, here is the response from the mentor. I can work on my own project because Keto doesn't seem to have the time to work with me. He keeps saying he is too busy, but all I ever see him do is play online games. So, back to the present, how about I have Tiago, in a simulated walk to a room through people, desks, items lying on the ground, and perhaps through a dedicated path, go to the professor and pick up a can of Sprite from the professor's desk and carry it away? here was the old research paper tasks: "Physically Safe Autonomous Navigation Around Humans: A TIAGo Case Study

                                        Abstract

This paper describes a simulation in which a PAL Robotics TIAGo robot autonomously traveled from Room 1 to Room 3 while avoiding stationary human figures placed along its route. The simulation was completed in Gazebo using ROS 2 Humble, Nav2, AMCL, and RViz2. TIAGo reached the goal without colliding with the human models and without using any Nav2 recovery behaviors. The robot did not recognize the models as people. It detected them as physical obstacles and used the Nav2 costmaps to plan a safe route around them. This demonstration focuses on physical safety only. It does not show social navigation behaviors such as respecting personal space, predicting human movement, yielding, or understanding human intent.

Index Terms— autonomous navigation, human-robot interaction, Nav2, ROS 2, TIAGo, obstacle avoidance.

I. Introduction

A robot working around people must be able to move safely without hitting anyone. However, avoiding a collision is not the same as understanding people or following social rules.

Kruse et al. explain that human-aware navigation includes both physical safety and social behavior [1]. A robot may avoid contact with a person but still move too close, approach from an uncomfortable direction, or behave in a way that is difficult for the person to understand.

The purpose of this demonstration was to test the physical part of human-robot interaction. I wanted to see whether a simulated TIAGo robot could travel through several rooms while avoiding human figures placed in its path.

The demonstration did not include person recognition, facial recognition, human intent prediction, or social decision-making. TIAGo treated the human figures as obstacles in the same way it would treat a wall, chair, table, or box.

II. System Setup

The demonstration used a simulated PAL Robotics TIAGo mobile manipulator in Gazebo. The software environment was ROS 2 Humble.

Nav2 controlled TIAGo’s autonomous navigation. AMCL was used to localize the robot on a known map. RViz2 was used to view the map, costmaps, robot position, planned path, and navigation status.

The simulation environment contained several connected rooms. TIAGo started in Room 1 and was given a navigation goal in Room 3.

Four stationary human models were placed in the environment. One figure was located near the starting room, two figures were placed close together along the route, and another figure was positioned separately farther ahead.

The human figures were placed where they would interfere with a direct route to the destination. This required Nav2 to plan a path around them.

III. Method

TIAGo was first spawned in Room 1. AMCL was activated so that the robot could estimate its position on the map.

After localization was established, the human models were inserted into the Gazebo world. At least one model was placed directly in the path TIAGo would normally use to travel toward Room 3.

I then used the RViz2 2D Goal Pose tool to select the destination. Nav2 calculated the route and controlled the robot during the entire trip.

No keyboard teleoperation was used to guide TIAGo around the figures. Once the goal was issued, the robot completed the task autonomously.

The simulation was recorded from start to finish. I observed whether TIAGo changed its route, avoided the human figures, reached the destination, or activated any recovery behaviors.

IV. How the Avoidance Worked

TIAGo avoided the human figures through standard Nav2 obstacle detection and path planning.

The robot’s simulated sensors detected objects in the environment. When the sensors detected a human model, Nav2 did not identify it as a person. The model simply appeared as an occupied area.

Nav2 uses costmaps to represent the environment [2]. The obstacle layer marks areas where the robot’s sensors detect a physical object. The inflation layer then places additional cost around that obstacle.

The inflated area creates a safety buffer. The cells closest to the obstacle have the highest cost, while cells farther away have lower cost.

The global planner uses this information when calculating a path to the goal. A route that passes through an occupied area is rejected. A route that passes very close to an obstacle may also have a higher cost than a route with more clearance.

The local controller continues checking the nearby costmap while TIAGo is moving. This allows the robot to adjust its movement based on current sensor information.

In this demonstration, the two human figures standing close together created overlapping inflated areas. Nav2 treated the pair as one larger obstacle region. TIAGo moved around the outside instead of attempting to travel between them.

The separately positioned figure created a smaller independent obstacle region. TIAGo made a smaller route adjustment when passing that figure.

The size of each detour depended on the size and location of the inflated costmap area. It did not depend on any social understanding of the people.

V. Results

TIAGo successfully traveled from Room 1 to Room 3.

At the start of the test, the planned route curved away from the person near the starting location. This showed that the obstacle and inflation layers were affecting the route before TIAGo began moving.

As the robot entered the corridor, the path continued to bend around occupied areas. TIAGo followed the planned route without contacting the human figures or the walls.

The largest detour occurred near the two people standing close together. Their inflation regions overlapped, creating one larger high-cost area. TIAGo moved around the outside of that area rather than passing between them.

The robot made a smaller adjustment around the person who was standing separately.

TIAGo reached the destination successfully. No collisions were observed during the test.

The Nav2 recovery counter remained at zero. The robot did not need to spin in place, back up, clear the costmaps, or use another recovery behavior.

The task was completed through normal Nav2 planning and obstacle avoidance.

VI. Physical Safety and Social Navigation

This demonstration showed basic physical safety. TIAGo detected obstacles and avoided making contact with them.

It did not demonstrate full social navigation.

A socially aware robot would need to recognize that an obstacle is a person and respond differently than it would to furniture or a wall.

For example, a robot may need to leave more personal space around a person than the minimum distance needed to prevent a collision. It may also need to slow down, yield, avoid moving between people, or approach from a direction that feels less threatening.

The robot may also need to move in a way that makes its intentions clear. A person should be able to understand where the robot is going and whether it plans to stop, turn, or pass.

None of those behaviors were included in this simulation. The human models were treated as stationary geometric obstacles.

For that reason, the result should be described as physically safe navigation around humans, not complete human-aware or socially aware navigation.

VII. Limitations and Future Work

The main limitation is that the human models did not move. The simulation therefore did not test what would happen if a person suddenly walked across TIAGo’s path.

The robot also did not classify the obstacles as people. All detected objects received the same basic costmap treatment.

The test used only one environment and one arrangement of human figures. More trials would be needed to test different room layouts, corridor widths, robot speeds, and numbers of people.

Simulation also does not include every problem that may occur with a physical robot. Real sensors may experience noise, blind spots, communication delays, or inaccurate readings.

A future version of the project could include walking human models. Person detection could also be added so that people receive a larger safety area than ordinary objects.

Another improvement would be a custom Nav2 costmap layer that creates different safety zones around people depending on their position and facing direction. TIAGo could also reduce its speed when moving near a person.

These changes would move the project beyond basic collision avoidance and closer to the human-aware navigation methods discussed by Kruse et al. [1].

VIII. Conclusion

This case study demonstrated that a simulated TIAGo robot could autonomously navigate from Room 1 to Room 3 while avoiding four stationary human figures.

The robot reached the destination without collision and without using Nav2 recovery behaviors.

The avoidance was produced by standard geometric obstacle detection. TIAGo’s sensors detected the models, the Nav2 costmaps marked their locations, and the planner selected a route around the inflated obstacle areas.

The demonstration supports a limited but important conclusion: TIAGo can use Nav2 to maintain basic physical safety around stationary people in a simulated indoor environment.

The robot did not understand that the obstacles were human, predict their behavior, or follow social rules. Those abilities remain future work.

References

[1] T. Kruse, A. K. Pandey, R. Alami, and A. Kirsch, “Human-aware robot navigation: A survey,” Robotics and Autonomous Systems, vol. 61, no. 12, pp. 1726–1743, Dec. 2013.

[2] Open Navigation LLC, “Nav2 Costmap 2D,” Navigation 2 Documentation. [Online]. Available: https://docs.nav2.org. Accessed: Jul. 2026." Val, she says the deadline was changed to the 3 of Aug but, I still see the same July 24, 2026 deadline?

 


I can definitely review that video, but I can't open a Google Drive link directly from chat.

Please do one of these:

Upload the video directly into this chat (preferred), or

Download it from Google Drive and drag the .mp4 or .mov file into this conversation.

Once I have the video, I'll analyze it frame by frame and tell you:

✅ Whether the Nav2 navigation is working correctly.

✅ Whether TIAGo is avoiding people and obstacles properly.

✅ Whether the path planning in RViz looks correct.

✅ Whether the arm is positioned correctly for grasping.

✅ Exactly why the grasp succeeds or fails.

✅ What needs to change before your mentor arrives tomorrow.

Tomorrow's plan
I know your mentor is making a surprise visit to check your progress. Based on what she told you, I would focus on producing one polished demonstration instead of several incomplete ones.

The target demonstration should be:

Part 1
Room 1
      ↓
Autonomous navigation with Nav2
      ↓
Avoid people
      ↓
Avoid desks
      ↓
Reach Room 3
Then:

Part 2
Arrive at professor's desk
      ↓
Move arm with MoveIt
      ↓
Open gripper
      ↓
Grasp Sprite can
      ↓
Lift can
      ↓
Carry can away
If we can get that working in Gazebo, it naturally strengthens your paper because it demonstrates both autonomous navigation and mobile manipulation.

I'm ready to help tomorrow morning starting at 8:00 AM Pacific. While I can't stay continuously connected between messages, every time you send me screenshots, terminal output, code, or videos, I'll help you troubleshoot the next step as quickly and methodically as possible. I think we have a realistic chance of making substantial progress before your mentor arrives.



IMG_9882.mov
File

Worked for 25s
Selwyn, I watched your video, and I have to say this is a solid foundation. It demonstrates exactly what your current paper claims.

Here's what I observed:

✅ What worked well
Gazebo simulation remained stable throughout the run.

RViz displayed the map and planned path while TIAGo was navigating.

TIAGo autonomously followed the planned route—this did not look like manual teleoperation.

The robot navigated around the environment instead of driving straight through obstacles.

The simulation completed without any obvious crashes or resets.

This supports the central claim of your current paper:

TIAGo autonomously navigated from one room to another using Nav2 while avoiding obstacles.

What would make the video even stronger
Before your mentor arrives, if possible:

Start recording a few seconds earlier so viewers can clearly see TIAGo in Room 1 before the goal is sent.

Zoom or pan briefly to RViz so the planned path and costmaps are easier to see.

At the end, pause for a few seconds after TIAGo reaches the goal so it's obvious the task is complete.

These are presentation improvements—they don't require changing the robotics.

Tomorrow's second demonstration
The manipulation video should have the same style:

Start:
TIAGo beside professor's desk

↓

MoveIt plans arm motion

↓

Gripper opens

↓

Arm moves over Sprite can

↓

Gripper closes

↓

Can lifts from table

↓

TIAGo backs away with can
If you capture that successfully, you'll have two complementary demonstrations:

Video 1: Autonomous navigation.

Video 2: Autonomous mobile manipulation.

Together, they tell a much richer robotics story than navigation alone.

My assessment
Based on the navigation video you uploaded, I think you're past the "does Nav2 work?" stage. Your remaining challenge is integrating the manipulation task with MoveIt and the gripper.

Tomorrow morning, I'd spend nearly all of your effort on getting one successful grasp and lift of the Sprite can. If that works, updating the paper is much easier because you can extend the existing results instead of rewriting everything.

I think your mentor will be much more interested in seeing a simulation that navigates and manipulates than in hearing about planned features. If you can show both behaviors in Gazebo, you'll have a compelling demonstration to accompany the paper. Good luck tomorrow morning—I'm ready to help you troubleshoot each step as you go.


it is not impossible to pick up the can is it with Moveit2?

No. It is absolutely not impossible.

In fact, picking up objects is one of the primary purposes of MoveIt 2.

MoveIt 2 is used every day to:

✅ Pick up cubes

✅ Pick up cans

✅ Pick up bottles

✅ Pick up tools

✅ Pick up boxes

✅ Pick up laboratory objects

A Sprite can is a very reasonable object for a mobile manipulator like TIAGo.

What MoveIt 2 actually does
MoveIt 2 is responsible for planning the arm motion:

Current arm position
        ↓
Plan collision-free motion
        ↓
Move above can
        ↓
Lower to grasp position
        ↓
Close gripper
        ↓
Lift object
The difficult part is usually not MoveIt itself. It's making sure everything else is configured correctly.

Typical reasons a grasp fails include:

The arm isn't centered over the can.

The wrist orientation is incorrect.

The gripper closes too early or too late.

The can's collision geometry doesn't match its visual model.

The can isn't configured as a movable physics object in Gazebo.

The fingers don't generate enough friction or force to hold the can.

The object isn't attached to the robot after the grasp (depending on how the simulation is implemented).

None of those mean the task is impossible—they're engineering issues that can be diagnosed and fixed.

Why I'm optimistic
You already have the hardest part working:

✅ Gazebo

✅ ROS 2

✅ Nav2

✅ AMCL

✅ RViz

✅ TIAGo navigation

Adding manipulation is building on that foundation.

Tomorrow
I don't want us to guess.

We'll go through it methodically:

Can MoveIt plan to a pose above the can?

Does the arm execute that plan?

Is the gripper centered on the can?

Does the gripper close?

Does the can lift?

As soon as one step fails, we'll know exactly what to fix.

I think this is a realistic goal for tomorrow morning. It's challenging, but it's a standard capability for a simulated TIAGo using MoveIt 2, and we'll focus on identifying the specific obstacle rather than assuming the whole task can't be done.


```python
elif self._pick_phase == 3:
    # Lift cube (wait 100 steps)
```

Here's what each part means:

* **`elif`** = "else if." It checks another condition only if the previous `if` or `elif` conditions were false.
* **`self._pick_phase`** = A variable that stores the robot's current phase of the task.
* **`== 3`** = Checks whether the robot is currently in **Phase 3**.
* **`# Lift cube (wait 100 steps)`** = A comment for humans. Python ignores comments. It tells you that Phase 3 is responsible for lifting the cube and remaining in that phase for about **100 simulation steps**.

### Typical Pick-and-Place Sequence

Many Isaac Sim examples use a sequence similar to this:

| Phase | What the robot does                 |
| ----: | ----------------------------------- |
|     0 | Move above the cube                 |
|     1 | Lower the gripper toward the cube   |
|     2 | Close the gripper to grasp the cube |
| **3** | **Lift the cube upward**            |
|     4 | Move toward the destination         |
|     5 | Lower the cube                      |
|     6 | Open the gripper to release it      |
|     7 | Move the arm away                   |

### Why wait 100 steps?

The simulation advances one **physics step** at a time. Instead of instantly jumping upward, the controller often waits for about 100 physics updates so the motion completes smoothly and the cube has time to settle in the gripper.

For example, the code inside Phase 3 might look something like this:

```python
elif self._pick_phase == 3:
    self._step_count += 1

    if self._step_count > 100:
        self._pick_phase = 4
        self._step_count = 0
```

This means:

1. Stay in the lifting phase.
2. Count each simulation step.
3. After 100 steps, switch to Phase 4 (move to the target location).

### In plain English

You can think of this as the robot saying:

> "I've already grabbed the cube. Now I'll raise it into the air and keep lifting for about 100 simulation frames before moving to the next part of the task."

This lifting phase is important because it ensures the cube is safely off the table before the robot begins traveling toward the destination.
This line is one of the most important commands in the pick-and-place program. It tells the **Franka robot's end effector (gripper)** where to move.

```python
self._franka.set_end_effector_pose(
    np.array([[cube_pos[0], cube_pos[1], cube_pos[2] + 0.25]]),
    down_orient
)
```

Let's break it down piece by piece.

---

## `self._franka`

This refers to the **Franka Panda robot** object in Isaac Sim.

Think of it as:

> "Use the Franka robot."

---

## `.set_end_effector_pose(...)`

The **end effector** is the robot's hand (the gripper).

This function tells the robot:

> "Move your hand to this position and orientation."

A **pose** consists of two things:

1. **Position (x, y, z)**
2. **Orientation (rotation)**

---

## `np.array(...)`

```python
np.array(...)
```

`np` is NumPy.

It creates an array of numbers that Isaac Sim expects.

---

## `cube_pos`

Earlier in the code you'll usually see something like

```python
cube_pos = self._cube.get_world_pose()[0]
```

Suppose the cube is located at

```text
x = 0.40
y = 0.20
z = 0.03
```

Then

```python
cube_pos[0]
```

is

```text
0.40
```

the x-coordinate.

Likewise,

```python
cube_pos[1]
```

is

```text
0.20
```

the y-coordinate.

And

```python
cube_pos[2]
```

is

```text
0.03
```

the z-coordinate (height above the ground).

---

## `cube_pos[2] + 0.25`

This is the clever part.

Instead of moving directly to the cube,

the robot moves

```text
25 centimeters ABOVE it.
```

Example:

Cube:

```text
x = 0.40
y = 0.20
z = 0.03
```

Robot target:

```text
x = 0.40
y = 0.20
z = 0.28
```

because

```text
0.03 + 0.25 = 0.28
```

So the robot is hovering above the cube.

---

## Why not move directly to the cube?

Imagine reaching for a coffee mug.

You don't slam your hand into the side.

Instead you

1. move above it,
2. line up,
3. lower your hand,
4. grab it.

The robot does exactly the same thing.

This position is called the **pre-grasp pose**.

---

## Why the double brackets?

Notice

```python
[[ ... ]]
```

instead of

```python
[ ... ]
```

This creates a **2-dimensional NumPy array** with one row and three columns:

```text
[
 [0.40, 0.20, 0.28]
]
```

Many Isaac Sim functions are designed to accept multiple target positions at once, so they expect a 2D array even when you're supplying just one position.

---

## `down_orient`

This is the desired orientation (rotation) of the gripper.

Instead of only specifying **where** to go, you must also specify **how** the gripper is rotated.

Typically,

```python
down_orient
```

means

> "Point the fingers straight downward."

So while the robot is above the cube,

its gripper is already facing the correct direction for grasping.

---

## Putting it all together

If

```text
Cube Position
x = 0.40
y = 0.20
z = 0.03
```

then this command tells the robot:

```text
Move the gripper to

x = 0.40
y = 0.20
z = 0.28

while pointing straight downward.
```

The robot stops **25 cm above the cube**, ready for the next phase where it lowers the gripper, closes its fingers around the cube, and eventually lifts it. This "approach from above" is a standard and reliable strategy in industrial robot manipulation because it reduces the chance of colliding with the object or the table.
This code controls **when the robot leaves Phase 3 (lifting the cube)** and **enters Phase 4 (moving to the destination)**.

```python
if self._step_counter > 100:
    self._pick_phase = 4
    self._step_counter = 0
elif self._pick_phase == 4:
    # Move to target (wait 150 steps)
```

Let's examine it line by line.

---

## `if self._step_counter > 100:`

The robot has a variable called `self._step_counter`.

Every physics frame, the program usually does something like:

```python
self._step_counter += 1
```

So the counter increases like this:

```text
Step 1
Step 2
Step 3
...
Step 100
Step 101
```

This line asks:

> **"Have we been lifting the cube for more than 100 simulation steps?"**

If the answer is **yes**, the code inside the `if` statement runs.

---

## `self._pick_phase = 4`

This changes the robot's state.

Before:

```text
_pick_phase = 3
```

Meaning:

> Lift the cube.

After:

```text
_pick_phase = 4
```

Meaning:

> The lift is complete. Start moving toward the destination.

It's like advancing to the next step in a recipe.

---

## `self._step_counter = 0`

The counter is reset back to zero.

Why?

Because Phase 4 has its own timing.

If you didn't reset it, the counter would already be over 100, and Phase 4 might end almost immediately instead of lasting the intended amount of time.

So resetting gives Phase 4 a fresh timer.

---

## `elif self._pick_phase == 4:`

On the next simulation frame, `_pick_phase` is now `4`.

That means the program skips the earlier phases and executes the Phase 4 code.

The comment says:

```python
# Move to target (wait 150 steps)
```

So during this phase, the robot begins moving the cube to its goal location.

Typically, the code might look something like:

```python
elif self._pick_phase == 4:
    self._franka.set_end_effector_pose(target_pos, down_orient)

    self._step_counter += 1

    if self._step_counter > 150:
        self._pick_phase = 5
        self._step_counter = 0
```

This means:

1. Move toward the target.
2. Count simulation steps.
3. After 150 steps, advance to Phase 5 (usually lowering the cube).

---

## Timeline of what happens

```text
Phase 3
───────────────
Lift cube
Counter = 0

↓
Counter = 1
Counter = 2
Counter = 3
...
Counter = 100
Counter = 101

↓

Counter > 100?

YES

↓

_pick_phase = 4
_step_counter = 0

↓

Phase 4 begins

Move toward destination

↓

Counter = 1
Counter = 2
...
Counter = 150

↓

Advance to Phase 5
```

---

## In plain English

Imagine you're carrying a box across a room:

* **Phase 3:** Lift the box off the floor.
* Wait until you've finished lifting it (about 100 simulation steps).
* **Switch to Phase 4.**
* Reset the timer.
* **Phase 4:** Walk across the room carrying the box.
* Stay in that movement phase for about 150 simulation steps before continuing to the next part of the task (such as lowering and releasing the box).

This is an example of a **finite state machine**, where the robot progresses through one well-defined stage at a time: **approach → grasp → lift → transport → lower → release**.
### **Yes — you will see the end-effector move in real time.**

In Isaac Sim, when your code runs (whether it’s a Python script, an Action Graph, or a custom extension) and calls something like:

```python
self._franka.set_end_effector_pose(...)
```

the robot arm physically moves in the 3D viewport as the simulation steps forward. You watch the gripper travel to the new pose, lower, close, lift, and transport the object live on screen — exactly like watching a real robot.

### How it works in practice
- You press **Play** (or run your script while the simulation is playing).
- Every physics step, the robot controller tries to reach the target pose you set.
- The arm animates smoothly (or with whatever controller gains/speed you configured).
- You can orbit the camera, zoom in on the gripper, or even attach the camera to the end-effector if you want a first-person view.

You can also:
- Watch the live pose values in the Property panel or via Python printouts / UI overlays.
- Pause the simulation at any moment to inspect the exact current end-effector pose.
- Step frame-by-frame if you want to debug slowly.

So yes — as you command the approach, descend, close, lift, and place, you will see every movement happen in real time right in front of you. That’s one of the best parts of learning in Isaac Sim.
## Continuing with Issac Sim:
Absolutely. The key point is that **the gripper usually does *not* rotate after grasping** in the code you've been studying. The fingers close around the cube, but the wrist keeps the same downward orientation.

### Pre-grasp (hovering above the cube)

![Image](https://images.openai.com/static-rsc-4/Qn8UHxQ2U2hWhKLQW9x3qRztCqYfpWaQDeQXOoe6BGyZTSCDZMv9lcsSYSNs7R_na1E4h4R5XGjO2tbK4TRC2B85w7hXPLio3vCXhuWDVygiI1x8dTh55Zjv-lRhu8X83XudDq6Mpjmr3FWr8o8E7PvVBXQvrv9ivoNQzYFOGVDK5Xpn-42OdTClEIqD5S6n?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/FLEF11L1aNI9EG9F7z2VIZXnDw73tRds-4Ezp66tPGnbbCyQuLuEQx3bdd5sh09pPaKd8lJiNLsenZa8BjHkH4l3E0GFuuwxtd8uObSTnTihDz7gR41N8CZXNXTCKRW4BgqH2vMxFYcgY6A2vLz1Qyu6MSCFUW99Ik3jd0YrwK8GgLez6_AWm0DNuoEaKgN-?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/CCzcm893ifZN5gfPwhGBGlwqWtFjQtbQ8g5EIYO6Bmy7AcrXvaIVlnxdAaJPahcMJmNDUF4R17VZ5Az1XfmsBDFaErNw9PQxTS0Y6RNLrj1VuQF6hRryLiY5nBfi4ehZ0pTSazLJnnjMg9TP4FuBXONhdfWvOJTOnxuHBjRtkpVXGSiuka_chKGkVLWgaTnC?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/uJavlgWCcTwZsGAWJJAv7_LqgOuASzGEmIFmoJBHb35ah4gXJwrxUI6wsHNRsi3NkhZvmL7h3wCOfua8jx9j4p1l4goDESFHx_Rl3mIZy1AYdtajD_AFwDNu7DxZc5VXZYIy8DmRee7l3H-OyRM0P8t9kH3zctIs_2lytXNrgIoTZ6Pzoige5J6MvdCOjYhm?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/gdBSQM0AXGPG1CDdmMBVcyAGCXfD2OhJAi-iroih0CA5KBWZi7Snokr5rVic4LdrNzV88ERKtVEQAjtdceeC4xqlnqFu4QXGAizRB6lqsMvvlq0n9l_eCFZx73wGGE2W4rGu2zxdXNeTI1TfqEe49bE5Do8kukztXKL62PlnMr3FmJ7yR3XVzMDC-lY-zhz1?purpose=fullsize)

In this stage:

* Fingers are **open**.
* Gripper is **pointing downward**.
* The robot is hovering directly above the cube.
* The command is similar to:

```python
self._franka.set_end_effector_pose(
    np.array([[cube_x, cube_y, cube_z + 0.25]]),
    down_orient
)
```

The `+0.25` places the gripper **25 cm above** the cube.

---

### Post-grasp (after picking up the cube)

![Image](https://images.openai.com/static-rsc-4/YfGWqJw_iGG1WCb7OXiHfaK3cYQ1TjPv5yediTY-Njxcy-W_Fp63CdbTpD20jETZVfugwGkVk4ZjCeW3zQUr1VkrdrrI2P5CsGwxvQXlTTLPQbM26Wv33DfDAW78axlT35osIdfgVHgvX854mDFEnVPCkmKd7HZoFdpZa9JPXC7vkKbc1u2s2PDLzL0KZICk?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/iPcFiThnIsOb0DJo5xrN6JVlgfh3RmTg3LIyslR8XrlHkdFPvKghwM5QNkT5QeICEktZV6Cln5b25t8jz5mlfyD0vkPlAgGEW4hyCj4_XFLVSlWfJ2WAGVQEGg06Ms-Dawlk8xAMl792NBDSOFvFtquO6Ytwg9iKQe2x6DDkfg4Yj3Df1tboi2hfCOdY3RGG?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/FLEF11L1aNI9EG9F7z2VIZXnDw73tRds-4Ezp66tPGnbbCyQuLuEQx3bdd5sh09pPaKd8lJiNLsenZa8BjHkH4l3E0GFuuwxtd8uObSTnTihDz7gR41N8CZXNXTCKRW4BgqH2vMxFYcgY6A2vLz1Qyu6MSCFUW99Ik3jd0YrwK8GgLez6_AWm0DNuoEaKgN-?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/gdBSQM0AXGPG1CDdmMBVcyAGCXfD2OhJAi-iroih0CA5KBWZi7Snokr5rVic4LdrNzV88ERKtVEQAjtdceeC4xqlnqFu4QXGAizRB6lqsMvvlq0n9l_eCFZx73wGGE2W4rGu2zxdXNeTI1TfqEe49bE5Do8kukztXKL62PlnMr3FmJ7yR3XVzMDC-lY-zhz1?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/VQaXR5HvAR3NObfK5R86xFQ33k88zD1nR17dHlqXi1d49MKDS6IBxXvtadJJV76iPD5Win6S6hagOLB0DhlVtby1kE2LGmXq0hUriM_dEYtrCBcCDc42ReDT8YuNZDZAJpEDg5zkqRhs-DAcVxd5-EDEmeOCtNmtooSJKVqd1vXk5tF8l9ap_AKX2LKCTP3L?purpose=fullsize)

Notice what has changed:

* ✅ The fingers are now **closed** around the cube.
* ✅ The cube is lifted off the table.
* ✅ **The wrist is still pointing downward.**

The robot simply raises its arm. It does **not** automatically rotate the gripper so the "palm" faces upward.

Think of it like this:

```text
PRE-GRASP

      ↓
     | |
     | |   ← fingers open
      □

---------------- Table ----------------
```

After grasping:

```text
POST-GRASP

      ↓
     |█|   ← fingers closed
      □

        ↑
     Cube lifted
```

The arrow is still pointing **down** because the wrist orientation hasn't changed—only the **position** (higher `z`) has changed.

If you wanted the robot to rotate the cube so the "palm" faced upward, the code would have to change the orientation from `down_orient` to a different orientation (a different quaternion). That is **not** happening in the code you've been walking through.

If it would help, I can also draw a **3D side-view diagram** showing the arm, wrist, gripper, and cube before and after the grasp. Many students find that easier to understand than photographs.
This is **Phase 5** of the state machine. At this point, the robot has **already moved to the target location and opened its gripper** to release the cube.

Here's the code:

```python
elif self._pick_phase == 5:
    # Lift the arm (wait 150 steps)
    self._franka.set_end_effector_pose(
        np.array([[cube_pos[0], cube_pos[1], cube_pos[2] + 0.5]]),
        down_orient
    )
```

Let's go through it line by line.

---

## `elif self._pick_phase == 5:`

The robot asks:

> "Am I in Phase 5?"

If yes, it executes the code below.

---

## `# Lift the arm (wait 150 steps)`

This is a comment for the programmer.

It tells us the purpose of Phase 5:

> **Raise the robot's arm away from the object after releasing it.**

This is sometimes called a **retreat motion**.

---

## `self._franka.set_end_effector_pose(...)`

This tells the robot where to move its gripper.

The new position is

```python
np.array([[cube_pos[0], cube_pos[1], cube_pos[2] + 0.5]])
```

---

## `cube_pos[0]`

Keep the same **x-coordinate**.

Example:

```text
x = 0.40
```

---

## `cube_pos[1]`

Keep the same **y-coordinate**.

Example:

```text
y = 0.20
```

---

## `cube_pos[2] + 0.5`

Raise the gripper **50 centimeters above the cube's height**.

Suppose the cube is resting on the table at

```text
x = 0.40
y = 0.20
z = 0.03
```

The new target becomes

```text
x = 0.40
y = 0.20
z = 0.53
```

because

```text
0.03 + 0.50 = 0.53
```

So the gripper moves straight upward.

---

## `down_orient`

The wrist orientation is **still the same**.

The gripper remains pointing downward.

Only the **position** changes.

---

## Why lift the arm after releasing the cube?

Imagine placing a box on a table.

After letting go, you don't leave your hand resting on the box. You naturally pull your hand away.

The robot does the same thing:

1. Place the cube.
2. Open the gripper.
3. Raise the arm so it won't bump into the cube or table.
4. Finish the task.

---

## Visual example

### Before Phase 5 (just released)

```text
     ↓
    |  |
     □

────────────── Table
```

The fingers are open, and the cube is sitting on the table.

---

### During Phase 5

```text
       ↓
      |  |




     □

────────────── Table
```

The cube stays where it is.

The **robot's arm moves upward**, leaving the object behind.

---

## One thing to notice

There is something interesting about this code.

Earlier, the robot carried the cube to the destination:

```python
self._franka.set_end_effector_pose(
    np.array([[0.3, 0.3, 0.15]]),
    down_orient
)
```

But now it uses

```python
cube_pos[0], cube_pos[1], cube_pos[2] + 0.5
```

This only makes sense if `cube_pos` has been updated to the cube's **current location after it was released**. If `cube_pos` still contains the cube's original pickup location, the robot would move back above the original pickup point instead of retreating from the drop-off location.

So whether this behaves correctly depends on where `cube_pos` is updated in the rest of the program.

### In plain English

This phase tells the robot:

> **"The cube has been released. Now raise your gripper 50 cm straight upward while keeping it pointed downward. This safely moves the arm away from the object so the pick-and-place task can end without colliding with the cube or the table."**
This is a very simple but important piece of the robot's **timing logic**.

```python
if self._step_counter > 150:
    self._step_counter = 0
```

Let's examine it.

---

## `if self._step_counter > 150:`

This asks:

> **"Has the robot been in this phase for more than 150 simulation steps?"**

Remember that somewhere else in the code, the counter is probably being increased every physics frame:

```python
self._step_counter += 1
```

So the value changes like this:

```text
0
1
2
3
...
149
150
151
```

When it reaches **151**, the condition becomes true because:

```text
151 > 150
```

---

## `self._step_counter = 0`

Once the condition is true, the counter is reset.

Before:

```text
_step_counter = 151
```

After:

```text
_step_counter = 0
```

This gives the next phase (or the next use of the counter) a fresh starting point.

---

## Why reset it?

Imagine using a kitchen timer.

You set it for **2 minutes** to bake cookies.

When the timer rings, you don't leave it showing "2:00" forever—you reset it before timing the next task.

The robot's step counter works the same way.

---

## Example

Suppose the robot is lifting its arm.

| Simulation Step | `_step_counter` | Action                            |
| --------------: | --------------: | --------------------------------- |
|               1 |               1 | Continue lifting                  |
|              50 |              50 | Continue lifting                  |
|             100 |             100 | Continue lifting                  |
|             150 |             150 | Continue lifting                  |
|             151 |             151 | Condition becomes true            |
|     After reset |               0 | Timer is ready for the next phase |

---

## One important observation

Notice that this code **only resets the timer**:

```python
if self._step_counter > 150:
    self._step_counter = 0
```

It does **not** change the robot's phase.

Earlier in your program, you saw code like:

```python
self._pick_phase = 4
```

or

```python
self._pick_phase = 5
```

Those lines tell the robot to move to the next stage.

This snippet **doesn't do that**. It simply resets the timer.

That means one of two things is likely true:

1. **More code follows** (perhaps on the next line) that changes `_pick_phase`, or
2. **Phase 5 is the final phase**, and the author only wants to stop the timer because there are no more phases.

---

### In plain English

This code tells the robot:

> **"Once you've been in this phase for more than 150 simulation steps, reset your timer back to zero."**

By itself, it **doesn't make the robot do anything new**—it only prepares the timer for whatever comes next in the program.
Summary
This tutorial covered the following topics:

Adding multiple robots and objects (cube) to the scene

Using Cube, GeomPrim, and RigidPrim to create pushable objects

Using the Articulation class to control different robot types

Having a mobile robot (Jetbot) push objects towards a manipulator (Franka)

Building state machine logic to coordinate pushing, backing up, and picking

Using Franka for IK-based end-effector control and gripper operations

Next Steps
Continue on to the next tutorial in our Essential Tutorials series, Multiple Robot Scenarios, to learn how to add multiple tasks and manage them.
## Multiple Robot Scenarios
Learning Objectives
This tutorial describes how to create and manage multiple robot scenarios in NVIDIA Isaac Sim. It explains how to use parameterization and Python classes to scale your simulations with multiple instances of robots performing similar tasks. After this tutorial, you will have more experience building scalable multi-robot simulations in NVIDIA Isaac Sim.

15-20 Minute Tutorial:
Day_Log 07/28/2026:
Let's go through each line one at a time.

```cpp
#include <memory>
```

### What it does

This tells the C++ compiler to include the **Standard Library memory utilities**.

### Why it is needed

Behavior Tree plugins and ROS 2 frequently use **smart pointers** instead of ordinary pointers.

Examples include:

* `std::shared_ptr`
* `std::unique_ptr`
* `std::weak_ptr`

These automatically manage memory so you don't have to manually call `delete`.

Example:

```cpp
std::shared_ptr<MyClass> node;
```

Without

```cpp
#include <memory>
```

the compiler would not know what `std::shared_ptr` is.

---

```cpp
#include <string>
```

### What it does

Loads the C++ Standard Library string class.

Instead of using old C-style strings like

```cpp
char name[20];
```

modern C++ uses

```cpp
std::string
```

Example:

```cpp
std::string robot_name = "TIAGo";
```

Behavior Tree nodes often receive strings from XML ports.

Example:

```cpp
std::string goal;
```

---

```cpp
#include "rclcpp/rclcpp.hpp"
```

### What it does

This includes the main **ROS 2 C++ client library**.

Think of this as loading the entire ROS 2 toolbox into your program.

Without this header, your program cannot use ROS 2 features.

Examples include:

* Nodes
* Publishers
* Subscribers
* Timers
* Parameters
* Logging
* Services
* Clients

---

### Example

Logging:

```cpp
RCLCPP_INFO(logger, "Robot started");
```

Creating a node:

```cpp
auto node = rclcpp::Node::make_shared("robot");
```

Without

```cpp
#include "rclcpp/rclcpp.hpp"
```

none of those would exist.

---

### Why does BT Navigator need this?

Your Behavior Tree node is also a ROS 2 component.

It often needs to

* print messages
* read parameters
* communicate with topics
* call services

Those capabilities come from **rclcpp**.

---

```cpp
#include "behaviortree_cpp_v3/action_node.h"
```

This is the most important include in this tutorial.

### What it does

It loads the BehaviorTree.CPP library's **Action Node** class.

Your custom node will inherit from this class.

Think of it like this:

```
BehaviorTree Library
        │
        ▼
ActionNodeBase
        │
        ▼
BT::SyncActionNode
        │
        ▼
Your LogPrinter Node
```

Without this header, C++ has no idea what an Action Node is.

---

### Why Action Node?

Behavior Trees contain different node types.

```
Behavior Tree

         Root
           │
     ┌─────┴─────┐
     │           │
 Condition    Action
```

An Action Node **does something**.

Examples:

* Move robot
* Pick up object
* Open gripper
* Speak
* Print a log
* Wait
* Rotate
* Dock

In this tutorial the action simply prints a message.

---

### Why not another node type?

BehaviorTree.CPP provides several kinds of nodes.

| Node Type | Purpose                                    |
| --------- | ------------------------------------------ |
| Action    | Performs work                              |
| Condition | Returns Success or Failure based on a test |
| Decorator | Changes the behavior of another node       |
| Control   | Chooses which child nodes run              |

Examples:

Action:

```
MoveToGoal
```

Condition:

```
BatteryOK?
```

Decorator:

```
Repeat Until Success
```

Control:

```
Sequence
Fallback
Parallel
```

---

## Putting all four includes together

```cpp
#include <memory>
```

"I want to use smart pointers."

↓

```cpp
#include <string>
```

"I want to use C++ strings."

↓

```cpp
#include "rclcpp/rclcpp.hpp"
```

"I want to use ROS 2."

↓

```cpp
#include "behaviortree_cpp_v3/action_node.h"
```

"I want to create a Behavior Tree Action Node."

---

### In simple terms

Imagine you're building a robot mechanic's toolbox:

* `<memory>` → Gives you safe containers for robot objects (smart pointers).
* `<string>` → Lets the robot work with text like names and commands.
* `rclcpp.hpp` → Gives access to the ROS 2 operating system (logging, nodes, topics, services).
* `action_node.h` → Gives you the blueprint for creating a new Behavior Tree action, such as "Move Robot," "Close Gripper," or in this tutorial, "Print a Log Message."

These four `#include` statements don't perform any actions themselves—they simply make the necessary C++ classes and functions available so the rest of the program can compile and use them.
This code is defining a **Behavior Tree (BT) node** in C++. Let's go through it one line at a time.

```cpp
namespace goal_navigation_tutorial
{
```

### `namespace goal_navigation_tutorial Day_Log 07/28/2026:

A **namespace** is simply a way to organize code and avoid name conflicts.

Think of it like folders on your computer.

For example:

```
Desktop
├── Photos
│   └── cat.jpg
├── Work
│   └── report.docx
```

Both folders could contain a file named `notes.txt` without conflicting.

In C++, a namespace does the same thing:

```cpp
namespace math
{
    int add(int a, int b);
}

namespace finance
{
    int add(int income, int expense);
}
```

Now the compiler knows which `add()` function you mean.

---

## Next

```cpp
/**
* @brief A BT node that logs a message to the console
* with a specific log level that could be INFO, WARN, ERROR.
*/
```

This is **documentation**, not executable code.

`/** ... */` is a special comment format used by documentation tools such as **Doxygen**.

`@brief` means:

> "Here is a short description."

It tells us:

> This Behavior Tree node prints a message to the console.

The message can have different importance levels:

* **INFO** – Normal information
* **WARN** – Something unusual happened
* **ERROR** – Something went wrong

For example:

```
INFO: Robot reached waypoint.
```

```
WARN: Battery is getting low.
```

```
ERROR: Cannot find navigation map.
```

---

## The class definition

```cpp
class LogPrinter : public BT::AsyncActionNode
```

This line is very important.

It creates a new class called:

```cpp
LogPrinter
```

But notice:

```cpp
: public BT::AsyncActionNode
```

This means

> **LogPrinter inherits from `BT::AsyncActionNode`.**

It is saying:

> "Build my new node using the features already provided by BehaviorTree.CPP's asynchronous action node."

Think of it like inheritance in real life.

```
          Vehicle
             ↑
          Robot Car
```

The robot car automatically has wheels, steering, etc., because it inherits from `Vehicle`.

Likewise:

```
        BT::AsyncActionNode
                 ↑
            LogPrinter
```

`LogPrinter` automatically gets the behavior of an asynchronous Behavior Tree node.

---

## What is an `AsyncActionNode`?

Behavior Trees have different kinds of nodes.

```
Behavior Tree

├── Sequence
├── Fallback
├── Condition
├── Action
└── Async Action
```

A normal action finishes almost immediately.

For example:

```
Turn on LED
```

takes only milliseconds.

An **AsyncActionNode** is used when something takes time.

Examples:

* Navigate to a room
* Wait for an elevator
* Pick up an object
* Follow a person
* Scan the environment

These actions may take several seconds or minutes.

Instead of freezing the whole Behavior Tree, they run asynchronously while the tree continues ticking.

---

## Why use `LogPrinter` as an AsyncActionNode?

Imagine your Behavior Tree contains:

```
Sequence
   |
   +--> LogPrinter
   |
   +--> NavigateToGoal
```

When `LogPrinter` runs, it might print:

```
INFO: Starting navigation.
```

or

```
WARN: Localization confidence is low.
```

or

```
ERROR: Navigation failed.
```

The Behavior Tree can continue operating while the node manages its asynchronous execution according to the BehaviorTree.CPP framework.

---

### Overall picture

```cpp
namespace goal_navigation_tutorial
{
    class LogPrinter : public BT::AsyncActionNode
```

means:

* Create a namespace named **goal_navigation_tutorial**.
* Inside that namespace, define a new class called **LogPrinter**.
* Make `LogPrinter` inherit from **BehaviorTree.CPP's `AsyncActionNode`**.
* This node's purpose is to print log messages (INFO, WARN, or ERROR) during execution of the Behavior Tree.

This is a common pattern in ROS 2 navigation and BehaviorTree.CPP: you create custom nodes by inheriting from one of the base node classes (such as `ActionNodeBase`, `SyncActionNode`, `AsyncActionNode`, or `ConditionNode`) and then implement the behavior you want. 
This is the **constructor** for the `LogPrinter` class. A constructor is a special function that runs **automatically when an object is created**. Let's go through it line by line.

```cpp
public:
```

## `public:`

A C++ class can have three access levels:

* `public` → Anyone can use these functions and variables.
* `private` → Only the class itself can use them.
* `protected` → The class and its child classes can use them.

Here, `public:` means:

> "Everything that follows can be accessed from outside the class."

---

```cpp
/**
 * @brief A constructor for goal_navigation_tutorial::LogPrinter
 *
 * @param xml_tag_name Name for the XML tag for this node
 * @param conf BT node configuration
 */
```

Again, this is **documentation** for programmers.

### `@brief`

A short description:

> This is the constructor for `LogPrinter`.

### `@param xml_tag_name`

Explains the first parameter.

For example, in a Behavior Tree XML file you might write:

```xml
<LogPrinter message="Starting navigation"/>
```

The XML tag name would be:

```text
LogPrinter
```

That name is passed into the constructor as:

```cpp
xml_tag_name
```

---

### `@param conf`

This is the **Behavior Tree configuration**.

It contains information such as:

* Input ports
* Output ports
* Blackboard connections
* Parameters from the XML

Think of it as a package of settings that the Behavior Tree engine hands to every node when it creates it.

---

## The constructor declaration

```cpp
LogPrinter(
```

This is **not** an ordinary function.

Because its name matches the class name:

```cpp
class LogPrinter
```

it is the constructor.

When you create an object:

```cpp
LogPrinter logger(...);
```

this constructor automatically runs.

---

## First parameter

```cpp
const std::string & xml_tag_name,
```

Let's break this apart.

### `std::string`

A C++ string.

Example:

```cpp
"LogPrinter"
```

---

### `&`

The ampersand means:

**Pass by reference.**

Instead of making a copy of the string, the constructor uses the original.

Imagine a 100-page document.

Without `&`:

> Make another 100-page copy.

With `&`:

> Just hand me the original document.

This saves memory and is much faster.

---

### `const`

`const` means:

> "I promise not to change this string."

So

```cpp
const std::string &
```

means:

> "Give me the original string, and I promise not to modify it."

This is a very common pattern in modern C++.

---

## Second parameter

```cpp
const BT::NodeConfiguration & conf
```

Again:

* `const` → Don't modify it.
* `&` → Don't make a copy.
* `BT::NodeConfiguration` → The configuration information supplied by BehaviorTree.CPP.

This object contains all the setup information for the node.

---

## Closing parenthesis

```cpp
)
```

The constructor now knows everything it needs:

* the XML node name
* the configuration

---

## This next line is extremely important

```cpp
: BT::AsyncActionNode(xml_tag_name, conf)
```

This is called the **constructor initializer list**.

It runs **before** the body of the constructor.

Since `LogPrinter` inherits from:

```cpp
BT::AsyncActionNode
```

the base class must be constructed first.

You're telling C++:

> "Before constructing `LogPrinter`, first construct the parent `AsyncActionNode` using these arguments."

It is equivalent in concept to saying:

```
Build the parent class first.
        ↓
Parent receives:
    xml_tag_name
    conf
        ↓
Now build LogPrinter.
```

Without this line, the parent class might not be initialized correctly.

---

## Why not do it inside `{ }`?

Imagine you have:

```cpp
class Car
{
public:
    Car(std::string color);
};
```

and

```cpp
class SportsCar : public Car
{
};
```

The `Car` constructor must run **before** the `SportsCar` constructor.

That's why C++ uses the initializer list.

The order is:

```
Create parent class
        ↓
Initialize its variables
        ↓
Create child class
        ↓
Run child constructor body
```

---

## What values might these parameters contain?

Suppose your Behavior Tree XML contains:

```xml
<LogPrinter message="Robot Started"/>
```

The Behavior Tree engine might call:

```cpp
LogPrinter(
    "LogPrinter",
    configuration
);
```

where:

* `xml_tag_name` = `"LogPrinter"`
* `conf` = all the node's configuration information

The initializer list then passes both directly to the parent class:

```cpp
BT::AsyncActionNode(
    "LogPrinter",
    configuration
);
```

so the base `AsyncActionNode` is fully initialized before `LogPrinter` adds its own behavior.

---

### Overall flow

```
Behavior Tree loads XML
          │
          ▼
Reads <LogPrinter ...>
          │
          ▼
Calls LogPrinter constructor
          │
          ▼
Initializer list executes first
BT::AsyncActionNode(xml_tag_name, conf)
          │
          ▼
Parent AsyncActionNode is built
          │
          ▼
LogPrinter construction continues
```

This pattern—using an initializer list to initialize the base class—is standard C++ and is used throughout ROS 2 and BehaviorTree.CPP because it ensures the inherited base class is correctly set up before the derived class begins executing its own logic.
Excellent. Now we're inside the **body of the constructor**. This code runs **after** the parent `BT::AsyncActionNode` has been initialized. Its job is to get the information the node needs before it can run.

Let's go through it line by line.

```cpp
{
```

This opens the constructor body.

Everything between `{` and `}` runs automatically when a `LogPrinter` object is created.

---

## Line 1

```cpp
node_ = config().blackboard->get<rclcpp::Node::SharedPtr>("node");
```

This is probably the most important line in the constructor.

Let's break it apart.

### `config()`

Remember this?

```cpp
LogPrinter(
    const BT::NodeConfiguration & conf)
```

The parent class (`AsyncActionNode`) stored that configuration for us.

Calling

```cpp
config()
```

returns the node's configuration.

---

### `.blackboard`

BehaviorTree.CPP has something called a **Blackboard**.

Think of it as a **shared notebook** that every Behavior Tree node can read from and write to.

```
             Blackboard

        +--------------------+
        | Robot Node         |
        | Goal Position      |
        | Current Battery    |
        | Current Map        |
        +--------------------+

          ↑      ↑      ↑

   Navigate   Planner   LogPrinter
```

Instead of every node making its own copy of the robot information, they all share the Blackboard.

---

### `->get<...>("node")`

This says:

> "Retrieve the object named `"node"` from the Blackboard."

The `<...>` tells C++ what type to expect.

```cpp
<rclcpp::Node::SharedPtr>
```

means:

> "I expect `"node"` to be a shared pointer to a ROS 2 node."

---

### `rclcpp::Node::SharedPtr`

A **SharedPtr** (shared pointer) is a smart pointer.

Instead of manually managing memory, several objects can safely share ownership of the same ROS node.

Imagine one robot controller used by several Behavior Tree nodes.

```
                ROS Node
             +-----------+
             | rclcpp    |
             +-----------+
              ↑    ↑    ↑
              |    |    |
         Planner Logger Navigator
```

Every node shares the same ROS node.

---

### `node_ = ...`

Store that pointer inside this class.

Now `LogPrinter` can call ROS logging functions like:

```cpp
RCLCPP_INFO(...)
```

or

```cpp
RCLCPP_WARN(...)
```

---

## Line 2

```cpp
getInput("log_text", log_text_);
```

Behavior Tree nodes receive values from the XML.

Suppose the XML contains

```xml
<LogPrinter
    log_text="Robot starting"
    log_level="INFO"/>
```

This line says:

> "Read the input port named `log_text`."

Then store it in

```cpp
log_text_
```

Afterward,

```
log_text_ = "Robot starting"
```

---

## Line 3

```cpp
getInput("log_level", log_level_);
```

Exactly the same idea.

Read

```
log_level
```

from the XML.

Now

```
log_level_ = "INFO"
```

or perhaps

```
"WARN"
```

or

```
"ERROR"
```

depending on what the XML specified.

---

## Line 4

```cpp
if (log_text_.empty()) {
```

Now we check whether

```
log_text_
```

contains anything.

Suppose someone wrote

```xml
<LogPrinter log_level="INFO"/>
```

Notice the mistake?

There is **no**

```
log_text
```

parameter.

Then

```
log_text_
```

would be an empty string.

```
""
```

Calling

```cpp
.empty()
```

asks

> "Does this string have zero characters?"

If yes,

execute the code inside.

---

## Inside the first `if`

```cpp
RCLCPP_WARN(
    node_->get_logger(),
    "LogPrinter: log_text is empty");
```

This prints a ROS warning.

The console might display

```
[WARN]
LogPrinter: log_text is empty
```

Notice

```cpp
node_->get_logger()
```

We're using the ROS node we got from the Blackboard earlier.

Without that pointer,

we couldn't log messages.

---

## Next

```cpp
if (log_level_.empty()) {
```

Now check whether the user forgot to provide

```
log_level
```

Suppose the XML is

```xml
<LogPrinter
    log_text="Starting robot"/>
```

No log level was specified.

Then

```
log_level_
```

is empty.

---

## Inside the second `if`

```cpp
RCLCPP_WARN(
    node_->get_logger(),
    "LogPrinter: log_level is empty, setting to INFO");
```

ROS prints

```
[WARN]
LogPrinter: log_level is empty,
setting to INFO
```

This warns the programmer that a value was missing.

---

## Final line

```cpp
log_level_ = "INFO";
```

Instead of crashing,

the node chooses a sensible default.

Now

```
log_level_ = "INFO"
```

Even though the XML omitted it,

the node can still continue working.

This is a common defensive programming technique: if an optional value is missing, use a safe default rather than failing.

---

# Example

Suppose the XML contains:

```xml
<LogPrinter
    log_text="Navigation started"
    log_level="WARN"/>
```

Execution looks like this:

```
Constructor starts
        │
        ▼
Get ROS node from Blackboard
        │
        ▼
node_ points to ROS node
        │
        ▼
Read log_text
        │
        ▼
log_text_ = "Navigation started"
        │
        ▼
Read log_level
        │
        ▼
log_level_ = "WARN"
        │
        ▼
Is log_text empty?
        │
       No
        │
        ▼
Is log_level empty?
        │
       No
        │
        ▼
Constructor finished
```

Now imagine this XML instead:

```xml
<LogPrinter/>
```

Execution changes:

```
Constructor starts
        │
        ▼
Get ROS node from Blackboard
        │
        ▼
Read log_text → ""
        │
        ▼
Read log_level → ""
        │
        ▼
WARNING:
log_text is empty
        │
        ▼
WARNING:
log_level is empty
        │
        ▼
Set
log_level_ = "INFO"
        │
        ▼
Constructor finished
```

So, by the time the constructor completes, the `LogPrinter` object has:

* Retrieved the shared ROS 2 node from the Behavior Tree Blackboard.
* Loaded the `log_text` and `log_level` inputs from the Behavior Tree XML.
* Warned the user if required values were missing.
* Assigned `"INFO"` as a default log level if none was provided.

This ensures the node is properly initialized and ready to execute when the Behavior Tree ticks it.
This is one of the **most important functions** in a BehaviorTree.CPP node. It tells the Behavior Tree engine:

> **"What information does my node expect to receive?"**

Let's go through it line by line.

---

```cpp
/**
 * @brief Creates list of BT ports
 * @return BT::PortsList Containing basic ports along with node-specific ports
 */
```

This is documentation.

### `@brief`

A short description:

> This function creates a list of ports.

### `@return`

It tells programmers:

> This function returns a `BT::PortsList`.

A **PortsList** is simply a collection of input and output ports.

Think of ports like the plugs on the back of a computer.

```
               LogPrinter

         +----------------------+
         |                      |
Input ---> log_text             |
Input ---> log_level            |
         |                      |
         +----------------------+
```

The Behavior Tree engine needs to know what plugs (ports) this node has.

---

## Next line

```cpp
static BT::PortsList providedPorts()
```

Let's break it apart.

### `static`

This is important.

Normally, functions belong to an object.

For example:

```cpp
LogPrinter printer(...);
printer.tick();
```

But a **static** function belongs to the **class itself**, not a specific object.

That means the Behavior Tree engine can ask:

> "Before I even create a `LogPrinter`, what ports does it require?"

without creating the object.

So it can simply call:

```cpp
LogPrinter::providedPorts();
```

Notice there is no object.

---

### `BT::PortsList`

This is the return type.

It means

> "I'm going to return a list of ports."

---

### `providedPorts()`

This is a special function name recognized by BehaviorTree.CPP.

When the library registers your node, it automatically calls this function.

Imagine the library asking:

```
Behavior Tree Engine

        │
        ▼
"What inputs do you need?"
        │
        ▼
LogPrinter::providedPorts()
        │
        ▼
Returns:
log_text
log_level
```

Now the engine knows how to connect values from the XML.

---

## Opening brace

```cpp
{
```

Beginning of the function.

---

## Return statement

```cpp
return
{
```

We're returning a collection.

Think of it as returning a shopping list.

```
Shopping List

Milk
Bread
Eggs
```

Instead we're returning a list of ports.

---

## First port

```cpp
BT::InputPort<std::string>(
    "log_text",
    "Text to be logged"
),
```

Let's examine this.

### `BT::InputPort`

This creates an **input port**.

An input port receives information from outside the node.

Think of it like a mailbox.

```
Outside World
      │
      ▼
+---------------+
|   log_text    |
+---------------+
      │
      ▼
 LogPrinter
```

---

### `<std::string>`

This tells the compiler the type.

```
InputPort<std::string>
```

means

> This port accepts text.

Examples:

```
"Robot Started"

"Goal Reached"

"Battery Low"

"Navigation Failed"
```

---

### `"log_text"`

This is the name of the port.

The XML must use this exact name.

Example:

```xml
<LogPrinter
    log_text="Robot started"
    log_level="INFO"/>
```

BehaviorTree.CPP matches:

```
XML
──────────────
log_text

↓

Input Port
──────────────
log_text
```

If the names don't match,

the value won't be passed to the node.

---

### `"Text to be logged"`

This is just a description.

It helps documentation tools and developers understand the purpose of the port.

It does **not** affect execution.

---

## Second port

```cpp
BT::InputPort<std::string>(
    "log_level",
    "Log level (INFO, WARN, ERROR)"
),
```

Exactly the same idea.

Create another input port.

Its name is

```
log_level
```

Its type is

```
std::string
```

Possible values are

```
INFO
WARN
ERROR
```

Again, the second string is just a helpful description.

---

## Closing braces

```cpp
};
```

This completes the list of ports being returned.

---

## Final brace

```cpp
}
```

End of the function.

---

# How this connects to the constructor

Earlier, the constructor contained these lines:

```cpp
getInput("log_text", log_text_);
getInput("log_level", log_level_);
```

Where do those values come from?

From the ports declared here.

The flow looks like this:

```
Behavior Tree XML

<LogPrinter
    log_text="Starting navigation"
    log_level="INFO"/>

          │
          ▼

providedPorts()

Declares:

log_text
log_level

          │
          ▼

BehaviorTree.CPP

Creates those ports

          │
          ▼

Constructor

getInput("log_text", log_text_);
getInput("log_level", log_level_);

          │
          ▼

Variables now contain

log_text_ = "Starting navigation"

log_level_ = "INFO"
```

---

# Real-world analogy

Imagine you're filling out a shipping form.

```
Shipping Form

Name:
Address:
Phone:
```

Those fields define what information is expected.

Later, someone fills in:

```
Name: Alice
Address: 123 Main St.
Phone: 555-1234
```

The shipping company reads those values into its system.

`providedPorts()` works the same way:

1. It defines the **fields** (ports) the node accepts.
2. The XML provides the **values**.
3. `getInput()` reads those values into the node's member variables.

---

## Overall picture

```
          LogPrinter Class
         ┌──────────────────────────┐
         │ providedPorts()          │
         │                          │
         │ log_text   (string)      │
         │ log_level  (string)      │
         └─────────────┬────────────┘
                       │
                       ▼
             Behavior Tree Engine
                       │
                       ▼
           Reads Behavior Tree XML
                       │
                       ▼
<LogPrinter
    log_text="Goal reached"
    log_level="INFO"/>
                       │
                       ▼
          getInput() copies values
                       │
                       ▼
log_text_ = "Goal reached"
log_level_ = "INFO"
```

So, `providedPorts()` is essentially the **interface** of your Behavior Tree node. It advertises, "These are the inputs I expect." Later, when the node is constructed and executed, `getInput()` retrieves the values supplied for those ports from the Behavior Tree XML.
Excellent! This is the **heart of the Behavior Tree node**. Everything you've studied so far (constructor, ports, Blackboard) was preparing for this function.

The **`tick()`** function is what the Behavior Tree calls every time it wants this node to do its job.

Let's examine it line by line.

---

```cpp
private:
```

## `private:`

Earlier we saw:

```cpp
public:
```

Now we have:

```cpp
private:
```

This means:

> "Everything below can only be used inside the `LogPrinter` class."

Outside code cannot directly call or modify these members.

---

```cpp
/**
 * @brief Function to perform some user-defined operation on tick
 */
```

This is documentation.

It tells programmers:

> This function contains the work the node performs whenever the Behavior Tree "ticks" it.

---

## The function

```cpp
BT::NodeStatus tick() override
```

This is probably the single most important function in a BehaviorTree.CPP node.

Let's break it apart.

---

### `BT::NodeStatus`

Every Behavior Tree node must return a status.

There are three possible values:

```text
SUCCESS
FAILURE
RUNNING
```

Think of them like traffic lights.

```
SUCCESS  ✅ Finished successfully

FAILURE  ❌ Could not complete

RUNNING  ⏳ Still working
```

Since `LogPrinter` only prints one message, it finishes immediately.

So it only returns

```
SUCCESS
```

or

```
FAILURE
```

---

### `tick()`

Every time the Behavior Tree reaches this node, it calls

```cpp
tick();
```

You can think of it like this:

```
Behavior Tree

Sequence
   |
   +---- LogPrinter
```

When execution reaches `LogPrinter`:

```
Behavior Tree
      │
      ▼
Call tick()
      │
      ▼
Print message
      │
      ▼
Return SUCCESS
```

---

### `override`

This keyword tells C++:

> "I'm replacing the `tick()` function defined in the parent class."

Remember:

```
BT::AsyncActionNode
        ▲
        │
   LogPrinter
```

The parent class already declares a virtual `tick()`.

Your class supplies its own implementation.

The compiler checks that you're actually overriding an existing virtual function. If you accidentally misspell the function name or use the wrong signature, `override` causes a compile-time error instead of silently creating a different function.

---

## Beginning of the function

```cpp
{
```

Everything inside executes whenever the node is ticked.

---

## First decision

```cpp
if (log_level_ == "INFO") {
```

Here we're asking:

> Is the log level equal to `"INFO"`?

Suppose earlier the constructor read:

```xml
<LogPrinter
    log_text="Navigation started"
    log_level="INFO"/>
```

Then

```
log_level_

↓

"INFO"
```

The answer is **Yes**.

---

## Print an INFO message

```cpp
RCLCPP_INFO(
    node_->get_logger(),
    "%s",
    log_text_.c_str());
```

This is a ROS 2 logging macro.

It prints an informational message.

Suppose

```
log_text_

↓

"Navigation started"
```

The console prints

```
[INFO]
Navigation started
```

---

### Why `%s`?

This comes from the C `printf()` style formatting.

`%s` means:

> Insert a string here.

---

### `log_text_.c_str()`

`log_text_` is a C++ `std::string`.

The logging macro expects a C-style string (`const char*`).

So we convert it.

```
std::string

↓

.c_str()

↓

const char*
```

For example,

```
std::string

↓

"Robot Ready"

↓

.c_str()

↓

Pointer to

R o b o t   R e a d y \0
```

The macro then prints that text.

---

## Return

```cpp
return BT::NodeStatus::SUCCESS;
```

This tells the Behavior Tree:

> I completed my job successfully.

Execution continues to the next node.

---

# Second case

```cpp
else if (log_level_ == "WARN")
```

Suppose

```
log_level_

↓

"WARN"
```

Now this branch runs.

---

```cpp
RCLCPP_WARN(...)
```

Console output:

```
[WARN]
Battery Low
```

Then

```cpp
return SUCCESS;
```

Again, the node finished correctly.

---

# Third case

```cpp
else if (log_level_ == "ERROR")
```

Suppose

```
log_level_

↓

"ERROR"
```

Now we print

```cpp
RCLCPP_ERROR(...)
```

Console output:

```
[ERROR]
Motor Failure
```

Again,

```
SUCCESS
```

is returned because the **job of this node is to log a message**, and it did that successfully.

Notice something important:

The log message being an "ERROR" **does not mean the Behavior Tree node failed**.

It simply means:

> "Print this message with ERROR severity."

The node itself still completed its assigned task.

---

# Final case

```cpp
else
```

What if someone writes:

```xml
<LogPrinter
    log_text="Hello"
    log_level="PURPLE"/>
```

The code checks:

```
INFO?  No

WARN?  No

ERROR? No
```

Nothing matched.

So execution reaches

```cpp
else
```

---

## Print error

```cpp
RCLCPP_ERROR(
    node_->get_logger(),
    "Invalid log level: %s",
    log_level_.c_str());
```

Console:

```
[ERROR]
Invalid log level: PURPLE
```

Notice we're **not** printing the user's message.

We're printing an error about the invalid configuration.

---

## Return failure

```cpp
return BT::NodeStatus::FAILURE;
```

This tells the Behavior Tree:

> Something went wrong.

The node could not perform its intended job because it received an unsupported log level.

---

# Complete execution flow

Imagine the XML contains:

```xml
<LogPrinter
    log_text="Goal reached!"
    log_level="WARN"/>
```

The Behavior Tree executes:

```
Behavior Tree
      │
      ▼
tick()
      │
      ▼
Is INFO?
      │
      └──No
      │
      ▼
Is WARN?
      │
      └──Yes
      │
      ▼
Print:

[WARN]
Goal reached!
      │
      ▼
Return SUCCESS
```

---

# What if the XML contains an invalid value?

```xml
<LogPrinter
    log_text="Hello"
    log_level="BLUE"/>
```

Execution becomes

```
tick()
   │
   ▼
INFO?
  No
   │
WARN?
  No
   │
ERROR?
  No
   │
   ▼
Print

Invalid log level: BLUE
   │
   ▼
Return FAILURE
```

---

# Overall picture of the entire class

```
Behavior Tree XML

<LogPrinter
    log_text="Navigation Started"
    log_level="INFO"/>

            │
            ▼
Constructor

Gets:
• ROS node
• log_text
• log_level

            │
            ▼
Behavior Tree later calls

tick()

            │
            ▼
Checks log_level

INFO?
WARN?
ERROR?

            │
            ▼
Prints correct ROS log

            │
            ▼
Returns

SUCCESS
or
FAILURE
```

### In plain English

This `tick()` function is the **worker** of the `LogPrinter` node. Every time the Behavior Tree executes it, the function:

1. Looks at the requested log level (`INFO`, `WARN`, or `ERROR`).
2. Prints the stored message (`log_text_`) using the matching ROS 2 logging function.
3. Returns `SUCCESS` if the log level is valid and the message was logged.
4. Returns `FAILURE` if the log level is invalid, allowing the Behavior Tree to react to the configuration error.
Great! This is the **end of the class** and the **registration** of the node with BehaviorTree.CPP. Without the last few lines, your `LogPrinter` node would exist in C++, but the Behavior Tree engine would **never know it exists**.

Let's go through it line by line.

---

```cpp
std::string log_text_;
```

## `std::string log_text_;`

This declares a **member variable** (also called a data member).

Think of a member variable as the class's own storage.

Earlier, in the constructor, we had:

```cpp
getInput("log_text", log_text_);
```

That line copied the value from the XML into this variable.

For example:

```xml
<LogPrinter
    log_text="Robot started"
    log_level="INFO"/>
```

After the constructor:

```text
log_text_
↓

"Robot started"
```

Later, `tick()` uses it:

```cpp
RCLCPP_INFO(..., log_text_.c_str());
```

So the flow is:

```text
XML

"log_text"

      │
      ▼

Constructor

getInput()

      │
      ▼

log_text_

      │
      ▼

tick()

Print it
```

---

## Next

```cpp
std::string log_level_;
```

Another member variable.

It stores the logging level.

Again,

```cpp
getInput("log_level", log_level_);
```

fills it.

For example:

```text
log_level_

↓

"INFO"
```

Later,

```cpp
if (log_level_ == "INFO")
```

uses this stored value.

So these two variables remember the information the constructor loaded.

---

## Next

```cpp
rclcpp::Node::SharedPtr node_;
```

This stores a pointer to the ROS 2 node.

Earlier we saw:

```cpp
node_ =
config().blackboard->get<rclcpp::Node::SharedPtr>("node");
```

That copied the shared ROS node from the Blackboard.

Now this member variable points to it.

Think of it like this:

```text
LogPrinter

+-------------------------+
| node_ --------------------+
| log_text_              |  |
| log_level_             |  |
+-------------------------+  |
                              |
                              ▼
                     ROS 2 Node
                 +----------------+
                 | Logger         |
                 | Parameters     |
                 | Topics         |
                 +----------------+
```

Whenever the class wants to log something,

it uses

```cpp
node_->get_logger()
```

Without `node_`, the node couldn't call

```cpp
RCLCPP_INFO(...)
```

or

```cpp
RCLCPP_WARN(...)
```

---

## Closing brace

```cpp
};
```

This ends the class definition.

Everything between

```cpp
class LogPrinter
{
```

and

```cpp
};
```

belongs to the class.

---

## Closing namespace

```cpp
}  // namespace goal_navigation_tutorial
```

This ends the namespace.

Remember the beginning:

```cpp
namespace goal_navigation_tutorial
{
```

Now we're leaving that namespace.

Outside of it,

the class's full name is

```cpp
goal_navigation_tutorial::LogPrinter
```

---

# Now comes something very important

```cpp
#include "behaviortree_cpp_v3/bt_factory.h"
```

This includes the Behavior Tree Factory.

What is a factory?

A **factory** is simply an object that knows how to create other objects.

Think of a car factory.

```text
Toyota Factory

Request:
"Build Corolla"

↓

Factory creates Corolla
```

BehaviorTree.CPP works the same way.

```text
Behavior Tree Factory

Request:
"Build LogPrinter"

↓

Factory creates LogPrinter object
```

---

# Registration macro

```cpp
BT_REGISTER_NODES(factory)
```

This is a macro supplied by BehaviorTree.CPP.

When your plugin is loaded,

this macro automatically runs.

You can imagine it expanding conceptually into something like:

```text
When loaded,

register all custom nodes.
```

---

## Opening brace

```cpp
{
```

Inside here,

you tell the factory which nodes exist.

---

## Registration

```cpp
factory.registerNodeType<
goal_navigation_tutorial::LogPrinter
>("LogPrinter");
```

This is probably the most important line after `tick()`.

Let's break it apart.

---

### `factory`

This is the Behavior Tree Factory.

It manages all available node types.

---

### `registerNodeType`

This means:

> Add a new node type to the factory.

---

### `<goal_navigation_tutorial::LogPrinter>`

This tells the compiler:

> The C++ class is

```cpp
goal_navigation_tutorial::LogPrinter
```

---

### `"LogPrinter"`

This is the XML name.

Now the Behavior Tree engine knows:

```text
XML

<LogPrinter>

↓

means

↓

goal_navigation_tutorial::LogPrinter
```

Without this registration,

the XML parser would say:

> Unknown node type: LogPrinter

because it has never heard of your class.

---

# Entire process

Suppose your XML contains

```xml
<BehaviorTree>

    <Sequence>

        <LogPrinter
            log_text="Robot Ready"
            log_level="INFO"/>

    </Sequence>

</BehaviorTree>
```

Here's what happens:

```text
Behavior Tree starts

        │
        ▼

Loads plugin

        │
        ▼

BT_REGISTER_NODES()

        │
        ▼

Factory now knows

"LogPrinter"

↓

goal_navigation_tutorial::LogPrinter

        │
        ▼

Reads XML

<LogPrinter>

        │
        ▼

Factory creates

LogPrinter object

        │
        ▼

Constructor runs

        │
        ▼

tick()

        │
        ▼

Console prints

[INFO]

Robot Ready
```

---

# Think of the factory like a phone book

Imagine a phone book.

```text
Name

↓

Phone Number

John

↓

555-1234
```

The Behavior Tree Factory stores

```text
XML Name

↓

C++ Class

"LogPrinter"

↓

goal_navigation_tutorial::LogPrinter
```

Whenever the XML asks for

```xml
<LogPrinter>
```

the factory immediately knows which C++ class to instantiate.

---

# Summary of the entire file

By the time you've reached the end of this tutorial, you've built a complete custom Behavior Tree node:

```text
1. Namespace
      │
      ▼
Organizes the code

2. Class
      │
      ▼
Creates LogPrinter

3. Constructor
      │
      ▼
Gets ROS node from Blackboard
Reads XML inputs

4. providedPorts()
      │
      ▼
Declares:

• log_text
• log_level

5. tick()
      │
      ▼
Prints INFO/WARN/ERROR messages

6. Member variables
      │
      ▼
Store:

• log_text_
• log_level_
• node_

7. Register node
      │
      ▼
Behavior Tree Factory learns:

XML:

<LogPrinter>

↓

C++:

goal_navigation_tutorial::LogPrinter
```

This is the complete lifecycle of a custom BehaviorTree.CPP node: **declare what inputs it accepts (`providedPorts()`), initialize it (constructor), perform its work (`tick()`), store any needed state (member variables), and finally register it so the Behavior Tree engine can create it from an XML file.**
## Compile and Install Day_Log 07/29/2026:
# Day_Log 07/31/2026:
Certainly. This is the build configuration that tells ROS 2 and CMake what libraries your Behavior Tree plugin needs before it can be compiled.

Let's go through it line by line.

---

### `find_package(ament_cmake REQUIRED)`

```cmake
find_package(ament_cmake REQUIRED)
```

### What it does

This tells CMake:

> "Find the ROS 2 build system called **ament_cmake**."

Think of **ament_cmake** as the **project manager** for your ROS 2 package.

Without it:

* ROS 2 doesn't know how to build your package.
* `colcon build` would fail.

### `REQUIRED`

This means:

> "If you can't find this package, stop immediately."

So if `ament_cmake` isn't installed, you'll see an error like:

```
Could not find package ament_cmake
```

---

## `find_package(rclcpp REQUIRED)`

```cmake
find_package(rclcpp REQUIRED)
```

This loads the C++ ROS 2 client library.

Your plugin uses things like:

```cpp
rclcpp::Node
```

and

```cpp
RCLCPP_INFO(...)
```

Those all belong to **rclcpp**.

Without this package, the compiler wouldn't know what

```cpp
rclcpp::Node
```

means.

Think of it as:

> "I need all of ROS 2's C++ features."

---

## `find_package(rclcpp_components REQUIRED)`

```cmake
find_package(rclcpp_components REQUIRED)
```

This finds the ROS 2 Components library.

ROS Components let multiple ROS nodes run inside one process.

Many Nav2 plugins use Components.

Even if this tutorial doesn't use every feature directly, Nav2 plugins commonly depend on it.

Think of it as loading the toolbox for modular ROS nodes.

---

## `find_package(behaviortree_cpp_v3 REQUIRED)`

```cmake
find_package(behaviortree_cpp_v3 REQUIRED)
```

This loads the entire BehaviorTree.CPP library.

Without it, these lines would fail:

```cpp
BT::NodeStatus
```

```cpp
BT::SyncActionNode
```

```cpp
BT_REGISTER_NODES(factory)
```

All of those come from the BehaviorTree.CPP library.

Think of it as:

> "Load the Behavior Tree engine."

---

# Next section

```cmake
set(dependencies
     rclcpp
     rclcpp_lifecycle
     behaviortree_cpp_v3
)
```

This creates a variable named

```
dependencies
```

Inside it are three libraries.

```
dependencies
│
├── rclcpp
├── rclcpp_lifecycle
└── behaviortree_cpp_v3
```

Later, instead of writing

```cmake
ament_target_dependencies(
    my_plugin
    rclcpp
    rclcpp_lifecycle
    behaviortree_cpp_v3
)
```

they can simply write

```cmake
ament_target_dependencies(
    my_plugin
    ${dependencies}
)
```

`${dependencies}` means:

> "Replace this with everything stored in the variable named `dependencies`."

This makes the `CMakeLists.txt` shorter and easier to maintain.

---

## Why `rclcpp_lifecycle`?

```cmake
rclcpp_lifecycle
```

Nav2 uses **Lifecycle Nodes**.

Unlike ordinary nodes, Lifecycle Nodes have states such as:

```
Unconfigured
↓

Inactive
↓

Active
↓

Finalized
```

This lets Nav2 safely start and stop navigation components.

Even if your plugin doesn't directly use lifecycle features, Nav2 often expects them to be available.

---

# Next section

```cmake
include_directories(
   ${rclcpp_INCLUDE_DIRS}
   ${rclcpp_components_INCLUDE_DIRS}
   ${behaviortree_cpp_v3_INCLUDE_DIRS}
)
```

This tells the compiler where to find the header files (`.hpp` and `.h`) for these libraries.

Imagine your code contains:

```cpp
#include "rclcpp/rclcpp.hpp"
```

The compiler asks:

> "Where is `rclcpp.hpp`?"

These directories answer that question.

For example, `${rclcpp_INCLUDE_DIRS}` expands to something like:

```
/opt/ros/humble/include
```

So the compiler knows where to look.

Similarly:

* `${rclcpp_components_INCLUDE_DIRS}` points to the headers for ROS Components.
* `${behaviortree_cpp_v3_INCLUDE_DIRS}` points to the BehaviorTree.CPP headers.

Without these include directories, you would get errors such as:

```
fatal error:
rclcpp/rclcpp.hpp:
No such file or directory
```

or

```
fatal error:
behaviortree_cpp_v3/action_node.h:
No such file or directory
```

---

## Putting it all together

Think of the build process like building a house:

| CMake Command                                 | Real-world analogy                                                                  |
| --------------------------------------------- | ----------------------------------------------------------------------------------- |
| `find_package(...)`                           | Locate the warehouse that has the building materials.                               |
| `set(dependencies ...)`                       | Make a shopping list of the materials you'll need.                                  |
| `include_directories(...)`                    | Give the construction crew a map to where the blueprints (header files) are stored. |
| `ament_target_dependencies(...)` (used later) | Deliver those materials to the specific house (your plugin) being built.            |

So before your `LogPrinter` plugin can compile, CMake must:

1. Find the required ROS 2 and BehaviorTree libraries.
2. Keep a list of those libraries as dependencies.
3. Tell the compiler where their header files are located.
4. Link those libraries into the final plugin so it can run correctly.

This setup is a standard pattern you'll see in many ROS 2 packages, especially Nav2 plugins.
# Day_Log 08/00/2026:






