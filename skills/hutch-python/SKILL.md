---
name: hutch-python
description: Help users operate LCLS beamlines using hutch-python. Use when users ask about motor control, scanning, DAQ operations, device configuration, or beamline setup at SLAC LCLS.
license: BSD-3-Clause
compatibility: Requires LCLS environment with hutch-python installed
metadata:
  author: SLAC National Accelerator Laboratory
  version: "1.0"
  domain: scientific-instrumentation
---

# hutch-python Operations

hutch-python is the launcher and configuration system for LCLS (Linac Coherent Light Source) interactive IPython sessions. It provides access to beamline devices, scanning capabilities, and data acquisition.

## When to Use This Skill

- Users ask about LCLS beamline operations
- Questions about motor control, scanning, or DAQ
- Help with hutch-python configuration
- Device access and namespace navigation
- Running scans with Bluesky

## Launching Sessions

```bash
xxxpython                 # Interactive session (xxx = hutch name like xpp, mfx, cxi)
xxxpython --cfg conf.yml  # Use custom configuration file
xxxpython --exp expname   # Override the active experiment
xxxpython --sim           # Use simulated DAQ
xxxpython --debug         # Enable debug mode
xxxpython script.py       # Run a script instead of interactive mode
```

## Essential Namespaces

After launching, these namespaces provide access to devices:

| Namespace | Alias | Description |
|-----------|-------|-------------|
| `motors` | `m` | All EpicsMotor objects |
| `slits` | `s` | All Slits objects |
| `all_objects` | `a` | Everything loaded |
| `user` | `x` | User/experiment object |
| `RE` | - | RunEngine for bluesky plans |
| `daq` | - | Data acquisition control |

## Basic Operations

### Moving Motors

```python
# Check motor position
motor.position

# Move to absolute position
motor.mv(10)

# Move relative
motor.mvr(0.5)

# Move to preset position
motor.mv_sample()
```

### Running Scans

```python
# Simple count
RE(count([daq], num=10))

# Motor scan
RE(scan([daq], motor, start, stop, num_points))

# Example: scan motor from 0 to 10 in 11 steps
RE(scan([daq], my_motor, 0, 10, 11))
```

### DAQ Operations

```python
# Configure DAQ
daq.preconfig(events=120)

# Check DAQ state
daq.state
```

### Listing Devices

```python
# List all motors
list(motors)

# Print formatted table
motors

# Access specific device
motors.my_motor
m.my_motor
```

## Configuration

hutch-python is configured via `conf.yml`. Key options:

```yaml
hutch: xpp              # Hutch name
db: /path/to/db.json    # happi database path
load: xpp.beamline      # Modules to import
load_level: STANDARD    # Device loading depth
daq_type: lcls1         # DAQ type
```

## Detailed References

For more detailed information, see:

- [CONFIGURATION.md](references/CONFIGURATION.md) - Complete conf.yml options
- [SCANNING.md](references/SCANNING.md) - Bluesky scanning guide
- [DAQ.md](references/DAQ.md) - Data acquisition operations
- [NAMESPACES.md](references/NAMESPACES.md) - Device organization and access
- [PRESETS.md](references/PRESETS.md) - Motor preset system
