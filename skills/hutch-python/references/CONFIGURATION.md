# hutch-python Configuration Reference

This document provides complete documentation for the `conf.yml` configuration file.

## Configuration File Location

The configuration file is typically located at:
- `/reg/g/pcds/pyps/apps/hutch-python/{hutch}/conf.yml`

Or specified via command line:
```bash
xxxpython --cfg /path/to/conf.yml
```

## Complete Configuration Options

### hutch (required)

The hutch name. Used for device filtering, banner display, and DAQ setup.

```yaml
hutch: xpp
```

Valid values: `xpp`, `xcs`, `mfx`, `cxi`, `mec`, `tmo`, `rix`, etc.

### db (required)

Path to the happi device database (JSON file).

```yaml
db: /reg/g/pcds/pyps/apps/hutch-python/device_config/db.json
```

### load

Modules to import at startup. Can be a string or list.

```yaml
# Single module
load: xpp.beamline

# Multiple modules
load:
  - xpp.beamline
  - xpp.experiments
```

### load_level

Controls which devices are loaded from the happi database.

```yaml
load_level: STANDARD
```

| Level | Description |
|-------|-------------|
| `UPSTREAM` | Only devices upstream of the hutch |
| `STANDARD` | Standard hutch devices (default) |
| `ALL` | All available devices |

### experiment

Override the active experiment (normally auto-detected).

```yaml
experiment:
  proposal: ls2516
  run: 1
```

Or use command line:
```bash
xxxpython --exp ls2516
```

### obj_config

Path to object configuration YAML file for customizing tab completion and Kind attributes.

```yaml
obj_config: /path/to/obj_config.yml
```

### daq_type

Type of DAQ (Data Acquisition) system to use.

```yaml
daq_type: lcls1
```

| Type | Description |
|------|-------------|
| `lcls1` | LCLS1 DAQ (default) |
| `lcls1-sim` | Simulated LCLS1 DAQ |
| `lcls2` | LCLS2 DAQ |
| `nodaq` | No DAQ integration |

### daq_host

Collection host for LCLS2 DAQ.

```yaml
daq_host: drp-srcf-cmp001
```

### daq_platform

Platform selection for DAQ. Can specify default and per-hostname values.

```yaml
daq_platform:
  default: 0
  specific-hostname: 2
```

### exclude_devices

List of device names to skip when loading from the database.

```yaml
exclude_devices:
  - broken_motor
  - offline_detector
```

### additional_devices

Additional devices to load from other beamlines using search criteria.

```yaml
additional_devices:
  tmo_devices:
    beamline: TMO
    device_class: pcdsdevices.sqr1.SQR1
```

## Example Configurations

### Minimal Configuration

```yaml
hutch: xpp
db: /reg/g/pcds/pyps/apps/hutch-python/device_config/db.json
load: xpp.beamline
```

### Full Configuration

```yaml
hutch: xpp
db: /reg/g/pcds/pyps/apps/hutch-python/device_config/db.json
load:
  - xpp.beamline
  - xpp.custom_devices
load_level: STANDARD
daq_type: lcls1
daq_platform:
  default: 0

experiment:
  proposal: ls2516
  run: 1

obj_config: /reg/g/pcds/pyps/apps/hutch-python/xpp/obj_config.yml

exclude_devices:
  - problematic_motor

additional_devices:
  fee_devices:
    beamline: FEE
```

### Development/Testing Configuration

```yaml
hutch: tst
db: /path/to/test_db.json
load: tst.beamline
daq_type: lcls1-sim
load_level: ALL
```

## Object Configuration File

The `obj_config` file allows customizing device behavior:

```yaml
# Tab completion whitelist
tab_whitelist:
  Motor:
    - position
    - move
    - stop

# Tab completion blacklist
tab_blacklist:
  Signal:
    - _metadata

# Modify ophyd Kind attributes
kind_config:
  MyDevice:
    important_signal: hinted
    internal_signal: omitted
```
