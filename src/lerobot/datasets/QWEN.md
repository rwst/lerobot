# LeRobot Datasets Module

This document provides an overview of the datasets module in the LeRobot library, which is responsible for handling robotic datasets for training and evaluation.

## Overview

The LeRobot datasets module provides a standardized interface for working with robotic datasets. It supports loading, creating, and managing datasets in various formats, with special support for video and image data. The module is designed to work seamlessly with the rest of the LeRobot framework.

## Core Components

### LeRobotDataset

The `LeRobotDataset` class is the primary interface for working with datasets in LeRobot. It provides functionality for:

- Loading existing datasets from local storage or Hugging Face Hub
- Creating new datasets for data recording
- Managing dataset metadata and statistics
- Handling video and image data efficiently
- Supporting delta timestamps for temporal data

#### Key Features:

1. **Flexible Data Loading**: Can load datasets from local storage or directly from the Hugging Face Hub.
2. **Video Support**: Built-in support for video data with efficient decoding using multiple backends (torchcodec, pyav).
3. **Chunked Storage**: Datasets are organized in chunks for efficient storage and retrieval.
4. **Metadata Management**: Comprehensive metadata handling including dataset statistics, task information, and feature definitions.
5. **Delta Timestamps**: Support for temporal data with delta timestamp functionality.

#### Dataset Structure:

A typical LeRobotDataset has the following structure:

```
├── data/
│   ├── chunk-000/
│   │   ├── episode_000000.parquet
│   │   ├── episode_000001.parquet
│   │   └── ...
│   └── ...
├── meta/
│   ├── episodes.jsonl
│   ├── info.json
│   ├── stats.json
│   └── tasks.jsonl
└── videos/
    ├── chunk-000/
    │   ├── observation.images.camera1/
    │   │   ├── episode_000000.mp4
    │   │   ├── episode_000001.mp4
    │   │   └── ...
    │   └── ...
    └── ...
```

#### Key Methods:

- `__init__()`: Initialize a dataset from existing data
- `create()`: Create a new empty dataset for recording
- `add_frame()`: Add a frame to the current episode buffer
- `save_episode()`: Save the current episode buffer to disk
- `push_to_hub()`: Push the dataset to the Hugging Face Hub
- `__getitem__()`: Access individual frames with optional delta timestamp support

### LeRobotDatasetMetadata

The `LeRobotDatasetMetadata` class handles all metadata for a dataset, including:

- Dataset information (version, robot type, FPS, etc.)
- Feature definitions and shapes
- Episode information
- Task definitions
- Statistical information

### MultiLeRobotDataset

The `MultiLeRobotDataset` class allows combining multiple `LeRobotDataset` instances into a single dataset. This is useful for training on multiple datasets simultaneously.

## Dataset Conversion and Versioning

### Version 2.0 Conversion

The module includes utilities for converting datasets from version 1.6 to 2.0 format:

- `convert_dataset_v1_to_v2.py`: Script for converting v1.6 datasets to v2.0
- Supports single-task and multi-task datasets
- Handles robot configuration information

### Version 2.1 Conversion

Support for converting datasets from version 2.0 to 2.1:

- `convert_dataset_v20_to_v21.py`: Script for converting v2.0 datasets to v2.1
- Introduces per-episode statistics instead of global statistics

## Key Utilities

### Video Utilities

The `video_utils.py` module provides:

- Video frame decoding with multiple backends (torchcodec, pyav)
- Video encoding using ffmpeg
- Video information extraction
- Frame sampling and timestamp synchronization

### Statistics Computation

The `compute_stats.py` module provides:

- Statistical computation for dataset features
- Per-episode and aggregated statistics
- Image sampling for efficient statistics computation
- Parallel algorithm for variance computation

### Data Transforms

The `transforms.py` module provides:

- Image transformation utilities
- Random subset application of transforms
- Sharpness jittering
- Configurable image augmentation pipelines

### Factory Functions

The `factory.py` module provides:

- Dataset creation functions
- Delta timestamp resolution
- ImageNet statistics integration

## Data Processing Pipeline

### Online Buffer

The `OnlineBuffer` class provides:

- FIFO data buffer for online training
- Memory-mapped storage for efficient access
- Support for delta timestamps
- Circular buffer implementation

### Sampler

The `EpisodeAwareSampler` class provides:

- Episode-aware sampling for training
- Support for dropping frames from episode boundaries
- Shuffle functionality

### Image Writer

The `AsyncImageWriter` class provides:

- Asynchronous image writing for high frame rate recording
- Thread-based and process-based implementations
- Queue-based task management

## Dataset Features

### Feature Types

Datasets support various feature types:

- **State**: Robot state information (joint positions, etc.)
- **Action**: Robot action information (joint commands, etc.)
- **Image**: Camera image data stored as PNG files
- **Video**: Camera video data stored as MP4 files
- **Timestamp**: Frame timestamps for synchronization

### Feature Normalization

The module supports feature normalization:

- Per-feature statistics (mean, std, min, max)
- Image normalization (0-255 to 0-1 range)
- Integration with ImageNet statistics

## Configuration

### Image Transforms Configuration

The module supports configurable image transforms:

```python
ImageTransformsConfig(
    enable=True,
    max_num_transforms=3,
    random_order=False,
    tfs={
        "brightness": ImageTransformConfig(
            weight=1.0,
            type="ColorJitter",
            kwargs={"brightness": (0.8, 1.2)},
        ),
        # ... other transforms
    }
)
```

## Best Practices

### Working with Large Datasets

1. Use chunked storage for efficient data access
2. Leverage video compression for image sequences
3. Use appropriate video backends for your platform
4. Consider using delta timestamps for temporal data

### Data Recording

1. Use `AsyncImageWriter` for high frame rate recording
2. Batch video encoding for better performance
3. Handle interruptions gracefully with context managers
4. Validate frame data before adding to dataset

### Dataset Creation

1. Define appropriate features for your robot
2. Choose suitable FPS for your application
3. Decide between image and video storage based on your needs
4. Provide accurate task descriptions for multi-task datasets

## Error Handling

The module includes comprehensive error handling:

- Version compatibility checks
- Data validation for frames and episodes
- Disk space checking for buffer operations
- Video decoding error handling
- Network error handling for Hub operations

## Extending the Framework

### Adding New Feature Types

To add support for new feature types:

1. Extend the feature definition system
2. Implement appropriate data validation
3. Add statistics computation support
4. Update the data loading pipeline

### Custom Video Backends

To add support for new video backends:

1. Implement a new decode function
2. Add backend selection logic
3. Handle backend-specific error cases
4. Update documentation

## Integration with Training Pipeline

The datasets module integrates seamlessly with the training pipeline:

- Factory functions for dataset creation from configs
- Automatic delta timestamp resolution
- ImageNet statistics integration
- Sampler weight computation for online training

## Requirements

To use the datasets module, you'll need:

- PyTorch for tensor operations
- Hugging Face datasets for data loading
- Hugging Face Hub for dataset sharing
- FFmpeg for video encoding
- TorchVision for image operations
- PyAV or TorchCodec for video decoding (optional)