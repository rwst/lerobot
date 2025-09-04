# LeRobot Scripts

This document provides an overview of the scripts available in the LeRobot library, which are designed to facilitate various aspects of robotic learning, including training, evaluation, data visualization, and system information.

## Overview

The scripts in the LeRobot library are command-line tools that leverage the draccus library for configuration management. They cover a wide range of functionalities, from training policies on datasets to evaluating trained models on real robots, and from visualizing dataset content to checking system compatibility.

## Core Scripts

### Training

#### `train.py`

The main training script for LeRobot. It supports training policies on datasets using various optimization techniques.

**Key Features:**
- Training pipeline configuration via `TrainPipelineConfig`
- Support for multiple policy types (ACT, Diffusion, etc.)
- Integration with Weights & Biases for experiment tracking
- Checkpointing and resuming capabilities
- Evaluation during training with simulated environments

**Usage Example:**
```bash
python -m lerobot.scripts.train \
    --dataset.repo_id=lerobot/pusht \
    --policy.type=act \
    --policy.name=act_pusht \
    --training.offline_steps=100000 \
    --training.eval_freq=10000 \
    --training.save_freq=10000 \
    --training.save_checkpoint=true
```

### Evaluation

#### `eval.py`

Evaluates a trained policy on an environment by running rollouts and computing metrics.

**Key Features:**
- Batched policy rollout with multiple environments
- Video rendering of episodes
- Success rate and reward computation
- Support for custom environments

**Usage Example:**
```bash
lerobot-eval \
    --policy.path=lerobot/diffusion_pusht \
    --env.type=pusht \
    --eval.batch_size=10 \
    --eval.n_episodes=10 \
    --use_amp=false \
    --device=cuda
```

### Data Visualization

#### `visualize_dataset.py`

Visualizes data from all frames of any episode of a LeRobotDataset using Rerun.

**Key Features:**
- Local and remote visualization modes
- Support for saving .rrd files for later viewing
- Real-time data streaming capabilities
- Visualization of camera images, actions, states, and rewards

**Usage Example:**
```bash
python -m lerobot.scripts.visualize_dataset \
    --repo-id lerobot/pusht \
    --episode-index 0
```

#### `visualize_dataset_html.py`

Generates an HTML interface for visualizing dataset episodes with interactive plots.

**Key Features:**
- Web-based interface for dataset exploration
- Interactive Dygraphs for time-series data visualization
- Video playback for camera observations
- Episode selection and navigation

**Usage Example:**
```bash
python -m lerobot.scripts.visualize_dataset_html \
    --repo-id lerobot/pusht \
    --episodes 0 1 2
```

#### `visualize_image_transforms.py`

Visualizes the effects of image transforms on dataset frames.

**Key Features:**
- Visualization of individual transforms
- Combined transform examples
- Configuration-based transform application

**Usage Example:**
```bash
python -m lerobot.scripts.visualize_image_transforms \
    --repo-id=lerobot/pusht \
    --episodes='[0]' \
    --image_transforms.enable=True
```

### System Information

#### `display_sys_info.py`

Displays system configuration information for troubleshooting and environment setup.

**Key Features:**
- LeRobot version information
- Platform and Python version details
- Hugging Face Hub and Datasets versions
- NumPy and PyTorch version information
- CUDA availability and version

**Usage:**
```bash
python -m lerobot.scripts.display_sys_info
```

## Robot Control Scripts

### Teleoperation and Data Collection

#### `control_robot.py`

A comprehensive script for controlling robots, supporting teleoperation, data recording, policy evaluation, and calibration.

**Key Features:**
- Multiple control modes: teleoperate, record, replay, calibrate, record(eval)
- Support for various robot types (Koch, SO100, etc.)
- Integration with gamepad and leader-follower teleoperation
- Dataset creation and uploading to Hugging Face Hub
- Real-time data visualization

**Usage Examples:**

**Teleoperation:**
```bash
python lerobot/scripts/control_robot.py \
    --robot.type=koch_follower \
    --robot.port=/dev/tty.usbmodem575E0031751 \
    --control.type=teleoperate
```

**Recording a dataset:**
```bash
python lerobot/scripts/control_robot.py \
  --robot.type=koch_follower \
  --robot.port=/dev/tty.usbmodem575E0031751 \
  --control.type=record \
  --control.single_task="Pick up the lego block and place it in the bowl." \
  --dataset.repo_id=$HF_USER/koch_test \
  --dataset.num_episodes=50
```

**Replaying an episode:**
```bash
python lerobot/scripts/control_robot.py \
  --robot.type=koch_follower \
  --robot.port=/dev/tty.usbmodem575E0031751 \
  --control.type=replay \
  --dataset.repo_id=$HF_USER/koch_test \
  --dataset.episode=0
```

### Calibration

#### `calibrate.py`

Calibrates robots to ensure consistent positioning between different robot instances.

**Key Features:**
- Automatic calibration procedure for supported robots
- Manual calibration for custom robots
- Calibration data saving and loading
- Safety checks during calibration

**Usage Example:**
```bash
lerobot-calibrate \
    --robot.type=koch_follower \
    --robot.port=/dev/tty.usbmodem58760431551 \
    --robot.id=my_awesome_follower_arm
```

### Motor Setup

#### `setup_motors.py`

Sets up motors for robots, including ID assignment and baudrate configuration.

**Key Features:**
- Automatic motor ID assignment
- Baudrate configuration for communication
- Support for multiple motor types (Dynamixel, Feetech)
- Safety checks during setup

**Usage Example:**
```bash
lerobot-setup-motors \
    --robot.type=koch_follower \
    --robot.port=/dev/tty.usbmodem575E0031751
```

### Port Discovery

#### `find_port.py`

Finds USB ports associated with robot motor controllers.

**Key Features:**
- Automatic port detection for motor buses
- Support for multiple robot types
- Cross-platform compatibility

**Usage:**
```bash
lerobot-find-port
```

## RL Training Scripts

### Distributed Training

#### `rl/actor.py`

Actor server for distributed HILSerl robot policy training. Executes policies in robot environments and sends transitions to learner servers.

**Key Features:**
- Distributed training architecture
- Human-in-the-loop intervention support
- gRPC communication with learner servers
- Policy parameter synchronization
- Transition streaming to learners

#### `rl/learner.py`

Learner server for distributed HILSerl robot policy training. Initializes policies, maintains replay buffers, and updates policies based on transitions.

**Key Features:**
- SAC policy training with distributed actors
- Replay buffer management
- Policy checkpointing and resuming
- Weights & Biases integration
- gRPC server for actor communication

#### `rl/gym_manipulator.py`

Robot environment for LeRobot manipulation tasks with human intervention support.

**Key Features:**
- Gym-compatible robot environments
- Multiple robot type support (SO100, SO101, Koch)
- Human intervention via leader-follower control or gamepad
- End-effector and joint space control
- Image processing (cropping and resizing)

### Dataset Processing

#### `rl/crop_dataset_roi.py`

Crops regions of interest from LeRobot datasets for focused training.

**Key Features:**
- Interactive ROI selection for images
- Batch processing of dataset episodes
- Cropping and resizing of observations
- Dataset recreation with transformed data

#### `rl/eval_policy.py`

Evaluates trained policies on real robots.

**Key Features:**
- Policy evaluation on physical robots
- Success rate computation
- Episode-based evaluation metrics

## Server Scripts

### Policy Server

#### `server/policy_server.py`

Serves trained policies over gRPC for remote inference.

**Key Features:**
- gRPC-based policy serving
- Observation preprocessing and action postprocessing
- Concurrent client handling
- FPS tracking and latency monitoring

### Robot Client

#### `server/robot_client.py`

Client for connecting robots to remote policy servers.

**Key Features:**
- gRPC communication with policy servers
- Observation streaming and action execution
- Action queue management
- Latency optimization and bandwidth control

## Configuration Management

All scripts use draccus for configuration management, allowing for:

1. **Command-line Arguments**: All configuration options can be set via command-line arguments
2. **Configuration Files**: Support for JSON and YAML configuration files
3. **Hierarchical Configuration**: Nested configuration structures with inheritance
4. **Type Safety**: Strong typing with automatic validation
5. **Documentation**: Built-in help generation from docstrings

## Common Patterns

### Error Handling

Scripts include comprehensive error handling:

- Device connection and disconnection management
- Configuration validation
- Runtime error reporting
- Graceful shutdown procedures

### Logging

All scripts use standardized logging:

- Colored console output for better readability
- File-based logging for debugging
- Structured log messages with timestamps
- Verbosity control via command-line arguments

### Concurrency

Many scripts support concurrent execution:

- Threading for I/O-bound operations
- Multiprocessing for CPU-intensive tasks
- Async I/O for network operations
- Proper resource management and cleanup

## Best Practices

### When Using Scripts

1. **Always check system requirements** before running scripts
2. **Use virtual environments** to avoid dependency conflicts
3. **Enable logging** for troubleshooting
4. **Validate configurations** before starting long-running processes
5. **Monitor resource usage** during training and evaluation

### When Developing Scripts

1. **Follow the existing patterns** for consistency
2. **Use draccus for configuration** management
3. **Implement proper error handling** and recovery
4. **Include comprehensive logging** for debugging
5. **Write unit tests** for core functionality
6. **Document all public APIs** and command-line options

## Extending the Framework

### Adding New Scripts

To add a new script:

1. Create a new Python file in `src/lerobot/scripts/`
2. Use the `@parser.wrap()` decorator for draccus integration
3. Implement the main functionality in a function decorated with `@parser.wrap()`
4. Include comprehensive help text in docstrings
5. Add example usage in comments

### Custom Configuration

To add custom configuration options:

1. Extend the appropriate configuration dataclass
2. Use draccus field annotations for help text and validation
3. Implement validation logic in `__post_init__` methods
4. Update dependent scripts to handle new options

This script framework provides a solid foundation for robotic learning experiments while maintaining flexibility for custom extensions and research directions.