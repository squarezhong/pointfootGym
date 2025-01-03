# Code Summary

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


### scripts

#### `export_policy_as_onnx.py`

#### `play.py`

#### `train.py`


### tests

#### `test_env.py`


### utils

#### `helpers.py`

#### `logger.py`

#### `math.py`

#### `task_registry.py`

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


## MISC

#### `install.sh`
A shell script to install the project dependencies.

#### `setup.py`
A script for setting up the `legged_gym` python package.