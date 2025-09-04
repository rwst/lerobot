# PI0 Policy Documentation

## Overview

PI0 is a Vision-Language-Action Flow Model for General Robot Control developed by Physical Intelligence. This implementation is a PyTorch port of the original JAX implementation from the openpi repository.

**Disclaimer**: This port is not expected to perform as well as the original implementation. The original JAX implementation can be found at: https://github.com/Physical-Intelligence/openpi

## Key Components

### Configuration (`configuration_pi0.py`)
The PI0Config class defines all hyperparameters and settings for the policy:
- Input/output structure (observation steps, chunk size, action steps)
- Normalization settings for different input types
- Image preprocessing parameters
- Model architecture parameters (tokenizer, projector, decoding)
- Attention implementation options
- Finetuning settings
- Training presets (optimizer, scheduler)

### Main Model (`modeling_pi0.py`)
The core implementation includes:
- PI0Policy: Wrapper class for training and inference within LeRobot
- PI0FlowMatching: The main flow matching model implementation
- Various helper functions for data preprocessing and postprocessing

### PaliGemma with Expert (`paligemma_with_expert.py`)
Implements the combined vision-language model:
- PaliGemma as the base vision-language model
- Gemma Expert as the action generation expert
- Custom attention implementations

## Usage

### Training
Example of finetuning the pi0 pretrained model:
```bash
lerobot-train \
--policy.path=lerobot/pi0 \
--dataset.repo_id=danaaubakirova/koch_test
```

Example of finetuning with pretrained VLM parameters:
```bash
lerobot-train \
--policy.type=pi0 \
--dataset.repo_id=danaaubakirova/koch_test
```

### Inference
Example of using the pi0 pretrained model outside LeRobot training framework:
```python
policy = Pi0Policy.from_pretrained("lerobot/pi0")
```

## Model Architecture

The PI0 architecture consists of:
1. PaliGemma: Processes images and language tokens
2. Gemma Expert: Generates actions based on the encoded inputs
3. Projection layers: Transform state and action representations
4. Time embedding MLPs: Incorporate temporal information in the diffusion process

The model uses a flow matching approach for action generation, where noise is gradually removed from random noise to produce coherent actions.

## Conversion Scripts

The policy includes scripts for converting JAX checkpoints to PyTorch:
- `convert_pi0_to_hf_lerobot.py`: Main conversion script
- `conversion_utils.py`: Utility functions for configuration handling

## Attention Implementations

Three attention implementations are supported:
- `eager`: Standard PyTorch implementation
- `fa2`: Flash Attention 2 (not yet implemented)
- `flex`: Flex Attention (available from PyTorch 2.5+)

## Special Features

### ALOHA Adaptation
The model includes special handling for ALOHA datasets:
- State decoding/encoding for compatibility with PI0's training space
- Gripper position transformations
- Joint flipping for specific motors

### Normalization
Comprehensive normalization support for different input types:
- Visual inputs: Identity normalization
- State inputs: Mean/std normalization
- Action inputs: Mean/std normalization

## Requirements

To use the PI0 policy, install the extra dependencies:
```bash
pip install -e ".[pi0]"
```