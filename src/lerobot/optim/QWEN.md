# LeRobot Optimizers Framework

This document provides an overview of the optimizers framework implemented in the LeRobot library, which manages optimization algorithms and learning rate schedulers for training robotic policies.

## Overview

The LeRobot optimizers framework provides a standardized interface for configuring and managing optimization algorithms and learning rate schedulers. It supports multiple optimizer types and learning rate scheduling strategies through a draccus-based configuration system.

## Core Components

### Optimizer Configuration

The `OptimizerConfig` class is an abstract base class that defines the standard interface for optimizer configurations using the draccus choice registry pattern:

#### Key Features:
- Draccus choice registry for polymorphic configurations
- Common hyperparameters (learning rate, weight decay, gradient clipping)
- Abstract build method for optimizer instantiation

#### Supported Optimizers:

1. **Adam** (`adam`)
   - Default learning rate: 1e-3
   - Betas: (0.9, 0.999)
   - Epsilon: 1e-8
   - Weight decay: 0.0
   - Gradient clipping norm: 10.0

2. **AdamW** (`adamw`)
   - Default learning rate: 1e-3
   - Betas: (0.9, 0.999)
   - Epsilon: 1e-8
   - Weight decay: 1e-2
   - Gradient clipping norm: 10.0

3. **SGD** (`sgd`)
   - Default learning rate: 1e-3
   - Momentum: 0.0
   - Dampening: 0.0
   - Nesterov: False
   - Weight decay: 0.0
   - Gradient clipping norm: 10.0

4. **Multi-Adam** (`multi_adam`)
   - Multiple Adam optimizers with different parameter groups
   - Group-specific hyperparameters
   - Default learning rate: 1e-3
   - Default weight decay: 0.0
   - Gradient clipping norm: 10.0

### Learning Rate Scheduler Configuration

The `LRSchedulerConfig` class is an abstract base class for learning rate scheduler configurations:

#### Supported Schedulers:

1. **Diffuser** (`diffuser`)
   - Integration with diffusers optimization library
   - Scheduler types: "linear", "cosine", "cosine_with_restarts", etc.
   - Warmup steps configuration

2. **VQBeT** (`vqbet`)
   - Specialized scheduler for VQ-BeT training
   - Warmup steps for VQ-VAE training
   - Cosine decay with cycles
   - Peak and decay learning rates

3. **Cosine Decay with Warmup** (`cosine_decay_with_warmup`)
   - Linear warmup followed by cosine decay
   - Configurable peak and decay learning rates
   - Custom warmup and decay steps

## Configuration System

### Draccus Integration

The framework uses draccus for command-line argument parsing and configuration management:

```bash
# Configure optimizer from command line
python train.py --optimizer.type=adam --optimizer.lr=1e-4 --optimizer.weight_decay=1e-5

# Configure scheduler
python train.py --scheduler.type=cosine_decay_with_warmup --scheduler.num_warmup_steps=1000
```

### Choice Registry Pattern

Optimizers and schedulers are registered using the draccus choice registry pattern:

```python
@OptimizerConfig.register_subclass("adam")
@dataclass
class AdamConfig(OptimizerConfig):
    # implementation
    pass
```

## Factory Functions

### Optimizer and Scheduler Creation

The `make_optimizer_and_scheduler` function creates optimizers and schedulers based on configurations:

```python
from lerobot.optim.factory import make_optimizer_and_scheduler

# Create optimizer and scheduler
optimizer, scheduler = make_optimizer_and_scheduler(train_config, policy)
```

#### Parameters:
- `cfg`: TrainPipelineConfig containing optimizer and scheduler configurations
- `policy`: PreTrainedPolicy from which parameters are extracted

#### Returns:
- `tuple[Optimizer, LRScheduler | None]`: Optimizer and optional scheduler

## State Management

### Optimizer State Persistence

Functions for saving and loading optimizer states:

```python
from lerobot.optim.optimizers import save_optimizer_state, load_optimizer_state

# Save optimizer state
save_optimizer_state(optimizer, save_dir)

# Load optimizer state
optimizer = load_optimizer_state(optimizer, save_dir)
```

#### Features:
- Support for single optimizers and dictionaries of optimizers
- Safe tensor serialization using safetensors
- Parameter group preservation
- Cross-platform compatibility

### Scheduler State Persistence

Functions for saving and loading scheduler states:

```python
from lerobot.optim.schedulers import save_scheduler_state, load_scheduler_state

# Save scheduler state
save_scheduler_state(scheduler, save_dir)

# Load scheduler state
scheduler = load_scheduler_state(scheduler, save_dir)
```

## Advanced Features

### Multi-Optimizer Support

The MultiAdamConfig supports multiple optimizers for different parameter groups:

```python
# Configuration for multiple optimizers
optimizer_config = MultiAdamConfig(
    optimizer_groups={
        "policy": {"lr": 1e-4, "weight_decay": 1e-5},
        "value": {"lr": 1e-3, "weight_decay": 1e-4}
    }
)

# Build optimizers for different parameter groups
params_dict = {
    "policy": policy_parameters,
    "value": value_parameters
}
optimizers = optimizer_config.build(params_dict)
```

### Gradient Clipping

Built-in gradient clipping support:

```python
# Gradient clipping is automatically applied during training
# Based on the grad_clip_norm parameter in optimizer configuration
optimizer_config = AdamConfig(
    lr=1e-4,
    grad_clip_norm=5.0  # Clip gradients to norm 5.0
)
```

### Custom Scheduler Implementation

Creating custom learning rate schedulers:

```python
@LRSchedulerConfig.register_subclass("custom")
@dataclass
class CustomSchedulerConfig(LRSchedulerConfig):
    def build(self, optimizer: Optimizer, num_training_steps: int) -> LambdaLR:
        def lr_lambda(current_step):
            # Custom learning rate schedule
            return max(0.0, 1.0 - current_step / num_training_steps)
        
        return LambdaLR(optimizer, lr_lambda, -1)
```

## Best Practices

### 1. Optimizer Selection

Choose optimizers based on use case:
- **Adam/AdamW**: General purpose, good default choice
- **SGD**: When fine-tuning with momentum
- **Multi-Adam**: For policies with distinct parameter groups

### 2. Learning Rate Scheduling

Select appropriate schedulers:
- **Cosine decay**: Smooth decay for most training scenarios
- **Warmup**: Essential for transformer-based policies
- **Custom schedules**: For specialized training procedures

### 3. Hyperparameter Tuning

Recommended starting points:
```python
# For transformer policies
optimizer_config = AdamWConfig(
    lr=1e-4,
    weight_decay=1e-3,
    grad_clip_norm=1.0
)

scheduler_config = CosineDecayWithWarmupSchedulerConfig(
    num_warmup_steps=1000,
    num_decay_steps=50000,
    peak_lr=1e-4,
    decay_lr=1e-6
)
```

### 4. State Management

Proper state management for resumable training:
```python
# Save states during checkpointing
save_optimizer_state(optimizer, checkpoint_dir)
save_scheduler_state(scheduler, checkpoint_dir)

# Load states when resuming
optimizer = load_optimizer_state(optimizer, checkpoint_dir)
scheduler = load_scheduler_state(scheduler, checkpoint_dir)
```

## Extending the Framework

### Adding New Optimizers

1. Create a new config class inheriting from `OptimizerConfig`
2. Register it with the choice registry
3. Implement the required build method

```python
@OptimizerConfig.register_subclass("rmsprop")
@dataclass
class RMSPropConfig(OptimizerConfig):
    lr: float = 1e-3
    alpha: float = 0.99
    eps: float = 1e-8
    weight_decay: float = 0.0
    grad_clip_norm: float = 10.0
    
    def build(self, params: dict) -> torch.optim.Optimizer:
        kwargs = asdict(self)
        kwargs.pop("grad_clip_norm")
        return torch.optim.RMSprop(params, **kwargs)
```

### Adding New Schedulers

1. Create a new config class inheriting from `LRSchedulerConfig`
2. Register it with the choice registry
3. Implement the required build method

```python
@LRSchedulerConfig.register_subclass("step")
@dataclass
class StepSchedulerConfig(LRSchedulerConfig):
    step_size: int
    gamma: float = 0.1
    num_warmup_steps: int = 0
    
    def build(self, optimizer: Optimizer, num_training_steps: int) -> LRScheduler:
        return torch.optim.lr_scheduler.StepLR(
            optimizer, 
            step_size=self.step_size, 
            gamma=self.gamma
        )
```

## Integration with Training Pipeline

### Configuration in TrainPipelineConfig

The optimization framework integrates seamlessly with the training pipeline:

```python
from lerobot.configs.train import TrainPipelineConfig

# Training configuration with optimizer and scheduler
train_config = TrainPipelineConfig(
    optimizer=AdamWConfig(lr=1e-4, weight_decay=1e-3),
    scheduler=CosineDecayWithWarmupSchedulerConfig(
        num_warmup_steps=1000,
        num_decay_steps=50000,
        peak_lr=1e-4,
        decay_lr=1e-6
    ),
    use_policy_training_preset=True
)
```

### Policy Integration

Policies can define training presets:

```python
class MyPolicy(PreTrainedPolicy):
    def get_optimizer_preset(self) -> OptimizerConfig:
        return AdamWConfig(lr=1e-4, weight_decay=1e-3)
    
    def get_scheduler_preset(self) -> LRSchedulerConfig:
        return CosineDecayWithWarmupSchedulerConfig(
            num_warmup_steps=1000,
            num_decay_steps=50000,
            peak_lr=1e-4,
            decay_lr=1e-6
        )
```

## Requirements

To use the optimization framework, you'll need:
- PyTorch for optimizer implementations
- Draccus for configuration management
- Safetensors for state persistence
- Diffusers (optional) for diffuser schedulers

## Troubleshooting

### Common Issues

1. **Optimizer Mismatch**: Ensure parameter groups match optimizer expectations
2. **Scheduler Compatibility**: Verify scheduler is compatible with optimizer type
3. **State Loading Failures**: Check version compatibility between save/load operations
4. **Gradient Explosion**: Adjust grad_clip_norm parameter or learning rate

### Debugging Tips

1. Enable verbose logging to monitor learning rates
2. Use learning rate finder tools to determine optimal rates
3. Monitor gradient norms during training
4. Validate optimizer states after loading from checkpoints

This optimization framework provides a flexible and extensible foundation for training robotic policies with various optimization strategies and learning rate scheduling approaches.