# PI0+FAST Policy Documentation

## Overview

PI0+FAST is an efficient action tokenization method for Vision-Language-Action models developed by Physical Intelligence. This implementation is a PyTorch port of the original JAX implementation from the openpi repository.

The key innovation of PI0+FAST is the use of discrete action tokenization via the FAST (Function Approximation for Scalable Tokens) tokenizer, which enables more efficient action representation and generation compared to continuous methods.

**Disclaimer**: This port is not expected to perform as well as the original implementation. The original JAX implementation can be found at: https://github.com/Physical-Intelligence/openpi

## Key Components

### Configuration (`configuration_pi0fast.py`)
The PI0FASTConfig class defines all hyperparameters and settings for the policy:
- Input/output structure (observation steps, chunk size, action steps)
- Normalization settings for different input types
- Image preprocessing parameters
- Model architecture parameters (tokenizer, projector, decoding)
- Attention and caching settings
- Finetuning settings
- Training presets (optimizer, scheduler)

### Main Model (`modeling_pi0fast.py`)
The core implementation includes:
- PI0FASTPolicy: Wrapper class for training and inference within LeRobot
- PI0FAST: The main model implementation with action tokenization
- Various helper functions for data preprocessing and postprocessing

## Usage

### Training
Example of finetuning the pi0+FAST pretrained model:
```bash
lerobot-train \
--policy.path=lerobot/pi0fast_base \
--dataset.repo_id=danaaubakirova/koch_test
```

Example of training the pi0+FAST neural network from scratch:
```bash
lerobot-train \
--policy.type=pi0fast \
--dataset.repo_id=danaaubakirova/koch_test
```

### Inference
Example of using the pi0+FAST pretrained model outside LeRobot training framework:
```python
policy = PI0FASTPolicy.from_pretrained("lerobot/pi0fast_base")
```

## Model Architecture

The PI0+FAST architecture consists of:
1. PaliGemma: Processes images and language tokens
2. FAST Tokenizer: Discrete action tokenization using DCT-based compression
3. Action Decoding: Converts discrete tokens back to continuous actions

The model uses a tokenized approach for action generation, where continuous actions are compressed into discrete tokens using the FAST tokenizer, and then decompressed during inference.

## Key Features

### FAST Tokenization
- Uses Discrete Cosine Transform (DCT) for action compression
- Tokenizes actions into discrete representations
- Enables efficient sequence modeling of continuous actions

### Block Causal Attention
- Implements block-wise attention masking for efficient training
- Allows prefix processing with separate attention for action tokens

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

## Special Components

### prepare_inputs_for_generation
Custom input preparation for generation that handles:
- Block causal attention masks
- Position IDs management
- Pixel values handling for different generation stages

### Action Decoding
The model implements specialized functions for:
- Converting tokens back to actions using inverse DCT
- Handling variable sequence lengths with relaxed decoding
- Padding/truncation for consistent action dimensions

## Training Details

The model uses cross-entropy loss for training on the tokenized actions:
- Next-token prediction with causal masking
- Loss masking to exclude padding and prefix tokens
- Gradient clipping for stable training

## Requirements

The PI0+FAST policy requires:
- Standard LeRobot dependencies
- Transformers library with PaliGemma support
- FAST tokenizer from Physical Intelligence
- Image processing libraries (PIL, scipy)