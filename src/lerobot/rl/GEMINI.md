# GEMINI.md: Python Scripts Overview

This document provides a brief overview of the Python scripts in the `lerobot/scripts/rl/` directory. These scripts are components of a distributed reinforcement learning (RL) framework, likely for robot manipulation tasks.

## `actor.py`

This script runs the **actor** process in a distributed RL setup. The actor's primary role is to interact with the robot environment. It loads the current policy from the learner, executes actions, collects experience (observations, actions, rewards), and sends this data back to the learner for training. It is designed to handle human-in-the-loop (HIL) interventions, allowing a human to guide the robot during training.

## `crop_dataset_roi.py`

This is a utility script for **dataset preprocessing**. It provides an interactive tool for a user to select a rectangular Region of Interest (ROI) from a sample image in a `LeRobotDataset`. The script then processes the entire dataset, cropping and resizing all images according to the selected ROI, and saves it as a new dataset. This is crucial for focusing the policy's attention on the relevant parts of the visual input.

## `eval_policy.py`

This script is used to **evaluate a trained policy**. It loads a policy from a saved checkpoint and runs it in the robot environment for a specified number of episodes. It then calculates and logs performance metrics, such as the total reward or success rate, to assess how well the policy performs its task.

## `gym_manipulator.py`

This file defines a custom `gymnasium.Env` for robot manipulation, providing a standardized interface for RL agents. It includes a factory function, `make_robot_env`, which constructs the environment and applies a stack of wrappers to add functionality. These wrappers handle tasks like:
- Preprocessing observations (e.g., cropping/resizing images, adding joint velocities).
- Handling different modes of human intervention (gamepad, keyboard, or a leader-follower robot setup).
- Converting data formats to be compatible with LeRobot policies.

## `learner.py`

This script is the central **learner** in the distributed RL architecture. It is responsible for training the policy. The learner runs a gRPC server to receive experience data from actors. It manages one or more replay buffers (for both online and offline data), samples batches of data, performs the optimization steps to update the policy's neural networks, and logs training metrics. Periodically, it sends the updated policy parameters back to the actors.

## `learner_service.py`

This script implements the **gRPC service** for the learner. It defines the server-side logic for the communication protocol between the learner and actors. It handles RPCs for streaming policy parameters to actors and for receiving transitions and other interaction metadata from them. It acts as the network interface for the `learner.py` script.
