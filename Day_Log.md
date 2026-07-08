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
## Day_Log Thursday July 9 2026:
