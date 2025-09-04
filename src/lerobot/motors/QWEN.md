# LeRobot Motors Framework

This document provides an overview of the motors framework implemented in the LeRobot library, which supports various motor protocols for robotic control.

## Overview

The LeRobot motors framework provides a standardized interface for controlling different types of motors commonly used in robotics. It supports multiple motor protocols including Dynamixel and Feetech, with a modular design that allows for easy extension to other motor types.

## Core Components

### MotorsBus Abstract Base Class

The `MotorsBus` class is the foundation of the motor framework, providing a standardized interface for motor communication:

#### Key Features:
- Efficient read/write operations to multiple motors
- Support for both individual and synchronized operations
- Calibration management for motor ranges
- Connection management with automatic handshake
- Protocol compatibility checking

#### Main Methods:
- `connect()` / `disconnect()`: Manage serial port connections
- `read()` / `write()`: Individual motor register operations
- `sync_read()` / `sync_write()`: Batch operations for multiple motors
- `enable_torque()` / `disable_torque()`: Motor power control
- `ping()`: Motor identification and status checking

### Motor Configuration

Each motor is configured using the `Motor` dataclass:

```python
from lerobot.motors import Motor, MotorNormMode

motor = Motor(
    id=1,
    model="xl430-w250",
    norm_mode=MotorNormMode.RANGE_M100_100
)
```

#### Parameters:
- `id`: Motor ID on the bus
- `model`: Motor model identifier
- `norm_mode`: Normalization mode for position values

### Calibration System

The framework includes a comprehensive calibration system using `MotorCalibration`:

```python
from lerobot.motors import MotorCalibration

calibration = MotorCalibration(
    id=1,
    drive_mode=0,
    homing_offset=2048,
    range_min=0,
    range_max=4095
)
```

#### Calibration Features:
- Range limiting for safe operation
- Homing offset adjustment
- Drive mode configuration
- Automatic range centering

## Supported Motor Protocols

### Dynamixel Protocol

The Dynamixel implementation supports:
- Protocol 2.0 communication
- Multiple motor models (X series, XL series, XM series)
- Operating modes: Current, Velocity, Position, Extended Position, Current Position, PWM
- Drive modes: Normal and Inverted

#### Key Features:
- Broadcast ping for motor discovery
- Group synchronization for efficient communication
- Two's complement encoding for signed values
- Automatic baudrate detection

#### Example Usage:
```python
from lerobot.motors.dynamixel import DynamixelMotorsBus, OperatingMode

bus = DynamixelMotorsBus(
    port="/dev/ttyUSB0",
    motors={"joint1": Motor(id=1, model="xl430-w250")}
)
bus.connect()
bus.write("Operating_Mode", "joint1", OperatingMode.POSITION.value)
position = bus.read("Present_Position", "joint1")
```

### Feetech Protocol

The Feetech implementation supports:
- Protocol versions 0 and 1
- Multiple motor models (STS series, SCS series, SMS series)
- Operating modes: Position, Velocity, PWM, Step
- Drive modes: Normal and Inverted

#### Key Features:
- Protocol version compatibility checking
- Sign-magnitude encoding for signed values
- Firmware version consistency checking
- Custom broadcast ping implementation

#### Example Usage:
```python
from lerobot.motors.feetech import FeetechMotorsBus, OperatingMode

bus = FeetechMotorsBus(
    port="/dev/ttyUSB0",
    motors={"joint1": Motor(id=1, model="sts3215")},
    protocol_version=0
)
bus.connect()
bus.write("Operating_Mode", "joint1", OperatingMode.POSITION.value)
position = bus.read("Present_Position", "joint1")
```

## Motor Control Features

### Position Normalization

The framework supports multiple normalization modes:
- `RANGE_0_100`: Normalize to 0-100% range
- `RANGE_M100_100`: Normalize to -100% to +100% range
- `DEGREES`: Normalize to degrees of rotation

### Calibration Tools

#### Range Recording
Interactive tool for recording motor ranges:
```python
# Record min/max positions by manually moving joints
mins, maxes = bus.record_ranges_of_motion()
```

#### Half-Turn Homing
Automatically center motor ranges:
```python
# Set homing offset to center current position as half-turn
homings = bus.set_half_turn_homings()
```

#### Calibration Management
```python
# Read current calibration
calibration = bus.read_calibration()

# Write calibration to motors
bus.write_calibration(calibration)

# Reset to factory defaults
bus.reset_calibration()
```

## Connection Management

### Port Scanning
Discover motors on available ports:
```python
# Scan port at all supported baudrates
baudrate_ids = DynamixelMotorsBus.scan_port("/dev/ttyUSB0")
```

### Motor Setup
Configure individual motors:
```python
# Setup single motor with automatic ID/baudrate detection
bus.setup_motor("joint1", initial_baudrate=57600, initial_id=2)
```

### Error Handling
Robust error handling with retry mechanisms:
```python
# Retry failed operations
bus.write("Goal_Position", "joint1", 2048, num_retry=3)
```

## Advanced Features

### Torque Management
Fine-grained torque control:
```python
# Disable torque for configuration
with bus.torque_disabled(["joint1", "joint2"]):
    # Safe configuration operations
    bus.write("Homing_Offset", "joint1", 1000)

# Enable/disable specific motors
bus.enable_torque("joint1")
bus.disable_torque(["joint1", "joint2"])
```

### Synchronized Operations
Efficient batch operations:
```python
# Read same register from multiple motors
positions = bus.sync_read("Present_Position", ["joint1", "joint2"])

# Write same value to multiple motors
bus.sync_write("Goal_Position", {"joint1": 2048, "joint2": 1500})
```

### Timeout Configuration
Adjust communication timeouts:
```python
# Set custom timeout
bus.set_timeout(2000)  # 2 seconds
```

## Best Practices

### 1. Connection Management
Always properly manage connections:
```python
try:
    bus.connect()
    # ... operations ...
finally:
    bus.disconnect()
```

### 2. Calibration Usage
Ensure proper calibration before operation:
```python
if not bus.is_calibrated:
    # Perform calibration
    bus.set_half_turn_homings()
```

### 3. Error Handling
Implement robust error handling:
```python
try:
    position = bus.read("Present_Position", "joint1")
except ConnectionError:
    # Handle communication errors
    pass
```

### 4. Resource Cleanup
Use context managers for automatic cleanup:
```python
with bus.torque_disabled():
    # Safe operations
    bus.write("Homing_Offset", "joint1", 1000)
```

## Extending the Framework

### Adding New Motor Protocols
To add support for new motor protocols:

1. Create a new subclass of `MotorsBus`
2. Implement abstract methods
3. Define protocol-specific control tables
4. Handle protocol-specific encoding/decoding

### Custom Calibration
Implement custom calibration routines:
```python
class CustomMotorsBus(MotorsBus):
    def read_calibration(self) -> dict[str, MotorCalibration]:
        # Custom calibration reading
        pass
    
    def write_calibration(self, calibration_dict: dict[str, MotorCalibration]) -> None:
        # Custom calibration writing
        pass
```

## Requirements

To use the motors framework, you'll need:
- Appropriate motor SDKs (dynamixel_sdk, scservo_sdk)
- Serial communication libraries
- Python 3.7+
- Supported motor hardware

## Troubleshooting

### Common Issues
1. **Port Connection Failures**: Check USB cables and permissions
2. **Motor Not Found**: Verify baudrate and ID settings
3. **Communication Timeouts**: Adjust timeout settings
4. **Calibration Errors**: Ensure proper range recording

### Debugging Tips
1. Enable debug logging for detailed communication traces
2. Use port scanning to identify correct settings
3. Check motor firmware versions for compatibility
4. Verify power supply to motors

This motors framework provides a robust foundation for controlling various motor types in robotic applications, with support for efficient communication, comprehensive calibration, and extensible architecture.