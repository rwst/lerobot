# LeRobot Configuration System

This document provides an overview of the configuration system used in the LeRobot library, which leverages the draccus library for managing experiment configurations.

## Overview

The LeRobot configuration system provides a powerful and flexible way to manage experiment configurations using dataclasses and the draccus library. It supports hierarchical configurations, command-line overrides, and integration with Hugging Face Hub for model sharing.

## Core Components

### Configuration Classes

The system is organized around several core configuration classes:

1. **PreTrainedConfig**: Base class for policy configurations
2. **TrainPipelineConfig**: Configuration for training pipelines
3. **EvalPipelineConfig**: Configuration for evaluation pipelines
4. **DatasetConfig**: Configuration for datasets
5. **WandBConfig**: Configuration for Weights & Biases logging

### Key Features

#### Hierarchical Configuration
Configs can be nested to create complex hierarchical structures:

```python
@dataclass
class TrainPipelineConfig:
    dataset: DatasetConfig
    policy: PreTrainedConfig
    wandb: WandBConfig
```

#### Command-Line Overrides
Using draccus, configurations can be overridden from the command line:

```bash
python train.py --dataset.repo_id=my_dataset --policy.learning_rate=1e-4
```

#### Choice Registry Pattern
The system uses draccus choice registry for polymorphic configurations:

```python
@PreTrainedConfig.register_subclass("my_policy")
@dataclass
class MyPolicyConfig(PreTrainedConfig):
    # policy-specific configuration
    pass
```

Usage from command line:
```bash
python train.py --policy.type=my_policy --policy.some_param=value
```

#### Plugin System
Support for loading external plugins through command-line arguments:

```bash
python train.py --env.discover_packages_path=my_package.envs
```

## Configuration Files

### JSON Serialization
Configurations are serialized to JSON format for easy sharing and versioning:

```json
{
    "type": "my_policy",
    "learning_rate": 1e-4,
    "batch_size": 32
}
```

### Loading from Files
Configs can be loaded from local files or Hugging Face Hub:

```python
config = PreTrainedConfig.from_pretrained("my_org/my_policy")
```

## Policy Configuration

The `PreTrainedConfig` class serves as the base for all policy configurations:

### Key Properties

- **type**: The policy type identifier
- **n_obs_steps**: Number of observation steps
- **input_shapes**: Input data shapes
- **output_shapes**: Output data shapes
- **input_normalization_modes**: Normalization modes for inputs
- **output_normalization_modes**: Normalization modes for outputs
- **device**: Target device (cuda/cpu)
- **use_amp**: Whether to use Automatic Mixed Precision

### Feature Definitions

Policies define their input and output features:

```python
@dataclass
class PolicyFeature:
    type: FeatureType  # STATE, VISUAL, ACTION, etc.
    shape: tuple
```

### Normalization Modes

Supported normalization modes:
- `MIN_MAX`: Normalize to [-1, 1] range
- `MEAN_STD`: Normalize using mean and standard deviation
- `IDENTITY`: No normalization

## Training Pipeline Configuration

The `TrainPipelineConfig` manages training workflows:

### Key Components

- **dataset**: Dataset configuration
- **policy**: Policy configuration
- **optimizer**: Optimizer configuration
- **scheduler**: Learning rate scheduler configuration
- **eval**: Evaluation configuration
- **wandb**: Weights & Biases logging configuration

### Training Presets

Policies can define training presets for optimizer and scheduler:

```python
def get_optimizer_preset(self) -> OptimizerConfig:
    return AdamWConfig(lr=self.optimizer_lr)

def get_scheduler_preset(self) -> LRSchedulerConfig:
    return CosineDecayWithWarmupSchedulerConfig(...)
```

## Evaluation Configuration

The `EvalPipelineConfig` manages evaluation workflows:

### Key Properties

- **n_episodes**: Number of evaluation episodes
- **batch_size**: Batch size for evaluation
- **use_async_envs**: Whether to use asynchronous environments

## Dataset Configuration

The `DatasetConfig` manages dataset loading:

### Key Properties

- **repo_id**: Dataset repository ID
- **root**: Local dataset storage path
- **episodes**: Episode indices to load
- **image_transforms**: Image transformation configuration
- **use_imagenet_stats**: Whether to use ImageNet statistics for normalization

## Configuration Validation

Configs include validation logic in `__post_init__` methods:

```python
def __post_init__(self):
    if self.batch_size > self.n_episodes:
        raise ValueError("Batch size cannot be larger than number of episodes")
```

## Best Practices

### 1. Use Type Hints
Always use type hints for better IDE support and validation:

```python
@dataclass
class MyConfig:
    learning_rate: float = 1e-4
    batch_size: int = 32
```

### 2. Provide Sensible Defaults
Include reasonable defaults for all configurable parameters:

```python
@dataclass
class MyConfig:
    learning_rate: float = 1e-4
    weight_decay: float = 1e-6
    warmup_steps: int = 1000
```

### 3. Document Configuration Options
Add comments explaining the purpose of each configuration option:

```python
@dataclass
class MyConfig:
    # Learning rate for the optimizer
    learning_rate: float = 1e-4
    # Weight decay coefficient
    weight_decay: float = 1e-6
```

### 4. Use Validation
Implement validation logic to catch configuration errors early:

```python
def __post_init__(self):
    if self.learning_rate <= 0:
        raise ValueError("Learning rate must be positive")
```

## Command-Line Usage

### Basic Usage
```bash
python train.py --dataset.repo_id=my_dataset --policy.type=my_policy
```

### Nested Configuration
```bash
python train.py --policy.optimizer.lr=1e-4 --policy.scheduler.warmup_steps=1000
```

### Loading from File
```bash
python train.py --config_path=configs/my_experiment.json
```

### Loading Policy from Hub
```bash
python train.py --policy.path=my_org/my_policy --dataset.repo_id=my_dataset
```

## Integration with Hugging Face Hub

### Saving Configurations
Configs can be saved and uploaded to the Hugging Face Hub:

```python
config = MyPolicyConfig()
config.push_to_hub("my_org/my_policy")
```

### Loading Configurations
Configs can be loaded from the Hugging Face Hub:

```python
config = MyPolicyConfig.from_pretrained("my_org/my_policy")
```

## Extending the Configuration System

### Adding New Policy Types
1. Create a new config class inheriting from `PreTrainedConfig`
2. Register it with the choice registry
3. Implement required abstract methods

```python
@PreTrainedConfig.register_subclass("new_policy")
@dataclass
class NewPolicyConfig(PreTrainedConfig):
    # implementation
    pass
```

### Adding New Configuration Fields
Simply add new fields to the dataclass:

```python
@dataclass
class MyConfig(PreTrainedConfig):
    new_feature: bool = False
    advanced_param: float = 1.0
```

## Common Patterns

### Conditional Configuration
Use `__post_init__` for conditional validation:

```python
def __post_init__(self):
    if self.use_advanced_feature and self.advanced_param is None:
        raise ValueError("advanced_param required when use_advanced_feature=True")
```

### Computed Properties
Use `@property` for computed configuration values:

```python
@property
def total_training_steps(self) -> int:
    return self.epochs * self.steps_per_epoch
```

### Environment-Specific Configuration
Use environment variables for environment-specific settings:

```python
def __post_init__(self):
    if self.device is None:
        self.device = os.getenv("DEVICE", "cuda")
```

## Troubleshooting

### Configuration Conflicts
When both path and type arguments are provided:

```bash
# This will raise an error
python train.py --policy.path=my_policy --policy.type=some_type
```

### Missing Required Fields
Ensure all required fields without defaults are provided:

```python
@dataclass
class MyConfig:
    # This field must be provided
    required_field: str
    # This field has a default
    optional_field: str = "default"
```

### Validation Errors
Validation errors will be raised during initialization:

```bash
ValueError: Batch size cannot be larger than number of episodes
```

This configuration system provides a robust foundation for managing experiment configurations in LeRobot, enabling reproducible research and easy sharing of models and experimental setups.