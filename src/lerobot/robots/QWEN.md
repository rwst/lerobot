# LeRobot Robots Framework

This document provides an overview of the robot framework implemented in the LeRobot library, which supports various robotic platforms for real-world control and data collection.

## Overview

The LeRobot robot framework provides a standardized interface for controlling different types of robots. It supports multiple robot platforms including Koch, SO-100, SO-101, LeKiwi, Stretch 3, and others. The framework enables teleoperation, data recording, policy evaluation, and robot calibration.

## Core Components

### Robot Base Class

The `Robot` class is an abstract base class that defines the standard interface for all robot implementations:

- `observation_features`: Property describing the structure and types of observations produced by the robot
- `action_features`: Property describing the structure and types of actions expected by the robot
- `is_connected`: Property indicating whether the robot is currently connected
- `connect()`: Establish connection to the robot
- `is_calibrated`: Property indicating whether the robot is calibrated
- `calibrate()`: Calibrate the robot if applicable
- `configure()`: Apply runtime configuration to the robot
- `get_observation()`: Retrieve the current observation from the robot
- `send_action()`: Send an action command to the robot
- `disconnect()`: Disconnect from the robot and perform cleanup

### Robot Configuration

The `RobotConfig` class is an abstract base class for robot configurations using draccus choice registry pattern. It supports:

- Robot identification with unique IDs
- Calibration directory specification
- Camera configurations for visual observations

## Supported Robot Types

### Koch Robots

#### Koch Follower

The Koch follower arm is a 6-DOF robotic arm with Dynamixel servos:

##### Features:
- 6 motors: shoulder_pan, shoulder_lift, elbow_flex, wrist_flex, wrist_roll, gripper
- Support for both Koch v1.0 and v1.1 versions
- Calibration functionality
- Camera support for visual observations

##### Configuration:
```python
KochFollowerConfig(
    port="/dev/ttyUSB0",  # Port to connect to the arm
    disable_torque_on_disconnect=True,
    max_relative_target=5,  # Safety limit for relative target positions
    cameras={"wrist": OpenCVCameraConfig(...)},
    use_degrees=False
)
```

### SO-Series Robots

#### SO-100 Follower

The SO-100 follower arm is a 6-DOF robotic arm with Feetech servos:

##### Features:
- 6 motors: shoulder_pan, shoulder_lift, elbow_flex, wrist_flex, wrist_roll, gripper
- Calibration functionality
- Camera support for visual observations
- Position control with PID tuning

##### Configuration:
```python
SO100FollowerConfig(
    port="/dev/ttyUSB0",  # Port to connect to the arm
    disable_torque_on_disconnect=True,
    max_relative_target=5,  # Safety limit for relative target positions
    cameras={"wrist": OpenCVCameraConfig(...)},
    use_degrees=False
)
```

#### SO-101 Follower

The SO-101 follower arm is a 6-DOF robotic arm with Feetech servos:

##### Features:
- 6 motors: shoulder_pan, shoulder_lift, elbow_flex, wrist_flex, wrist_roll, gripper
- Calibration functionality
- Camera support for visual observations
- Position control with PID tuning

##### Configuration:
```python
SO101FollowerConfig(
    port="/dev/ttyUSB0",  # Port to connect to the arm
    disable_torque_on_disconnect=True,
    max_relative_target=5,  # Safety limit for relative target positions
    cameras={"wrist": OpenCVCameraConfig(...)},
    use_degrees=False
)
```

### Mobile Robots

#### LeKiwi

The LeKiwi mobile robot combines a three omniwheel mobile base with a remote follower arm:

##### Features:
- 6 arm motors: arm_shoulder_pan, arm_shoulder_lift, arm_elbow_flex, arm_wrist_flex, arm_wrist_roll, arm_gripper
- 3 base motors: base_left_wheel, base_back_wheel, base_right_wheel
- Base velocity control for omnidirectional movement
- Keyboard teleoperation support
- Camera support for visual observations

##### Configuration:
```python
LeKiwiConfig(
    port="/dev/ttyUSB0",  # Port to connect to the bus
    disable_torque_on_disconnect=True,
    max_relative_target=5,  # Safety limit for relative target positions
    cameras={"front": OpenCVCameraConfig(...), "wrist": OpenCVCameraConfig(...)},
    use_degrees=False
)
```

##### LeKiwi Client
Remote client for controlling the LeKiwi robot over network:

```python
LeKiwiClientConfig(
    remote_ip="192.168.1.100",  # IP address of the robot
    port_zmq_cmd=5555,
    port_zmq_observations=5556,
    teleop_keys={"forward": "w", "backward": "s", ...},
    cameras={"front": OpenCVCameraConfig(...), "wrist": OpenCVCameraConfig(...)},
    polling_timeout_ms=15,
    connect_timeout_s=5
)
```

### Bimanual Robots

#### BiSO100Follower

A bimanual robot with two SO-100 follower arms:

##### Features:
- Left and right SO-100 arms
- Independent control of both arms
- Camera support for visual observations
- Prefix-based action/observation handling

##### Configuration:
```python
BiSO100FollowerConfig(
    left_arm_port="/dev/ttyUSB0",
    right_arm_port="/dev/ttyUSB1",
    left_arm_disable_torque_on_disconnect=True,
    right_arm_disable_torque_on_disconnect=True,
    left_arm_max_relative_target=5,
    right_arm_max_relative_target=5,
    left_arm_use_degrees=False,
    right_arm_use_degrees=False,
    cameras={"front": OpenCVCameraConfig(...)}
)
```

### Manipulation Robots

#### HopeJR

The HopeJR robot includes both arm and hand components:

##### HopeJR Hand
5-finger dexterous hand with 16 motors:

```python
HopeJrHandConfig(
    port="/dev/ttyUSB0",
    side="right",  # or "left"
    disable_torque_on_disconnect=True,
    cameras={"wrist": OpenCVCameraConfig(...)}
)
```

##### HopeJR Arm
7-DOF arm with 7 motors:

```python
HopeJrArmConfig(
    port="/dev/ttyUSB0",
    disable_torque_on_disconnect=True,
    max_relative_target=5,
    cameras={"wrist": OpenCVCameraConfig(...)}
)
```

### Research Robots

#### Stretch 3

The Stretch 3 mobile manipulator from Hello Robot:

##### Features:
- Mobile base with differential drive
- Telescopic arm with 4 DOF
- Head with pan/tilt camera
- Multiple camera support (navigation, head, wrist)
- Mock mode for simulation

##### Configuration:
```python
Stretch3RobotConfig(
    max_relative_target=5,
    cameras={
        "navigation": OpenCVCameraConfig(...),
        "head": RealSenseCameraConfig(...),
        "wrist": RealSenseCameraConfig(...)
    },
    mock=False
)
```

## Robot Control Features

### Calibration

All robots support calibration procedures to ensure consistent positioning:

```python
# Manual calibration
robot.calibrate()

# Automatic calibration during connection
robot.connect(calibrate=True)
```

### Safety Limits

Robots can enforce safety limits on relative target positions:

```python
# Limit relative movement to 5 units
config = RobotConfig(max_relative_target=5)
```

### Camera Integration

Robots support multiple camera types for visual observations:

```python
cameras = {
    "wrist": OpenCVCameraConfig(
        index_or_path=0,
        fps=30,
        width=640,
        height=480
    ),
    "front": RealSenseCameraConfig(
        serial_number_or_name="0123456789",
        fps=30,
        width=1280,
        height=720
    )
}
```

### Asynchronous Communication

Robots support asynchronous observation reading for improved performance:

```python
# Non-blocking observation capture
observation = robot.get_observation()
```

## Configuration Management

### Draccus Integration

The framework uses draccus for command-line argument parsing and configuration management:

```bash
# Configure robot from command line
python script.py --robot.type=koch_follower --robot.port=/dev/ttyUSB0
```

### Choice Registry Pattern

Robot types are registered using the draccus choice registry pattern:

```python
@RobotConfig.register_subclass("koch_follower")
@dataclass
class KochFollowerConfig(RobotConfig):
    # implementation
    pass
```

## Robot Utilities

### Robot Factory

The robot factory provides utilities for creating robots:

```python
from lerobot.robots.utils import make_robot_from_config

# Create robot from configuration
robot = make_robot_from_config(config)
```

### Safety Functions

Utilities for ensuring safe robot operation:

```python
from lerobot.robots.utils import ensure_safe_goal_position

# Cap relative action target magnitude for safety
safe_positions = ensure_safe_goal_position(goal_present_pos, max_relative_target)
```

## Control Modes

### Teleoperation

Direct control of robots using input devices:

```bash
python lerobot/scripts/control_robot.py \
    --robot.type=koch_follower \
    --control.type=teleoperate
```

### Data Recording

Recording robot demonstrations for training:

```bash
python lerobot/scripts/control_robot.py \
    --robot.type=koch_follower \
    --control.type=record \
    --control.repo_id=user/dataset_name \
    --control.num_episodes=10
```

### Policy Evaluation

Running trained policies on real robots:

```bash
python lerobot/scripts/control_robot.py \
    --robot.type=koch_follower \
    --control.type=record \
    --control.policy.path=path/to/policy
```

### Calibration

Running robot calibration procedures:

```bash
python lerobot/scripts/control_robot.py \
    --robot.type=koch_follower \
    --control.type=calibrate
```

## Error Handling

The framework includes comprehensive error handling:

- `DeviceAlreadyConnectedError`: Raised when trying to connect an already connected robot
- `DeviceNotConnectedError`: Raised when operations are attempted on a disconnected robot
- `ConnectionError`: Raised when robot connection fails
- Runtime warnings for calibration issues

## Best Practices

### 1. Robot Connection Management

Always properly manage robot connections:

```python
try:
    robot.connect()
    # ... use robot ...
finally:
    robot.disconnect()
```

### 2. Calibration Usage

Ensure proper calibration before operation:

```python
if not robot.is_calibrated:
    robot.calibrate()
```

### 3. Safety Limits

Implement safety limits for robot movements:

```python
# Limit relative movement for safety
config = RobotConfig(max_relative_target=5)
```

### 4. Error Handling

Implement robust error handling:

```python
try:
    observation = robot.get_observation()
except DeviceNotConnectedError:
    # Handle disconnected robot
    pass
except RuntimeError as e:
    # Handle other errors
    pass
```

## Extending the Framework

### Adding New Robot Types

1. Create a new config class inheriting from `RobotConfig`
2. Register it with the choice registry
3. Implement the required abstract methods

```python
@RobotConfig.register_subclass("new_robot")
@dataclass
class NewRobotConfig(RobotConfig):
    # implementation
    pass

class NewRobot(Robot):
    config_class = NewRobotConfig
    name = "new_robot"
    
    # implementation of abstract methods
    pass
```

### Custom Configuration Fields

Add new fields to robot configurations:

```python
@dataclass
class CustomRobotConfig(RobotConfig):
    new_feature: bool = False
    advanced_param: float = 1.0
```

## Requirements

To use the robot framework, you'll need:

- Appropriate robot SDKs (dynamixel_sdk, scservo_sdk)
- Camera drivers (OpenCV, RealSense)
- Communication libraries (ZeroMQ for LeKiwi)
- Motor control libraries
- Appropriate hardware drivers

This robot framework provides a flexible and extensible foundation for controlling various robotic platforms in the LeRobot library, enabling real-world robot learning research and deployment.