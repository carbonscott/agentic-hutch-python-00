# Bluesky Scanning Reference

This document covers scanning operations using Bluesky with hutch-python.

## Overview

hutch-python integrates with [Bluesky](https://blueskyproject.io/) for scanning. The RunEngine (`RE`) executes plans that control devices and collect data.

## Available Plan Objects

After startup, these are available:

| Object | Description |
|--------|-------------|
| `RE` | RunEngine - executes plans |
| `bp` | bluesky.plans - standard plans |
| `bps` | bluesky.plan_stubs - plan building blocks |
| `bpp` | bluesky.preprocessors - plan modifiers |
| `re` | Namespace of ready-to-run scan functions |

## Basic Scanning

### count - Simple Counting

Count for a specified number of events:

```python
# Count 10 events
RE(count([daq], num=10))

# Count with a detector
RE(count([det], num=5))
```

### scan - Linear Motor Scan

Scan a motor from start to stop:

```python
# Syntax: scan(detectors, motor, start, stop, num_points)
RE(scan([daq], motor, 0, 10, 11))

# Scan with multiple detectors
RE(scan([daq, det1, det2], motor, -5, 5, 21))
```

### list_scan - Scan Through Positions

Scan through a list of specific positions:

```python
positions = [0, 1, 2, 5, 10, 20]
RE(list_scan([daq], motor, positions))
```

### rel_scan - Relative Scan

Scan relative to current position:

```python
# Scan from current-5 to current+5
RE(rel_scan([daq], motor, -5, 5, 11))
```

### grid_scan - 2D Scan

Scan two motors in a grid pattern:

```python
RE(grid_scan([daq],
             motor1, start1, stop1, num1,
             motor2, start2, stop2, num2))
```

### adaptive_scan - Adaptive Step Size

Automatically adjust step size based on signal:

```python
RE(adaptive_scan([det], 'det_signal', motor,
                 start, stop,
                 min_step, max_step,
                 target_delta))
```

## DAQ Integration

### Pre-configuring DAQ

Set up DAQ parameters before scanning:

```python
# Set number of events per point
daq.preconfig(events=120)

# Set recording mode
daq.preconfig(record=True)
```

### Including DAQ in Scans

Always include `daq` in the detector list:

```python
# DAQ collects data at each scan point
RE(scan([daq], motor, 0, 10, 11))

# DAQ with other detectors
RE(scan([daq, camera], motor, 0, 10, 11))
```

## Using the `re` Namespace

The `re` namespace provides pre-wrapped scan functions:

```python
# These are equivalent:
RE(scan([daq], motor, 0, 10, 11))
re.scan([daq], motor, 0, 10, 11)
```

## Plan Stubs (bps)

Building blocks for custom plans:

```python
# Move motor
bps.mv(motor, position)

# Move multiple motors
bps.mv(motor1, pos1, motor2, pos2)

# Sleep
bps.sleep(seconds)

# Trigger and read
bps.trigger_and_read([det])
```

## Custom Plans

Create custom plans using generators:

```python
from bluesky.plans import scan
from bluesky.plan_stubs import mv, sleep

def my_custom_scan(det, motor, positions):
    for pos in positions:
        yield from mv(motor, pos)
        yield from sleep(1)
        yield from trigger_and_read([det])

# Run the custom plan
RE(my_custom_scan([daq], motor, [0, 5, 10]))
```

## Scan Metadata

Add metadata to scans:

```python
RE(scan([daq], motor, 0, 10, 11),
   sample='my_sample',
   purpose='alignment')
```

## Interrupting Scans

- `Ctrl+C` once: Pause scan (can resume with `RE.resume()`)
- `Ctrl+C` twice: Abort scan
- `RE.abort()`: Abort from another terminal
- `RE.stop()`: Graceful stop

## Common Patterns

### Alignment Scan

```python
# Quick alignment scan
RE(rel_scan([daq], motor, -1, 1, 21))
```

### Energy Scan

```python
# Scan energy with DAQ
RE(scan([daq], energy_motor, 8000, 9000, 101))
```

### Time Series

```python
# Take data over time
RE(count([daq], num=100, delay=1))  # 100 points, 1 second apart
```
