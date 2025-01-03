# Code Summary

[TOC]

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

This file contains a class `BaseConfig` that used to initialize all its member classes recursively.

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

TODO

### scripts

#### `export_policy_as_onnx.py`

TODO

#### `play.py`
This file constains a function `play(args)` and mainly achieves the following functions:

1. 

2. 

3. TODO


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

- method `set_seed(seed)`:

- method `parse_sim_params(args, cfg)`:

- method `get_load_path(root, load_run=-1, checkpoint=-1)`:

- method `update_cfg_from_args(env_cfg, cfg_train, args)`:

- method `get_args()`:

- method `export_policy_as_jit(actor_critic, path)`:

- class `PolicyExporterLSTM(torch.nn.Module)`:
TODO

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

#### `simulator.py`


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

TODO

## MISC

#### `install.sh`

A shell script to install the project dependencies.

#### `setup.py`

A script for setting up the `legged_gym` python package.
