# Trajectory Generation and Publishing 
MATLAB scripts to help generate lawnmower trajectories and ROS2 package to publish the trajectories.

## Trajectory Publisher - Getting Started
The driver utilizes [NumPy](https://numpy.org/) and [progressbar2](https://progressbar-2.readthedocs.io/en/latest/), which both may be installed using [pip](https://pypi.org/project/pip/):
```
pip install numpy
pip install progressbar2
```

In addition, the driver is dependent on custom interfaces found in the package **bluerov_interfaces** and **brov2_interfaces**. *Two interface packages are used, since trajectory interface must be the same as utilized by the controller.*

Build and source the **brov2_trajectory** package and all it's dependencies:
```
colcon build --packages-up-to brov2_trajectory
source install/setup.bash
```
Run the `trajectory_publisher` node with default parameters:
```
ros2 run brov2_trajectory brov2_trajectory_exe
```

## Trajectory Publisher - Node Parameters
Configuring the settings can be done through making changes to the [node parameter file](params/params.yaml) and passing this as an argument when running the `trajectory_publisher` node:
```
ros2 run brov2_trajectory brov2_trajectory_exe --ros-args --params-file src/brov2_trajectory/params/params.yaml
```
* **trajectory_topic_name**: The name of the topic where pressure sensor readings are published; default="barometer_data"
* **trajectory_file_name**: The namce of the csv-file containing the trajectory to be published; default="MC_circle.csv"
* **trajectory_period**: The period in (seconds) between publishing set-point from trajectory; default=0.01
* **update_printout**: Bool allowing printout of current position and yaw set-point; default=True


## Trajectory Publisher - Interface Description
A trajectory consists of and is published as a consecutive row of separate set-points. Each set-point is described using the custom *Reference.msg* interface from **bluerov_interface** consisting of the following variables:
| Variable      | Description |
| -----------   | ----------- |
| pos           | The reference position [x,y,z] in the semi-global frame defined upon initialization of QEKF   |
| quat          | The reference orientation in quaternions [w,x,y,z]                                            |
| velocity      | The reference velocity (linear and angular)                                                   |
| acceleration  | The reference acceleration (linear and angular)                                               |  


## Trajectory Generation - Getting Started
The trajectory generating scripts are written using the [Sensor Fusion and Tracking Toolbox](https://se.mathworks.com/help/fusion/index.html?s_tid=CRUX_lftnav) in MATLAB. The main scripts are
* `trajectory_constraint_generator.m`: Script generating constraints for a specified trajectory
* `trajectory_generator.m`: Script creating and storing trajectory based on the stored constraints

and they are accompanied by several scripts containing support functions in order to generating trajectories easier. Two types of lawnmower trajectories are supported, namely *out-n-back* and *side-to-side*.  

*INSERT EXAMPLE IMAGES OF THE TWO TRAJECTORIES*

A trajectory is generated to adhere to multiple constraints on [Positions, timeOfArrival, Velocities, Orientation], and the toolbox will provide what it deems as the most efficient trajectory. To generate a trajectory the following steps must be executed:
1. Set the constraint variables
2. *Generate* constraints
2. Set the trajectory variables
4. *Create* and *store* the trajectory

## Trajectory Generation - Step 1: Set the Constraint Variables
The first step is to set the variables which help define constraints for a specific trajectory:

| Variable      | Description |
| -----------   | ----------- |
| surge_velocity            | The velocity when executing the trajectory in (meters/seconds)                |
| turn_radius               | The radius of each separate turn in (meters)                                  |
| line_distance             | The distance in the xy-plane to cover in each straight section in (meters)    |  
| number_of_lines           | The number of straight sections in the lawnmower path                         |
| turn_velocity_percentage  | Percentage of surge_velocity defining the velocity                            |
| start_point               | The starting point of the trajectory [x,y,z]                                  |
| start_angle               | Angle (degrees) with which to reorient path constraints                       |
| start_time                | Amount of time the vehicle can use to get to the starting point               |
| end_point                 | The ending point of the trajectory [x,y,z]                                    |
| path_type                 | The type of trajectory to generate; *out-n-back* or *side-to-side*            |
| turn_direction            | The direction of the first turn of the trajectory; *right* or *left*          |

Proceed to run *Section 1* using the **run section** option, and make sure the variables are defined in the workspace alongside with the *oriented_path_constraints* object.

## Trajectory Generation - Step 2: Generate Constraints
Note how the *oriented_path_constraints* object contrains the trajectory to a depth of **0** (all values along the 3rd column are 0) for all set-points. It is necessary to manually edit all depth values along the 3rd column in the *oriented_path_constraints* object prior to running *Section 2*. 

Run *Section 2* using the **run section** option; this will create a *trajectory_constraints* object containing all necessary constraints [timeOfArrival, Positions, Velocities, Orientation]. 

*Make sure no successive time stamps are equal to avoid error during trajectory generation.*

## Trajectory Generation - Step 3: Set the Trajectory Variables


## Trajectory Generation - Step 4: Create and Store the Trajectory