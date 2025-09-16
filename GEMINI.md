# LeRobot: A Comprehensive Overview

This document provides a high-level overview of the LeRobot codebase, a PyTorch library for state-of-the-art machine learning for real-world robotics. It outlines the project's structure, core components, and key functionalities.

## Project Goals

LeRobot aims to:
1.  Lower the barrier to entry for robotics research and development.
2.  Facilitate the sharing of datasets and pretrained models via the Hugging Face Hub.
3.  Provide robust tools for training, evaluating, and deploying robot policies.
4.  Support both simulation environments and a wide range of real-world robot hardware.

## Core Technologies

- **Language**: Python 3.10+
- **Core Frameworks**: PyTorch, Gymnasium
- **Data Management**: Hugging Face Datasets, Hugging Face Hub
- **Configuration**: Draccus (dataclasses-based CLI)
- **Video Processing**: FFmpeg, TorchCodec/PyAV
- **Code Quality**: Ruff (linting/formatting), Pre-commit hooks

## Key Modules

### Configuration (`lerobot.configs`)
The entire library is driven by a powerful configuration system built on `draccus`. It uses Python dataclasses to define hierarchical and type-safe configurations for all components, from policies and datasets to robots and training pipelines. This allows for easy modification and overrides via the command line.

### Datasets (`lerobot.datasets`)
The `LeRobotDataset` class is the cornerstone for data handling. It supports loading datasets from the Hugging Face Hub or local storage, efficient video decoding, chunked storage for large datasets, and comprehensive metadata management. The `MultiLeRobotDataset` allows for training on several datasets simultaneously.

### Environments (`lerobot.envs`)
LeRobot provides standardized `gymnasium.Env` interfaces for various simulation environments, including **Aloha**, **PushT**, and **XArm**. A factory utility makes it easy to create and configure single or vectorized environments for training and evaluation.

### Policies (`lerobot.policies`)
The library implements a variety of state-of-the-art robot control policies, such as:
- **ACT** (Action Chunking Transformers)
- **Diffusion Policy**
- **PI0** and **PI0+FAST** (Vision-Language-Action models)
- **SAC** (Soft Actor-Critic)
- **SmolVLA** (Vision-Language-Action Model)
- **TDMPC** (Temporal Difference Learning for Model Predictive Control)
- **VQBeT** (Behavior Generation with Latent Actions)

Policies are designed to be easily shared and loaded from the Hugging Face Hub.

### Data Processing (`lerobot.processor`)
The processor framework provides a flexible pipeline for data transformation. It uses a series of composable steps to process environment transitions, handling tasks like observation normalization, action scaling, and tensor device placement. This ensures that data is in the correct format for policy input and output.

### Optimizers & Schedulers (`lerobot.optim`)
This module provides a configurable system for managing PyTorch optimizers (e.g., Adam, AdamW) and learning rate schedulers. Like other components, these are configured via dataclasses, allowing for easy experimentation with different optimization strategies.

### Hardware Interfaces

#### Robots (`lerobot.robots`)
A standardized `Robot` base class provides a consistent interface for interacting with real-world hardware. Implementations exist for various robots, including **Koch**, **SO-100**, **SO-101**, **LeKiwi**, and **Stretch 3**. The framework supports teleoperation, data recording, and policy evaluation on physical systems.

#### Motors (`lerobot.motors`)
A hardware abstraction layer provides a unified API for controlling different motor buses, primarily **Dynamixel** and **Feetech**. It handles low-level communication, calibration, and synchronized read/write operations.

#### Cameras (`lerobot.cameras`)
A standardized camera interface supports various backends, including **OpenCV**-compatible webcams and **Intel RealSense** cameras. It provides a simple API for synchronous and asynchronous frame capture, including color and depth streams.

### Kinematics (`lerobot.model`)
The kinematics module, powered by the `placo` library, provides tools for forward and inverse kinematics. Using a robot's URDF file, it can compute end-effector poses from joint angles and vice-versa, enabling more advanced, pose-based control.

### Scripts (`lerobot.scripts`)
LeRobot includes a suite of command-line scripts that serve as the primary entry points for common workflows:
- **`lerobot-train`**: Trains a policy on a dataset.
- **`lerobot-eval`**: Evaluates a trained policy in a simulation environment.
- **`lerobot-record`**: Records demonstration data from a real robot.
- **`lerobot-teleoperate`**: Controls a robot using a gamepad or leader-follower setup.
- **`lerobot-calibrate`**: Calibrates robot motors.
- **Data Visualization**: Scripts to visualize datasets in 3D with Rerun or in a web-based HTML interface.

### Distributed Reinforcement Learning (`lerobot.scripts.rl`)
For advanced use cases, LeRobot provides a distributed RL framework.
- **`actor.py`**: Runs on the robot to collect experience and send it to the learner.
- **`learner.py`**: The central training process that receives data, updates the policy, and sends new parameters back to the actors.
- **`gym_manipulator.py`**: A custom `gymnasium.Env` that integrates real robots and human-in-the-loop (HIL) control into the RL loop.
