# LeRobot Camera Framework

This document provides an overview of the camera framework implemented in the LeRobot library, which supports various camera backends for robotic applications.

## Overview

The LeRobot camera framework provides a standardized interface for capturing frames from different types of cameras. It supports multiple backends including OpenCV-compatible cameras and Intel RealSense cameras.

## Core Components

### Camera Base Class

The `Camera` class is an abstract base class that defines the standard interface for all camera implementations:

- `is_connected`: Property to check camera connection status
- `connect()`: Establish connection to the camera
- `read()`: Synchronously capture a single frame
- `async_read()`: Asynchronously capture a single frame
- `disconnect()`: Disconnect from the camera and release resources
- `find_cameras()`: Static method to detect available cameras

### Camera Configuration

The `CameraConfig` class is an abstract base class for camera configurations using draccus choice registry pattern. It supports:

- FPS settings
- Resolution settings (width, height)
- Color mode (RGB/BGR)
- Rotation settings

## Supported Camera Types

### OpenCV Cameras

OpenCV camera support is provided through the `OpenCVCamera` class, which can interface with USB cameras, built-in webcams, and other OpenCV-compatible devices.

#### Features:
- Synchronous and asynchronous frame capture
- Configurable FPS, resolution, and color mode
- Image rotation support
- Automatic warmup period
- Thread-safe asynchronous reading

#### Configuration:
```python
OpenCVCameraConfig(
    index_or_path=0,  # Camera index or device path
    fps=30,
    width=1280,
    height=720,
    color_mode=ColorMode.RGB,
    rotation=Cv2Rotation.NO_ROTATION,
    warmup_s=1
)
```

#### Usage Example:
```python
from lerobot.cameras.opencv import OpenCVCamera
from lerobot.cameras.opencv.configuration_opencv import OpenCVCameraConfig

config = OpenCVCameraConfig(index_or_path=0)
camera = OpenCVCamera(config)
camera.connect()

# Synchronous read
color_image = camera.read()

# Asynchronous read
async_image = camera.async_read()

camera.disconnect()
```

### Intel RealSense Cameras

Intel RealSense camera support is provided through the `RealSenseCamera` class, which offers advanced features like depth sensing.

#### Features:
- Synchronous frame capture for color and depth
- Configurable FPS, resolution, and color mode
- Depth map capture capability
- Device identification by serial number or name
- Image rotation support
- Automatic warmup period

#### Configuration:
```python
RealSenseCameraConfig(
    serial_number_or_name="0123456789",  # Serial number or camera name
    fps=30,
    width=1280,
    height=720,
    color_mode=ColorMode.RGB,
    use_depth=False,  # Enable depth stream
    rotation=Cv2Rotation.NO_ROTATION,
    warmup_s=1
)
```

#### Usage Example:
```python
from lerobot.cameras.realsense import RealSenseCamera
from lerobot.cameras.realsense.configuration_realsense import RealSenseCameraConfig

config = RealSenseCameraConfig(serial_number_or_name="0123456789")
camera = RealSenseCamera(config)
camera.connect()

# Read color frame
color_image = camera.read()

# Read depth frame (if use_depth=True)
if config.use_depth:
    depth_map = camera.read_depth()

camera.disconnect()
```

## Camera Discovery

The framework provides utilities to discover connected cameras:

```bash
# Find OpenCV cameras
lerobot-find-cameras opencv

# Find RealSense cameras
lerobot-find-cameras realsense
```

## Key Features

### Thread-Safe Asynchronous Reading

Both camera implementations support asynchronous frame reading through background threads, allowing non-blocking frame capture for real-time applications.

### Image Processing

- Automatic color space conversion (RGB/BGR)
- Image rotation support (0°, 90°, 180°, 270°)
- Dimension validation
- Error handling for disconnected devices

### Configuration Management

- Type-safe configuration using dataclasses
- draccus choice registry for easy configuration management
- Validation of configuration parameters
- Support for partial configuration (using device defaults)

### Cross-Platform Support

- Windows: MSMF backend with hardware transform disabled
- Linux: Standard OpenCV backend
- macOS: Standard OpenCV backend

## Error Handling

The framework includes comprehensive error handling:

- `DeviceAlreadyConnectedError`: Raised when trying to connect an already connected camera
- `DeviceNotConnectedError`: Raised when operations are attempted on a disconnected camera
- `ConnectionError`: Raised when camera connection fails
- `RuntimeError`: Raised for various runtime issues
- `TimeoutError`: Raised for asynchronous read timeouts

## Best Practices

1. Always properly disconnect cameras when done:
```python
try:
    camera.connect()
    # ... use camera ...
finally:
    camera.disconnect()
```

2. Handle exceptions appropriately:
```python
try:
    frame = camera.read()
except DeviceNotConnectedError:
    # Handle disconnected camera
    pass
except RuntimeError as e:
    # Handle other errors
    pass
```

3. Use asynchronous reading for real-time applications to avoid blocking:
```python
# Non-blocking frame capture
try:
    frame = camera.async_read(timeout_ms=100)
except TimeoutError:
    # Handle timeout
    pass
```

## Requirements

To use the camera framework, you'll need:

- OpenCV for basic camera support
- pyrealsense2 for Intel RealSense camera support (optional)
- Appropriate camera drivers for your hardware