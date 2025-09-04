# 🤗 LeRobot Context for Qwen Code

This document provides an overview of the LeRobot codebase to help Qwen Code understand its structure and purpose for future interactions.

## Project Overview

LeRobot is a PyTorch library for state-of-the-art machine learning for real-world robotics. Its main goals are:

1.  Lower the barrier to entry for robotics research and development.
2.  Facilitate sharing of datasets and pretrained models within the community.
3.  Provide tools for training, evaluating, and deploying robot policies, with a focus on imitation learning and reinforcement learning.
4.  Support both simulation environments and real-world robot hardware.

The library provides:
*   **Policies**: Implementations of various robot control policies (e.g., ACT, Diffusion, TDMPC, VQ-BeT, PI0).
*   **Datasets**: Tools for loading, creating, and managing robotic datasets (`LeRobotDataset`), including support for videos and efficient data streaming.
*   **Environments**: Gymnasium-compatible simulation environments (Aloha, PushT, XArm) for training and evaluation.
*   **Robots**: Interfaces and control logic for various real-world robots (e.g., Koch, SO-100, SO-101, HopeJR, LeKiwi).
*   **Motors & Cameras**: Standardized interfaces for interacting with common motor buses (Dynamixel, Feetech) and cameras (OpenCV, Intel RealSense).
*   **Training & Evaluation Scripts**: Command-line tools (`lerobot-train`, `lerobot-eval`) for running experiments.
*   **Teleoperation & Data Collection**: Scripts (`lerobot-teleoperate`, `lerobot-record`) for controlling robots and collecting demonstration data.

## Core Technologies

*   **Language**: Python 3.10+
*   **Core Frameworks**: PyTorch, Gymnasium
*   **Data Management**: Hugging Face Datasets, Hugging Face Hub
*   **Configuration**: Draccus (dataclasses-based CLI)
*   **Video Processing**: FFmpeg, TorchCodec/PyAV
*   **Code Quality**: Ruff (linting/formatting), Pre-commit hooks

## Key Modules

*   `lerobot.scripts`: Main entry points for training, evaluation, visualization, and robot control.
*   `lerobot.policies`: Implementations of various robot control policies.
*   `lerobot.datasets`: `LeRobotDataset` class and utilities for dataset handling.
*   `lerobot.envs`: Simulation environments and their configurations.
*   `lerobot.robots`: Interfaces and configurations for real-world robots.
*   `lerobot.motors`: Abstractions for motor communication buses.
*   `lerobot.cameras`: Abstractions for camera interfaces.
*   `lerobot.model`: (Currently limited) Kinematics utilities (e.g., using Placo).
*   `lerobot.optim`: Optimizer and learning rate scheduler configurations.
*   `lerobot.configs`: Dataclass definitions for configuring training, evaluation, policies, datasets, etc.
*   `lerobot.processor`: Data transformation pipelines for observations, actions, etc.
*   `lerobot.utils`: General-purpose utilities.

## Building, Running, and Testing

### Installation

1.  **Setup Environment**:
    *   Create a virtual environment (e.g., with conda): `conda create -y -n lerobot python=3.10 && conda activate lerobot`
    *   Install FFmpeg (required for video processing): `conda install ffmpeg -c conda-forge`
2.  **Install LeRobot**:
    *   **From Source (Development)**: Clone the repo, navigate to it, and run `pip install -e .`. Add extras like `[aloha,pusht]` for simulations or `[dynamixel,feetech]` for motors.
    *   **From PyPI (Stable)**: Run `pip install lerobot`. Add extras like `[all]` for full functionality.

### Key Commands

*   **Training a Policy**:
    *   `lerobot-train --config_path path/to/config_or_policy` (e.g., `lerobot-train --config_path lerobot/diffusion_pusht`)
    *   Leverages `src/lerobot/scripts/train.py`.
*   **Evaluating a Policy**:
    *   `lerobot-eval --policy.path path/to/policy --env.type env_type` (e.g., `lerobot-eval --policy.path lerobot/diffusion_pusht --env.type pusht`)
    *   Leverages `src/lerobot/scripts/eval.py`.
*   **Visualizing a Dataset**:
    *   `python -m lerobot.scripts.visualize_dataset --repo-id repo_id` (e.g., `python -m lerobot.scripts.visualize_dataset --repo-id lerobot/pusht`)
*   **Teleoperating a Robot**:
    *   `lerobot-teleoperate --robot.type robot_type --robot.port port` (e.g., `lerobot-teleoperate --robot.type koch_follower --robot.port /dev/ttyUSB0`)
*   **Recording Data with a Robot**:
    *   `lerobot-record --robot.type robot_type --robot.port port --dataset.repo_id user/dataset_name`
*   **Calibrating a Robot**:
    *   `lerobot-calibrate --robot.type robot_type --robot.port port --robot.id robot_id`

### Testing

*   Tests are located in the `tests/` directory.
*   The test suite uses `pytest`.
*   Run the full suite with `python -m pytest -sv ./tests`.
*   Run specific tests with `python -m pytest tests/<TEST_FILE>.py`.

## Development Conventions

*   **Configuration**: Heavy use of dataclasses with `draccus` for defining and parsing configuration (e.g., `TrainPipelineConfig`, `EnvConfig`, `PreTrainedConfig`). This allows for hierarchical configurations and command-line overrides.
*   **Policies**: Policies inherit from `PreTrainedPolicy` and have associated configuration dataclasses (e.g., `ACTConfig`). They are designed to be loaded from and saved to the Hugging Face Hub.
*   **Datasets**: The core is `LeRobotDataset`, which handles loading data from the Hugging Face Hub or local storage, supports video encoding/decoding, and provides features like delta-timestamped frame access.
*   **Robots**: Robots inherit from the `Robot` base class and have configuration dataclasses (e.g., `KochFollowerConfig`). They standardize interaction via `connect`, `calibrate`, `get_observation`, `send_action`, and `disconnect`.
*   **Modularity**: The codebase is structured into distinct modules (`policies`, `datasets`, `envs`, `robots`, etc.) to promote modularity and reusability.
*   **Code Style**: Code formatting and linting are enforced using `ruff`. Pre-commit hooks are used to automate checks.
*   **Contribution**: Contributions are welcome via pull requests, following the guidelines in `CONTRIBUTING.md`. This includes forking, branching, making changes, running tests, and submitting PRs.
