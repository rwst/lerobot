# LeRobot Kinematics Module

This document provides an overview of the kinematics module implemented in the LeRobot library, which provides robot kinematics functionality using the placo library.

## Overview

The LeRobot kinematics module provides a standardized interface for computing forward and inverse kinematics for robotic systems using the placo library. This module enables robots to calculate end-effector poses from joint positions (forward kinematics) and determine joint positions needed to achieve desired end-effector poses (inverse kinematics).

## Core Components

### RobotKinematics Class

The `RobotKinematics` class is the main interface for robot kinematics operations using the placo library:

#### Initialization

```python
from lerobot.model.kinematics import RobotKinematics

# Initialize kinematics solver with URDF file
kinematics = RobotKinematics(
    urdf_path="path/to/robot.urdf",
    target_frame_name="gripper_frame_link",  # End-effector frame name
    joint_names=["joint1", "joint2", "joint3"]  # Optional: specific joint names
)
```

#### Constructor Parameters:
- `urdf_path`: Path to the robot URDF file describing the robot's structure
- `target_frame_name`: Name of the end-effector frame in the URDF (default: "gripper_frame_link")
- `joint_names`: List of joint names to use for the kinematics solver (optional, defaults to all joints in URDF)

### Forward Kinematics

Compute the end-effector pose from joint positions:

```python
# Compute forward kinematics
joint_positions = [0, 45, 90]  # Joint positions in degrees
ee_pose = kinematics.forward_kinematics(joint_positions)
# Returns 4x4 transformation matrix representing end-effector pose
```

#### Parameters:
- `joint_pos_deg`: Joint positions in degrees (numpy array)

#### Returns:
- 4x4 transformation matrix representing the end-effector pose in world coordinates

### Inverse Kinematics

Compute joint positions needed to achieve a desired end-effector pose:

```python
# Compute inverse kinematics
current_joints = [0, 45, 90]  # Current joint positions in degrees
desired_pose = np.eye(4)  # Desired end-effector pose as 4x4 matrix
solution = kinematics.inverse_kinematics(
    current_joints, 
    desired_pose,
    position_weight=1.0,
    orientation_weight=0.01
)
# Returns joint positions in degrees
```

#### Parameters:
- `current_joint_pos`: Current joint positions in degrees (used as initial guess)
- `desired_ee_pose`: Target end-effector pose as a 4x4 transformation matrix
- `position_weight`: Weight for position constraint in IK (default: 1.0)
- `orientation_weight`: Weight for orientation constraint in IK (default: 0.01, set to 0.0 for position-only)

#### Returns:
- Joint positions in degrees that achieve the desired end-effector pose

## Key Features

### URDF-Based Kinematics

The module uses URDF (Unified Robot Description Format) files to define robot structure:
- Joint hierarchies and transformations
- Link geometries and inertial properties
- End-effector frame definitions

### Placo Integration

Leverages the placo library for efficient kinematics computation:
- Forward kinematics using transformation matrices
- Inverse kinematics using optimization-based solvers
- Constraint-based kinematics solving

### Flexible Frame Specification

Support for custom end-effector frames:
- Specify any frame defined in the URDF
- Default gripper frame provided
- Support for multiple end-effectors

### Weighted Constraints

Configurable constraint weights for inverse kinematics:
- Position-weighted IK for precise positioning
- Orientation-weighted IK for precise orientation
- Combined position and orientation constraints

### Unit Conversion

Automatic conversion between degrees and radians:
- Input joint positions in degrees
- Internal computation in radians
- Output joint positions in degrees

### Gripper Preservation

Preserves gripper positions in joint solutions:
- Maintains gripper state when present in input
- Handles mixed joint configurations (arm + gripper)

## Installation Requirements

The kinematics module requires the placo library:

```bash
# Install placo library (may require additional dependencies)
pip install placo
```

If the placo library is not installed, an ImportError will be raised with instructions.

## Error Handling

### ImportError

Raised when placo library is not available:
```python
ImportError: placo is required for RobotKinematics. 
Please install the optional dependencies of `kinematics` in the package.
```

### Kinematics Solver Errors

Potential errors from placo library:
- Invalid URDF file paths
- Undefined frame names
- Kinematic singularities
- Solver convergence issues

## Best Practices

### 1. URDF Management

Ensure URDF files are properly formatted and accessible:
```python
# Verify URDF file exists
import os
assert os.path.exists("path/to/robot.urdf"), "URDF file not found"
```

### 2. Frame Naming

Use consistent frame names defined in the URDF:
```python
# Check available frames
kinematics = RobotKinematics(urdf_path)
# Verify frame exists in URDF
```

### 3. Initial Guesses

Provide good initial guesses for inverse kinematics:
```python
# Use current joint positions as initial guess
current_joints = robot.get_joint_positions()
solution = kinematics.inverse_kinematics(current_joints, target_pose)
```

### 4. Constraint Weights

Tune constraint weights for specific applications:
```python
# Position-only IK (ignore orientation)
solution = kinematics.inverse_kinematics(
    current_joints, target_pose,
    position_weight=1.0,
    orientation_weight=0.0
)

# Balanced position/orientation IK
solution = kinematics.inverse_kinematics(
    current_joints, target_pose,
    position_weight=1.0,
    orientation_weight=0.1
)
```

### 5. Solution Validation

Validate kinematics solutions:
```python
# Check if solution achieves target pose
achieved_pose = kinematics.forward_kinematics(solution)
# Compare with desired_pose
```

## Usage Examples

### Basic Forward Kinematics

```python
import numpy as np
from lerobot.model.kinematics import RobotKinematics

# Initialize kinematics
kinematics = RobotKinematics("robot.urdf")

# Compute end-effector pose
joint_angles = np.array([0, 45, 90])
ee_pose = kinematics.forward_kinematics(joint_angles)
print(f"End-effector pose:\n{ee_pose}")
```

### Basic Inverse Kinematics

```python
import numpy as np
from lerobot.model.kinematics import RobotKinematics

# Initialize kinematics
kinematics = RobotKinematics("robot.urdf")

# Define target pose
target_pose = np.eye(4)
target_pose[:3, 3] = [0.5, 0.0, 0.3]  # Position

# Compute joint solution
current_joints = np.array([0, 0, 0])
solution = kinematics.inverse_kinematics(current_joints, target_pose)
print(f"Joint solution: {solution}")
```

### Position-Only Inverse Kinematics

```python
# Compute position-only IK
solution = kinematics.inverse_kinematics(
    current_joints, target_pose,
    position_weight=1.0,
    orientation_weight=0.0  # Ignore orientation
)
```

## Extending the Framework

### Custom Kinematics Solvers

Create custom kinematics implementations:

```python
class CustomKinematics(RobotKinematics):
    def __init__(self, urdf_path, custom_parameter=None):
        super().__init__(urdf_path)
        self.custom_parameter = custom_parameter
    
    def custom_kinematics_method(self, joints):
        # Custom kinematics implementation
        pass
```

### Additional Frame Tasks

Add additional kinematics constraints:

```python
class ExtendedKinematics(RobotKinematics):
    def __init__(self, urdf_path):
        super().__init__(urdf_path)
        # Add additional frame tasks
        self.additional_frame = self.solver.add_frame_task("other_frame", np.eye(4))
```

## Troubleshooting

### Common Issues

1. **URDF File Not Found**
   - Verify file path is correct
   - Check file permissions
   - Ensure URDF is properly formatted

2. **Frame Not Defined**
   - Verify frame name exists in URDF
   - Check for typos in frame names
   - Ensure URDF includes all required links

3. **Solver Convergence**
   - Improve initial guess
   - Adjust constraint weights
   - Check for kinematic singularities

4. **Import Errors**
   - Install placo library
   - Check Python environment
   - Verify dependencies

### Debugging Tips

1. Print available frames:
```python
print(kinematics.robot.frame_names())
```

2. Check joint names:
```python
print(kinematics.joint_names)
```

3. Validate transformation matrices:
```python
pose = kinematics.forward_kinematics(joints)
assert pose.shape == (4, 4), "Invalid pose shape"
```

This kinematics module provides a robust foundation for robot kinematics computations in the LeRobot library, enabling precise control of robotic systems through forward and inverse kinematics operations.