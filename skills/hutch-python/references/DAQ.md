# Data Acquisition (DAQ) Reference

This document covers DAQ operations in hutch-python.

## Overview

The DAQ (Data Acquisition) system collects experimental data synchronized with beamline operations. hutch-python supports both LCLS1 and LCLS2 DAQ systems.

## DAQ Types

Configure the DAQ type in `conf.yml`:

| Type | Description |
|------|-------------|
| `lcls1` | LCLS1 DAQ system (default) |
| `lcls1-sim` | Simulated LCLS1 for testing |
| `lcls2` | LCLS2 DAQ system |
| `nodaq` | No DAQ integration |

```yaml
daq_type: lcls1
```

## Basic DAQ Operations

### Checking DAQ Status

```python
# Check current state
daq.state

# Check if connected
daq.connected
```

### Configuring DAQ

```python
# Set events per point
daq.preconfig(events=120)

# Set recording
daq.preconfig(record=True)

# Multiple settings
daq.preconfig(events=100, record=True)
```

### Manual DAQ Control

```python
# Start acquisition
daq.begin(events=100)

# Wait for completion
daq.wait()

# Stop acquisition
daq.stop()

# End run
daq.end_run()
```

## DAQ with Bluesky Scans

### Basic Integration

Include `daq` in the detector list:

```python
# DAQ collects at each scan point
RE(scan([daq], motor, 0, 10, 11))

# DAQ with other detectors
RE(scan([daq, camera], motor, 0, 10, 11))
```

### Pre-scan Configuration

Configure before running:

```python
# Configure events
daq.preconfig(events=120)

# Run scan
RE(scan([daq], motor, 0, 10, 11))
```

## LCLS1 DAQ Specifics

### Platform Selection

Set DAQ platform in configuration:

```yaml
daq_platform:
  default: 0
  specific-host: 2
```

### Run Control

```python
# Check run number
daq.run_number()

# Check if recording
daq.recording
```

## LCLS2 DAQ Specifics

### Configuration

```yaml
daq_type: lcls2
daq_host: drp-srcf-cmp001
```

### Collection Host

The collection host must be specified for LCLS2:

```python
# In conf.yml
daq_host: drp-srcf-cmp001
```

## Simulated DAQ

For testing without hardware:

```bash
# Launch with simulated DAQ
xxxpython --sim
```

Or in configuration:

```yaml
daq_type: lcls1-sim
```

The simulated DAQ:
- Mimics real DAQ behavior
- Useful for testing scripts
- Does not collect real data

## Common DAQ Patterns

### Standard Data Collection

```python
# Configure for experiment
daq.preconfig(events=120, record=True)

# Run motor scan with data collection
RE(scan([daq], motor, start, stop, num_points))
```

### Quick Test (No Recording)

```python
daq.preconfig(events=10, record=False)
RE(scan([daq], motor, 0, 1, 5))
```

### Long Acquisition

```python
# Collect many events at fixed position
daq.preconfig(events=10000, record=True)
RE(count([daq], num=1))
```

## Troubleshooting

### DAQ Not Connected

```python
# Check connection
daq.connected

# Try reconnecting
daq.connect()
```

### Events Not Collected

- Verify `events` parameter is set
- Check beam is on
- Verify detector configuration

### Recording Issues

- Ensure `record=True` in preconfig
- Check disk space
- Verify run database access

## DAQ States

| State | Description |
|-------|-------------|
| `Disconnected` | Not connected to DAQ |
| `Connected` | Connected, ready |
| `Configured` | Configured for acquisition |
| `Running` | Currently acquiring |
| `Open` | Run is open |

## Best Practices

1. Always `preconfig` before important scans
2. Use `record=False` for alignment scans
3. Check `daq.state` if scans hang
4. Use simulated DAQ for script testing
