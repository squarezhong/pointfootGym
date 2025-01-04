# Code Summary

[TOC]

> Some parts directly use code comments in TAs' code.

In summary, this template provides a framework for training and visualizing point-foot legged robots using PPO. 
It also provides sim-to-sim transfer capabilities between Isaac Gym and Mujoco.

## Project Structure

```python
├── LICENSE
├── README.md
├── install.sh
├── legged_gym
│   ├── __init__.py
│   ├── envs
│   │   ├── __init__.py
│   │   ├── base
│   │   │   └── base_config.py
│   │   └── pointfoot
│   │       ├── point_foot.py
│   │       └── pointfoot_rough_config.py
│   ├── scripts
│   │   ├── export_policy_as_onnx.py
│   │   ├── play.py
│   │   └── train.py
│   ├── tests
│   │   └── test_env.py
│   └── utils
│       ├── __init__.py
│       ├── helpers.py
│       ├── logger.py
│       ├── math.py
│       ├── task_registry.py
│       └── terrain.py
├── licenses
│   └── ...
├── pointfootMujoco
│   ├── README.md
│   ├── pointfoot-sdk-lowlevel # Low-level Controller
│   │   └── ...
│   ├── policy
│   │   └── PF_TRON1A
│   │       ├── params.yaml
│   │       └── policy
│   │           └── policy.onnx
│   ├── rl_controller.py
│   ├── robot-joystick
│   ├── robot_description
│   │   └── PF_TRON1A
│   │       ├── meshes # external resources for the robot
│   │       │   ├── abad_L_Link.STL
│   │       │   └── ...
│   │       │   
│   │       └── xml
│   │           └── robot.xml # urdf file
│   └── simulator.py
└── resources # urdf & mesh for different models
│   └── ...
└── setup.py
```


## legged_gym

#### `__init__.py`
Get `ROBOT_TYPE` from command line, if it is "PF", then register task with `PointFoot`, `PointFootRoughCfg()`, `PointFootRoughCfgPPO()`.

### envs/base

#### `base_config.py`

This file contains a class `BaseConfig` that used to initialize all its member classes **recursively**.

- Constructor: 
Initialize all member classes by calling `self.init_member_classes(self)`

- static method `init_member_classes(obj)`:
Recursively initialize all member classes of the given object.
    - iterate through all the attributes of the object.
    - retrieve the attribute object using `getattr()`
    - If the attribute is an instance, **instantiate** it and sets the attribute to the instance instead of the type .Then call `init_member_classes()` on the instance (recursively).


### envs/pointfoot

#### `point_foot.py`

#### `pointfoot_rough_config.py`

This file contains two main configuration classes:
- `PointFootRoughCfg`: Environment parameters
- `PointFootRoughCfgPPO`: Training parameters

##### class `PointFootRoughCfg(BaseConfig)`:

- inner class `env`:
Mainly contains the number of robots, actions, obersevations, etc.

- inner class `terrain`:
Params for generating terrain, including size, numbers, proportions, etc.

- inner class `commands`:
Params for the commands, especially the range (boundries of linear and angular velocities, and heading angle) of the commands.

- inner class `init_state`:
Init state of the robot, including the position, orientation, velocity of the base, and the joint positions.

- inner class `control`:
Set control type to "position". And set the stiffness and damping of the PD drive.

- inner class `asset`:
Set the asset path of the robot model based on the `ROBOT_TYPE`. And set other params related to the robot's physical and operational attributes.

- inner class `domain_rand`:
Rnage for some randomization parameters, including the friction, mass, push, etc.

- **inner class `rewards`**:
Scale coefficients for different rewards.

- inner class `normalization`:
Params for normalization.

- inner class `noise`:
Params for noise.

- inner class `viewer`:
Set the position of viewer.

- inner class `sim`:
Set params for simulation such as time interval `dt` and gravity etc. 
Besides that an inner-inner class `physx` contains the params for PhysX engine.

##### class `PointFootRoughCfgPPO(BaseConfig)`

This class uses `OnPolicyRunner`.

- inner class `policy`:
Set initial noise standard deviation to 1.0, activation function to "ELU" (Exponential Linear Unit).
The dimensionality of **actor** network is $512 \times 256 \times 128$ ,and for **critic** is also $512 \times 256 \times 128$.

- inner class `algorithm`:
Training params, such as clip value loss, entropy coefficient, etc.

- inner class `runner`:
Specify the policy class as "Actor-Critic" and algorithm class as "PPO". Then set some params for logging and loading.


### scripts

#### `export_policy_as_onnx.py`

- method `export_policy_as_onnx(args)`:
First get environment and training configurations from the `task_registry`. Then set the path and instantiate a actor-critic model. Finally, export the model as an ONNX file by calling `torch.onnx.export`.

#### `play.py`
This file constains a function `play(args)` and mainly achieves the following functions:

1. Override some parameters for testing. Then create a task environment and a algorithm runnerwith reduced number of robots and terrains.

2. If set `EXPORT_POLICY` to True, export the policy as a TorchScript module (`export_policy_as_jit(...)`) an ONNX model (`export_policy_as_onnx(...)`).

3. Set some parameters for the logger, start palaying the policy, then log and plot the states.


#### `train.py`

- method `train(args)`:
    - Create a task **environment** and a **training algorithm runner** using parsed arguments.
    - Train and save the model.


### tests

#### `test_env.py`
Set up a test environment with reduced number of parallel environments. Then test the env with zero actions.

### utils

#### `helpers.py`

Some auxiliary functions are defined in this file.

- method `class_to_dict(obj)`:
Serialize an object to a dictionary by iterating through all its attributes and converting them to a dictionary.

- method `update_class_from_dict(obj, dict)`:  
Set the attributes of an object from a dict by iterating through all the key-value pairs of the dict recursively.

- method `set_seed(seed)`:
To ensure reproducibility, set the seed for numpy, torch, and random.

- method `parse_sim_params(args, cfg)`:
Instantiate a `isaacgym.gymapi.SimParams` object and update it from the parsed arguments and the passed configuration.

- method `get_load_path(root, load_run=-1, checkpoint=-1)`:
Get the path to load the model from.
    - If `load_run` is specified, the load path is `root/{load_run}`, otherwise it is the latest run in `root`.
    - If `checkpoint` is specified, the model path is `model_{checkpoint}.pt`, otherwise it is the latest checkpoint in load path.

- method `update_cfg_from_args(env_cfg, cfg_train, args)`:
Update the environment and training configurations from the parsed arguments.

- method `get_args()`:
Parse the command line arguments using `isaacgym.gymutils.parse_arguments()`.

- method `export_policy_as_jit(actor_critic, path)`:
If the actor-critic model has an attribute "memory_a", use `PolicyExporterLSTM` to export the model as a TorchScript module. Otherwise, directly use `torch.jit.script()` to export the actor model.


##### class `PolicyExporterLSTM(torch.nn.Module)`:
This class encapsulates a actor-critic model, extracting its actor network and LSTM-based memory, then exports the model as a TrochScript module.

- Constructor:
    - duplicate the actor network and LSTM memory from the actor-critic model.
    - register `hidden_state` and `cell_state` as persistent buffers.

- method `forward(self, x)`:
    1. process input data through the LSTM.
    2. update the hidden and cell states.
    3. pass the processed data through the actor network.

- method `export(self, path)`:
Move model to CPU and export it as a TorchScript module.

#### `logger.py`

Class `Logger` is designed to log and manage state and reward for RL tasks.

- Constructor:
Store some member variables:
    - `self.state_log`: state data
    - `self.rew_log`: reward data
    - `self.dt`: time step value used for plot function
    - `self.num_episodes`: a counter to keep track of the number of rewards logged
    - `self.plot_process`: a placeholder for a `Process` to plot data in separate process.

- Three log_ method used to log data and a `reset(self)` function

- method `plot_states(self)`:
Start a process of `_plot(self)`

- method ` _plot(self)`:
Use matplotlib to plot a 3 by 3 figure. From left to right, from top to bottom in sequence is：
    1. joint targets and measured positions
    2. joint velocities
    3. base velocity x
    4. base velocity y 
    5. base angular velocity yaw
    6. base velocity z
    7. contact forces
    8. torque/velocity curves
    9. torques

- method `print_rewards(self)`:
Print average rewards per second and total number of episodes.

#### `math.py`

This file contains some math-related functions.

- method `quat_apply_yaw(quat, vec)`:
Copy the quaternion and reshape it to ensure proper dimensionality. Then it zeroes out the x and y (roll and pitch) components of the quaternion. After normalizing the modified quaternion, it applies the ywa only rotation to the input vector.

- method `wrap_to_pi(angles)`:
Wrap angles to [-pi, pi] range.

- method `torch_rand_sqrt_float(lower, upper, shape, device)`:
    - Generates random folat number that are more likely to be close to the **boundries** of the range.
    - First using ` 2*torch.rand(*shape, device=device) - 1` to generate a random tensor in [-1, 1] range. Then apply a square root transformation with sign preservation. The resulting valuea are then be normalized to the range [0, 1] using `(r + 1.) / 2`. Finally the values are scaled to the desired range [lower, upper].


#### `task_registry.py`

Class `TaskRegistry` is used to register and retrieve task-related configurations and classes in a structed way.

- `VecEnv` and `OnPolicyRunner` aren imported from the `rsl_rl` package, which provides the RL env and runners.

- method `register(self, name, task_class, env_cfg, train_cfg)`:
register a new task by storing its class (in this project is `PointFoot`), environment configuration (`PointFootRoughCfg`) and training configuration (`PointFootRoughCfgPPO`).

- method `get_cfgs(self, name)`:
get the environment and training configurations of the task with the given name.

- method `make_env(self, name, args=None, env_cfg=None)`：
creates an environment either from a registered name (get configuration from the class member variables) or from the provided configuration file.

- method `make_alg_runner(self, env, name=None, args=None, train_cfg=None, log_root="default")`:
    - Creates the training algorithm  either from a registered name (get configuration from the class member variables) or from the provided config file.
    - directory for logging is set to `log_root` if provided, otherwise to `default`.


#### `terrain.py`

This file contains a class `terrain` to generate terrain using `PointFootRoughCfg.terrain`.

- method `add_terrain_to_map(self, terrain, row, col)`
add a terrain segment to a larger map at the given row and column.

- method `make_terrain(self, choice, difficulty)`
    - first set params (slope, step_height, discrete_obstacles_height, gap_size...) of the terrain segment based on difficulty.
    - Based on "choice" and "proportions" in config, do a if-elif-else to generate the corresponding type of terrain.

- method `gap_terrain(terrain, gap_size, platform_size=1.)`:
generate a terrain with a platform and gap (very deep hole) around it.

- method `pit_terrain(terrain, depth, platform_size=1.)`:
generate a terrain with a pit in the middle.

- method `randomized_terrain(self)`:
randomly select choice and difficulty to generate a terrain. Repeat it.

- method `curiculum(self)`:
follow a curriculum to select choice and difficulty. Then generate a terrain and repeat the process.

- method `selected_terrain(self)`:
generate selected type of terrain based on `self.cfg.terrain_kwargs`.


## pointfootMujoco

#### `rl_controller.py`

The `PointfootController` class implements a robotic controller for point-foot (PF), wheel-foot (WF) or sole-foot (SF) robot in Mujoco. 
It need to be used along with the `SimulatorMujoco` class.
In this project it utilizes policy trained by Isaac Gym to control the PF robot's motion dynamically.

##### 1. Initialization (`__init__` method):

- Load YAML configuration and ONNX policy model.
- Initialize the robot commands (`RoborCmd`). state (`RobotState`) and IMU data (`ImuData`) with default values.
- Set up callbacks to handle updates to robot state, IMU data, sensor joystick input, and diagostics.

##### 2. Configuration Loading (`load_config` method):

- Read configuaration from YAML file.
- Assign configuration parameters to controller variables.
- Initialize variables for actions, observations, and commands.
- Initialize joint angles based on the initial configuration

##### 3. Main Control Loop (`run` method):

- Iteratively calls the `update` method to execute the control loop.
- Ends safely by reseting robot command values

##### 4. Modes of Operation

- `STAND` mode (`handle_stand_mode` method):  
Smoothly moves the robot to a standing pose using linear interpolation of joint angles.

- `WALK` mode (`handle_walk_mode` method):  
Uses the RL policy to generate actions for the robot.

##### 5. Observations and Actions

- Observations (`compute_observation` method):
Final observations (with `np.clip`):
    - Base angular velocity (scaled)
    - Projected gravity vector
    - Joint positions (difference from initial angles, scaled)
    - Joint velocities (scaled)
    - Last actions applied to the robot
    - Scaled command inputs

- Actions (`compute_action` method):
Computes the actions based on the current observations using the policy session.

##### 6. Callbacks

- `robot_state_callback` method:
Callback function to update the robot state from incoming data.

- `imu_data_callback` method:
Callback function to update IMU data from incoming data.

- `sensor_joy_callback` method:
Callback function for receiving sensor joy data

- `robot_diagnostic_callback` method:
Callback function for receiving diagnostic data

#### `simulator.py`

The `SimulatorMujoco` class simulates and visualizes the robot in Mujoco.

1. **Load a robot model** from an XML file.
2. **Control the robot's joints** using commands reveived during the simulation.
3. **Collect data from sensors** (like IMU, joint positions, velocities, torque etc.)
4. **Visualize the simuation** using Mujoco viewer.


### pointfoot-sdk-lowlevel

Python SDK for the low-level controller of the robot. Three architectures are available: `aarch64`, `amd64`, and `win`.


### policy/PF_TRON1A

#### `policy/policy.onnx`

The trained policy model in ONNX format is stored here.

#### params.yaml

The parameters (init_state, control, normalization ,etc.) used for simulation.


### robot_description/PF_TRON1A

Meshes and URDF files for the robot model, which are used in the Mujoco.


### robot-joystick

Store repo "limxdynamics/robot-joystick" for controlling the robot using a virtual joystick.

## resources

This folder contains the URDF and mesh files for different robot models.

## MISC

#### `install.sh`

A shell script to install the project dependencies.

#### `setup.py`

A script for setting up the `legged_gym` python package.
