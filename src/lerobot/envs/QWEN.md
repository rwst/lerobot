# LeRobot Environments

This document provides an overview of the environment framework implemented in the LeRobot library, which supports various robotic environments for training and evaluation.

## Overview

The LeRobot environment framework provides a standardized interface for interacting with different robotic environments. It supports multiple environment types including Aloha, PushT, XArm, and Human-In-the-Loop (HIL) environments.

## Core Components

### Environment Configuration

The `EnvConfig` class is an abstract base class that defines the standard interface for environment configurations using the draccus choice registry pattern:

- `task`: The task name for the environment
- `fps`: Frames per second for the environment
- `features`: Dictionary defining the features of the environment
- `features_map`: Mapping of feature names to standardized keys
- `gym_kwargs`: Abstract property for Gym environment keyword arguments

### Supported Environment Types

#### Aloha Environment

The Aloha environment is configured through the `AlohaEnv` class:

##### Features:
- Task: Typically "AlohaInsertion-v0"
- FPS: 50 Hz
- Episode length: 400 steps
- Observation types: "pixels" or "pixels_agent_pos"
- Render mode: "rgb_array"

##### Configuration:
```python
AlohaEnv(
    task="AlohaInsertion-v0",
    fps=50,
    episode_length=400,
    obs_type="pixels_agent_pos",
    render_mode="rgb_array"
)
```

##### Features:
- Action: 14-dimensional continuous actions
- Agent position: 14-dimensional state observations
- Top camera: 480x640x3 RGB images

#### PushT Environment

The PushT environment is configured through the `PushtEnv` class:

##### Features:
- Task: "PushT-v0"
- FPS: 10 Hz
- Episode length: 300 steps
- Observation types: "pixels_agent_pos" or "environment_state_agent_pos"
- Render mode: "rgb_array"
- Visualization resolution: 384x384

##### Configuration:
```python
PushtEnv(
    task="PushT-v0",
    fps=10,
    episode_length=300,
    obs_type="pixels_agent_pos",
    render_mode="rgb_array",
    visualization_width=384,
    visualization_height=384
)
```

##### Features:
- Action: 2-dimensional continuous actions
- Agent position: 2-dimensional state observations
- Pixels: 384x384x3 RGB images (for pixels_agent_pos)
- Environment state: 16-dimensional state observations (for environment_state_agent_pos)

#### XArm Environment

The XArm environment is configured through the `XarmEnv` class:

##### Features:
- Task: "XarmLift-v0"
- FPS: 15 Hz
- Episode length: 200 steps
- Observation types: "pixels_agent_pos"
- Render mode: "rgb_array"
- Visualization resolution: 384x384

##### Configuration:
```python
XarmEnv(
    task="XarmLift-v0",
    fps=15,
    episode_length=200,
    obs_type="pixels_agent_pos",
    render_mode="rgb_array",
    visualization_width=384,
    visualization_height=384
)
```

##### Features:
- Action: 4-dimensional continuous actions
- Agent position: 4-dimensional state observations
- Pixels: 84x84x3 RGB images

#### Human-In-the-Loop (HIL) Environment

The HIL environment is configured through the `HILEnvConfig` class:

##### Features:
- Task: "PandaPickCubeKeyboard-v0"
- FPS: 100 Hz
- Episode length: 100 steps
- Supports viewer and gamepad control
- Video recording capabilities

##### Configuration:
```python
HILEnvConfig(
    name="PandaPickCube",
    task="PandaPickCubeKeyboard-v0",
    use_viewer=True,
    use_gamepad=True,
    fps=100,
    episode_length=100
)
```

##### Features:
- Action: 4-dimensional continuous actions
- Observation image: 3x128x128 RGB images
- Observation state: 18-dimensional state observations

## Environment Factory

The environment factory provides utilities for creating environments:

### Creating Environment Configurations

```python
from lerobot.envs.factory import make_env_config

# Create Aloha environment configuration
env_config = make_env_config("aloha", task="AlohaInsertion-v0")

# Create PushT environment configuration
env_config = make_env_config("pusht", task="PushT-v0")
```

### Creating Environment Instances

```python
from lerobot.envs.factory import make_env

# Create a single environment
env = make_env(env_config, n_envs=1)

# Create multiple parallel environments
env = make_env(env_config, n_envs=4, use_async_envs=True)
```

## Environment Utilities

### Observation Preprocessing

The framework provides utilities for preprocessing environment observations:

```python
from lerobot.envs.utils import preprocess_observation

# Convert Gym observations to LeRobot format
lerobot_obs = preprocess_observation(gym_obs)
```

### Feature Mapping

Utilities for mapping environment features to policy features:

```python
from lerobot.envs.utils import env_to_policy_features

# Map environment features to policy features
policy_features = env_to_policy_features(env_config)
```

## Key Features

### Standardized Interface

All environments follow a standardized interface that makes it easy to switch between different environments:

```python
# Same interface for all environments
env.reset()
obs, reward, terminated, truncated, info = env.step(action)
env.close()
```

### Vectorized Environments

Support for both synchronous and asynchronous vectorized environments:

```python
# Synchronous vectorized environments
env = gym.vector.SyncVectorEnv([lambda: gym.make("aloha/AlohaInsertion-v0") for _ in range(4)])

# Asynchronous vectorized environments
env = gym.vector.AsyncVectorEnv([lambda: gym.make("aloha/AlohaInsertion-v0") for _ in range(4)])
```

### Observation Processing

Automatic processing of observations to match policy requirements:

- Image format conversion (channel last to channel first)
- Data type conversion (uint8 to float32)
- Normalization (0-255 to 0-1 range)
- Batch dimension handling

### Feature Standardization

Mapping of environment-specific feature names to standardized keys:

- `ACTION`: Standard action key
- `OBS_STATE`: Standard state observation key
- `OBS_IMAGE`: Standard image observation key
- `OBS_IMAGES`: Standard multi-camera image observation key
- `OBS_ENV_STATE`: Standard environment state key

## Configuration Management

### draccus Integration

Environment configurations use draccus for command-line argument parsing:

```bash
# Configure environment from command line
python train.py --env.type=aloha --env.task=AlohaInsertion-v0 --env.episode_length=500
```

### Choice Registry Pattern

Environment types are registered using the draccus choice registry pattern:

```python
@EnvConfig.register_subclass("aloha")
@dataclass
class AlohaEnv(EnvConfig):
    # implementation
    pass
```

## Error Handling

The framework includes comprehensive error handling:

- `ModuleNotFoundError`: Raised when environment packages are not installed
- `ValueError`: Raised for invalid configurations
- Runtime warnings for missing environment attributes

## Best Practices

### 1. Environment Installation

Install the required environment packages:

```bash
# Install Aloha environment
pip install 'lerobot[aloha]'

# Install PushT environment
pip install 'lerobot[pusht]'

# Install XArm environment
pip install 'lerobot[xarm]'
```

### 2. Proper Environment Management

Always properly close environments:

```python
try:
    env = make_env(env_config)
    # ... use environment ...
finally:
    env.close()
```

### 3. Feature Consistency

Ensure feature definitions match between environments and policies:

```python
# Check that environment features match policy requirements
env_features = env_to_policy_features(env_config)
# ... validate against policy features ...
```

### 4. Observation Processing

Use the provided utilities for observation preprocessing:

```python
# Standard observation preprocessing
processed_obs = preprocess_observation(raw_obs)
```

## Extending the Framework

### Adding New Environment Types

1. Create a new config class inheriting from `EnvConfig`
2. Register it with the choice registry
3. Implement the required abstract methods

```python
@EnvConfig.register_subclass("my_env")
@dataclass
class MyEnvConfig(EnvConfig):
    # implementation
    pass
```

### Custom Feature Definitions

Define custom features for new environments:

```python
@dataclass
class MyEnvConfig(EnvConfig):
    features: dict[str, PolicyFeature] = field(
        default_factory=lambda: {
            "action": PolicyFeature(type=FeatureType.ACTION, shape=(6,)),
            "observation": PolicyFeature(type=FeatureType.STATE, shape=(12,)),
        }
    )
```

## Requirements

To use the environment framework, you'll need:

- gymnasium for environment interfaces
- Environment-specific packages (gym-aloha, gym-pusht, etc.)
- Appropriate dependencies for each environment type
- einops for tensor operations
- numpy and torch for numerical computations

This environment framework provides a flexible and extensible foundation for working with various robotic environments in the LeRobot library.