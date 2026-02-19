# Namespaces and Device Access Reference

This document covers how devices are organized and accessed in hutch-python.

## Overview

hutch-python organizes devices into namespaces for easy access. Each namespace groups related devices together.

## Available Namespaces

After startup, these namespaces are available:

| Namespace | Alias | Description |
|-----------|-------|-------------|
| `motors` | `m` | All EpicsMotor objects |
| `slits` | `s` | All Slits objects |
| `all_objects` | `a` | Everything loaded |
| `user` | `x` | User/experiment object |
| `sim` | - | Simulated hardware |
| `RE` | - | RunEngine |
| `daq` | - | Data acquisition |

## Accessing Devices

### By Namespace

```python
# Access motor by namespace
motors.my_motor
m.my_motor  # Using alias

# Access slit
slits.my_slit
s.my_slit

# Access any object
all_objects.my_device
a.my_device
```

### By Database Module

Each hutch has a virtual `{hutch}.db` module:

```python
# Import specific device
from xpp.db import my_motor

# Import multiple
from xpp.db import motor1, motor2, detector
```

### Direct Name

Top-level devices are available directly:

```python
# If loaded, available by name
my_motor.position
```

## Listing Devices

### List All in Namespace

```python
# Get list of device names
list(motors)

# Print formatted table
motors  # In IPython, displays nice table
```

### Check Device Count

```python
len(motors)
len(all_objects)
```

### Iterate Over Devices

```python
# Loop through all motors
for motor in motors:
    print(motor.name, motor.position)
```

## HelpfulNamespace Features

Namespaces use `HelpfulNamespace` class which provides:

- Tab completion
- Formatted table display
- Iteration support
- Length checking
- List conversion

```python
# Table display
motors  # Shows name, prefix, position

# Convert to list
motor_list = list(motors)

# Tab completion
motors.<TAB>  # Shows all available motors
```

## Device Grouping

### By Class Type

Devices are automatically grouped by class:

```python
motors      # All EpicsMotor
slits       # All Slits
```

### By Metadata

Devices with `group` metadata are grouped:

```python
# If devices have group='sample_stack' metadata
sample_stack.motor1
sample_stack.motor2
```

## The Database Module ({hutch}.db)

The virtual `{hutch}.db` module provides database access:

```python
# Import from database
from xpp.db import some_device

# This loads the device from happi database
# Useful for devices not in main namespace
```

### Load Level

The `load_level` configuration controls what's loaded:

```yaml
load_level: STANDARD
```

| Level | Description |
|-------|-------------|
| `UPSTREAM` | Only upstream devices |
| `STANDARD` | Standard hutch devices |
| `ALL` | All available devices |

## User Namespace

The `user` (alias `x`) namespace contains experiment-specific objects:

```python
# Access user object
user
x  # Same thing

# User attributes depend on experiment code
user.sample_x
x.custom_method()
```

## Simulated Devices

The `sim` namespace contains simulated hardware:

```python
# Access simulated devices
sim.motor
sim.detector
```

Useful for testing without real hardware.

## Finding Devices

### Search by Name Pattern

```python
# Find motors matching pattern
[m for m in motors if 'sample' in m.name]
```

### Search by Attribute

```python
# Find motors at specific position
[m for m in motors if m.position > 0]
```

### Using class_namespace

Create custom groupings:

```python
from hutch_python.utils import class_namespace

# Group all motors
my_motors = class_namespace(EpicsMotor)

# Group specific class
my_detectors = class_namespace(AreaDetector)
```

## Best Practices

1. Use aliases (`m`, `s`, `a`, `x`) for quick access
2. Use `{hutch}.db` for explicit imports
3. Check `list(namespace)` to see available devices
4. Use tab completion to explore namespaces
5. Access specific devices by name when known
