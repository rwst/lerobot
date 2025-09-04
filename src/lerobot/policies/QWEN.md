# LeRobot Policies

This document provides an overview of the policies implemented in the LeRobot library, with a focus on the PI0 and PI0+FAST policies.

## Overview

LeRobot implements several robotic control policies, including:
- ACT (Action Chunking Transformers)
- Diffusion Policy
- PI0 (Vision-Language-Action Flow Model)
- PI0+FAST (Efficient Action Tokenization for Vision-Language-Action Models)
- SAC (Soft Actor-Critic)
- SmolVLA (Vision-Language-Action Model)
- TDMPC (Temporal Difference Learning for Model Predictive Control)
- VQBeT (Behavior Generation with Latent Actions)

## PI0 Policy

PI0 is a Vision-Language-Action Flow Model for General Robot Control developed by Physical Intelligence. This implementation is a PyTorch port of the original JAX implementation from the openpi repository.

### Key Features

- **Flow Matching Approach**: Uses a flow matching algorithm to generate actions from noise
- **Vision-Language Integration**: Combines PaliGemma vision-language model with a Gemma expert for action generation
- **Modular Architecture**: Separates vision encoding, language processing, and action generation
- **Flexible Attention**: Supports different attention implementations (eager, FA2, flex)

### Architecture

The PI0 architecture consists of:
1. PaliGemma: Processes images and language tokens
2. Gemma Expert: Generates actions based on the encoded inputs
3. Projection layers: Transform state and action representations
4. Time embedding MLPs: Incorporate temporal information in the diffusion process

The model uses a flow matching approach for action generation, where noise is gradually removed from random noise to produce coherent actions.

### Special Features

#### ALOHA Adaptation
The model includes special handling for ALOHA datasets:
- State decoding/encoding for compatibility with PI0's training space
- Gripper position transformations
- Joint flipping for specific motors

#### Normalization
Comprehensive normalization support for different input types:
- Visual inputs: Identity normalization
- State inputs: Mean/std normalization
- Action inputs: Mean/std normalization

### Configuration

Key configuration parameters include:
- Input/output structure (observation steps, chunk size, action steps)
- Normalization settings
- Image preprocessing parameters
- Model architecture parameters (tokenizer, projector, decoding)
- Attention implementation options
- Finetuning settings

## PI0+FAST Policy

PI0+FAST is an efficient action tokenization method for Vision-Language-Action models. It uses discrete action tokenization via the FAST (Function Approximation for Scalable Tokens) tokenizer.

### Key Features

- **FAST Tokenization**: Uses Discrete Cosine Transform (DCT) for action compression
- **Tokenized Actions**: Converts continuous actions into discrete tokens
- **Efficient Sequence Modeling**: Enables processing of compressed action sequences
- **Block Causal Attention**: Implements specialized attention patterns for training

### Architecture

The PI0+FAST architecture consists of:
1. PaliGemma: Processes images and language tokens
2. FAST Tokenizer: Discrete action tokenization using DCT-based compression
3. Action Decoding: Converts discrete tokens back to continuous actions

The model uses a tokenized approach for action generation, where continuous actions are compressed into discrete tokens using the FAST tokenizer, and then decompressed during inference.

### Key Components

#### FAST Tokenizer
- Uses Discrete Cosine Transform (DCT) for action compression
- Tokenizes actions into discrete representations
- Enables efficient sequence modeling of continuous actions

#### Block Causal Attention
- Implements block-wise attention masking for efficient training
- Allows prefix processing with separate attention for action tokens

### Special Features

#### ALOHA Adaptation
The model includes special handling for ALOHA datasets:
- State decoding/encoding for compatibility with PI0's training space
- Gripper position transformations
- Joint flipping for specific motors

#### Normalization
Comprehensive normalization support for different input types:
- Visual inputs: Identity normalization
- State inputs: Mean/std normalization
- Action inputs: Mean/std normalization

### Configuration

Key configuration parameters include:
- Input/output structure (observation steps, chunk size, action steps)
- Normalization settings
- Image preprocessing parameters
- Model architecture parameters (tokenizer, projector, decoding)
- Attention and caching settings
- Training presets

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

Example of finetuning the pi0+FAST pretrained model:
```bash
lerobot-train \
--policy.path=lerobot/pi0fast_base \
--dataset.repo_id=danaaubakirova/koch_test
```

### Inference
Example of using the pi0 pretrained model outside LeRobot training framework:
```python
policy = Pi0Policy.from_pretrained("lerobot/pi0")
```

Example of using the pi0+FAST pretrained model:
```python
policy = PI0FASTPolicy.from_pretrained("lerobot/pi0fast_base")
```

## Requirements

To use the PI0 and PI0+FAST policies, install the extra dependencies:
```bash
pip install -e ".[pi0]"
```