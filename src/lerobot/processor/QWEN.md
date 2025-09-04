# LeRobot Processor Framework

This document provides an overview of the processor framework implemented in the LeRobot library, which manages data transformation pipelines for robotic control systems.

## Overview

The LeRobot processor framework provides a standardized interface for configuring and managing data transformation pipelines. It supports composable processing steps that can modify observations, actions, rewards, and other transition components in a flexible and extensible manner.

## Core Components

### ProcessorStep Protocol

The `ProcessorStep` protocol defines the structural interface for a single processor step:

#### Required Methods:
- `__call__(transition: EnvTransition) -> EnvTransition`: Process a transition and return a modified transition
- `feature_contract(features: dict[str, PolicyFeature]) -> dict[str, PolicyFeature]`: Transform feature keys to a standardized contract

#### Optional Helper Methods:
- `get_config() -> dict[str, Any]`: Return user-defined JSON-serializable configuration
- `state_dict() -> dict[str, torch.Tensor]`: Return PyTorch tensor state
- `load_state_dict(state)`: Inverse of `state_dict`
- `reset()`: Clear internal buffers at episode boundaries

### RobotProcessor Class

The `RobotProcessor` class orchestrates an ordered collection of processor steps:

#### Key Features:
- Composable processing pipeline with ordered steps
- Support for both `EnvTransition` dicts and batch dictionaries
- Automatic conversion between formats
- Processor-level hooks for observation/monitoring
- Serialization and deserialization capabilities

#### Main Methods:
- `__call__(data)`: Process data through all steps
- `step_through(data)`: Yield intermediate results after each step
- `save_pretrained(save_directory)`: Serialize processor definition and parameters
- `from_pretrained(pretrained_model_name_or_path)`: Load a serialized processor
- `register_*_hook(fn)`: Attach hooks for monitoring
- `reset()`: Clear state in every step

### ProcessorStepRegistry

The `ProcessorStepRegistry` class manages processor step registration:

#### Key Features:
- Registry for processor steps with name-based lookup
- Decorator-based registration
- Support for retrieving registered steps by name
- Listing and clearing registered steps

#### Registration Example:
```python
@ProcessorStepRegistry.register(name="adaptive_normalizer")
class AdaptiveObservationNormalizer:
    # implementation
    pass
```

## Built-in Processor Steps

### ObservationProcessor

Base class for processors that modify only the observation component:

#### Key Features:
- Automatic extraction and reinsertion of observation data
- Focus on observation-specific processing logic
- Boilerplate reduction for transition dict manipulation

#### Example Implementation:
```python
class MyObservationScaler(ObservationProcessor):
    def __init__(self, scale_factor):
        self.scale_factor = scale_factor

    def observation(self, observation):
        return observation * self.scale_factor
```

### ActionProcessor

Base class for processors that modify only the action component:

#### Key Features:
- Automatic extraction and reinsertion of action data
- Focus on action-specific processing logic
- Boilerplate reduction for transition dict manipulation

#### Example Implementation:
```python
class ActionClipping(ActionProcessor):
    def __init__(self, min_val, max_val):
        self.min_val = min_val
        self.max_val = max_val

    def action(self, action):
        return np.clip(action, self.min_val, self.max_val)
```

### RewardProcessor

Base class for processors that modify only the reward component:

#### Key Features:
- Automatic extraction and reinsertion of reward data
- Focus on reward-specific processing logic
- Boilerplate reduction for transition dict manipulation

#### Example Implementation:
```python
class RewardScaler(RewardProcessor):
    def __init__(self, scale_factor):
        self.scale_factor = scale_factor

    def reward(self, reward):
        return reward * self.scale_factor
```

### DoneProcessor

Base class for processors that modify only the done flag:

#### Key Features:
- Automatic extraction and reinsertion of done flag
- Focus on done flag processing logic
- Boilerplate reduction for transition dict manipulation

#### Example Implementation:
```python
class TimeoutDone(DoneProcessor):
    def __init__(self, max_steps):
        self.steps = 0
        self.max_steps = max_steps

    def done(self, done):
        self.steps += 1
        return done or self.steps >= self.max_steps

    def reset(self):
        self.steps = 0
```

### TruncatedProcessor

Base class for processors that modify only the truncated flag:

#### Key Features:
- Automatic extraction and reinsertion of truncated flag
- Focus on truncated flag processing logic
- Boilerplate reduction for transition dict manipulation

#### Example Implementation:
```python
class EarlyTruncation(TruncatedProcessor):
    def __init__(self, threshold):
        self.threshold = threshold

    def truncated(self, truncated):
        return truncated or some_condition > self.threshold
```

### InfoProcessor

Base class for processors that modify only the info dictionary:

#### Key Features:
- Automatic extraction and reinsertion of info dictionary
- Focus on info dictionary processing logic
- Boilerplate reduction for transition dict manipulation

#### Example Implementation:
```python
class InfoAugmenter(InfoProcessor):
    def __init__(self):
        self.step_count = 0

    def info(self, info):
        info = info.copy()
        info["steps"] = self.step_count
        self.step_count += 1
        return info

    def reset(self):
        self.step_count = 0
```

### ComplementaryDataProcessor

Base class for processors that modify only the complementary data:

#### Key Features:
- Automatic extraction and reinsertion of complementary data
- Focus on complementary data processing logic
- Boilerplate reduction for transition dict manipulation

### IdentityProcessor

Identity processor that does nothing:

#### Key Features:
- Pass-through processing without modification
- Useful as a placeholder or default processor

### DeviceProcessor

Processes transitions by moving tensors to the specified device:

#### Key Features:
- Moves all tensors in the transition to the specified device
- Supports both CPU and GPU devices
- Non-blocking transfers for CUDA devices

#### Example Usage:
```python
processor = DeviceProcessor(device="cuda")
processed_transition = processor(transition)
```

### NormalizerProcessor

Normalizes observations and actions in a single processor step:

#### Key Features:
- Handles normalization of both observation and action tensors
- Supports mean/std normalization and min/max scaling
- Configurable to normalize only specific keys
- Integration with LeRobotDataset statistics

#### Example Usage:
```python
normalizer = NormalizerProcessor.from_lerobot_dataset(
    dataset=dataset,
    features=features,
    norm_map=norm_map
)
processed_transition = normalizer(transition)
```

### UnnormalizerProcessor

Inverse normalization for observations and actions:

#### Key Features:
- Exactly mirrors NormalizerProcessor but applies the inverse transform
- Supports mean/std denormalization and min/max denormalization
- Integration with LeRobotDataset statistics

#### Example Usage:
```python
unnormalizer = UnnormalizerProcessor.from_lerobot_dataset(
    dataset=dataset,
    features=features,
    norm_map=norm_map
)
processed_transition = unnormalizer(transition)
```

### VanillaObservationProcessor

Processes environment observations into the LeRobot format:

#### Key Features:
- Handles both images and states
- Converts channel-last images to channel-first format
- Normalizes uint8 images to float32
- Adds batch dimensions when missing
- Supports single images and image dictionaries

#### Example Usage:
```python
processor = VanillaObservationProcessor()
processed_observation = processor.observation(observation)
```

### RenameProcessor

Renames keys in the observation:

#### Key Features:
- Maps observation keys according to a rename map
- Preserves keys not in the rename map
- Simple key transformation

#### Example Usage:
```python
rename_map = {"old_key": "new_key"}
processor = RenameProcessor(rename_map=rename_map)
processed_observation = processor.observation(observation)
```

## Data Structures

### EnvTransition

Typed dictionary representing environment transitions:

#### Keys:
- `OBSERVATION`: dict[str, Any] | None
- `ACTION`: Any | torch.Tensor | None
- `REWARD`: float | torch.Tensor | None
- `DONE`: bool | torch.Tensor | None
- `TRUNCATED`: bool | torch.Tensor | None
- `INFO`: dict[str, Any] | None
- `COMPLEMENTARY_DATA`: dict[str, Any] | None

### TransitionKey

Enumeration for accessing EnvTransition dictionary components:

#### Values:
- `OBSERVATION`
- `ACTION`
- `REWARD`
- `DONE`
- `TRUNCATED`
- `INFO`
- `COMPLEMENTARY_DATA`

## Configuration System

### JSON Serialization

The framework supports JSON serialization for processor configurations:

#### Features:
- Safe serialization of processor configurations
- Support for complex data types
- Integration with Hugging Face Hub

#### Example Configuration:
```json
{
  "name": "MyProcessor",
  "steps": [
    {
      "registry_name": "normalizer_processor",
      "config": {
        "eps": 1e-8
      }
    }
  ]
}
```

### State Management

Safe tensor serialization using safetensors:

#### Features:
- Efficient tensor serialization
- Cross-platform compatibility
- Integration with processor state management

#### Example Usage:
```python
# Save processor state
processor.save_pretrained(save_directory)

# Load processor state
processor = RobotProcessor.from_pretrained(save_directory)
```

## Advanced Features

### Hook System

Processor-level hooks for observation/monitoring:

#### Features:
- Before-step hooks executed before each processor step
- After-step hooks executed after each processor step
- Non-modifying hooks for logging and debugging

#### Example Usage:
```python
def before_hook(step_index, transition):
    print(f"Before step {step_index}")

processor.register_before_step_hook(before_hook)
```

### Feature Contract System

Automatic feature key transformation:

#### Features:
- Standardized feature key mapping
- Integration with policy feature contracts
- Support for complex key transformations

#### Example Usage:
```python
features = {"pixels": PolicyFeature(type=FeatureType.VISUAL, shape=(3, 224, 224))}
contracted_features = processor.feature_contract(features)
```

### Pipeline Composition

Flexible pipeline composition:

#### Features:
- Ordered execution of processor steps
- Support for slicing and indexing
- Dynamic pipeline modification

#### Example Usage:
```python
# Create a pipeline with multiple steps
steps = [NormalizerProcessor(...), DeviceProcessor(...)]
processor = RobotProcessor(steps=steps)

# Slice the pipeline
sub_processor = processor[1:3]
```

## Best Practices

### 1. Processor Design

Design processors with clear responsibilities:
- Single responsibility principle
- Immutable input/output handling
- Proper error handling and validation

### 2. Configuration Management

Manage processor configurations effectively:
- Use JSON serialization for persistence
- Validate configuration parameters
- Handle version compatibility

### 3. State Management

Handle processor state appropriately:
- Separate configuration from state
- Use safetensors for tensor serialization
- Implement proper state loading/unloading

### 4. Error Handling

Implement robust error handling:
- Validate input data formats
- Handle device compatibility
- Provide meaningful error messages

### 5. Performance Optimization

Optimize processor performance:
- Minimize data copying
- Use efficient tensor operations
- Implement proper batching

## Extending the Framework

### Adding New Processor Steps

1. Create a new class implementing the ProcessorStep protocol
2. Register it with the ProcessorStepRegistry
3. Implement required methods

#### Example:
```python
@ProcessorStepRegistry.register(name="my_processor")
class MyProcessor:
    def __init__(self, param):
        self.param = param
    
    def __call__(self, transition):
        # Processing logic
        return transition
    
    def get_config(self):
        return {"param": self.param}
    
    def feature_contract(self, features):
        return features
```

### Custom Serialization

Implement custom serialization for complex processors:

#### Example:
```python
class CustomProcessor:
    def get_config(self):
        return {"complex_param": self.serialize_complex_param()}
    
    def state_dict(self):
        return {"tensor_state": self.tensor_param}
    
    def load_state_dict(self, state):
        self.tensor_param = state["tensor_state"]
```

## Integration with Training Pipeline

### Configuration in TrainPipelineConfig

The processor framework integrates seamlessly with the training pipeline:

#### Example:
```python
from lerobot.configs.train import TrainPipelineConfig

train_config = TrainPipelineConfig(
    processor=RobotProcessor(
        steps=[
            VanillaObservationProcessor(),
            NormalizerProcessor(...),
            DeviceProcessor(device="cuda")
        ]
    )
)
```

### Policy Integration

Policies can define processor presets:

#### Example:
```python
class MyPolicy(PreTrainedPolicy):
    def get_processor_preset(self) -> RobotProcessor:
        return RobotProcessor(
            steps=[
                VanillaObservationProcessor(),
                NormalizerProcessor(...),
                DeviceProcessor(device="cuda")
            ]
        )
```

## Requirements

To use the processor framework, you'll need:
- PyTorch for tensor operations
- Safetensors for state persistence
- Einops for tensor rearrangement
- Hugging Face Hub for model sharing
- NumPy for numerical operations

## Troubleshooting

### Common Issues

1. **Processor Registration**: Ensure processors are properly registered
2. **Device Compatibility**: Verify tensor device placement
3. **State Loading Failures**: Check version compatibility between save/load operations
4. **Feature Contract Mismatches**: Validate feature key transformations

### Debugging Tips

1. Enable verbose logging to monitor processing steps
2. Use step_through method to debug pipeline execution
3. Validate processor configurations before instantiation
4. Monitor tensor memory usage during processing

This processor framework provides a flexible and extensible foundation for data transformation in robotic control systems, with support for composable processing pipelines, comprehensive state management, and seamless integration with training workflows.