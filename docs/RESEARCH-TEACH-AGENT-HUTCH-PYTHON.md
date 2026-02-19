# Research: Teaching AI Agents to Use hutch-python

This document provides comprehensive research on how to teach an AI agent to use hutch-python, combining knowledge of the agent skill format with detailed hutch-python functionality.

---

## Executive Summary

This research covers two key areas:

1. **Agent Skills System** - A standardized format for teaching AI agents new capabilities through `SKILL.md` files with optional scripts and references.

2. **hutch-python** - The launcher and configuration system for LCLS (Linac Coherent Light Source) interactive IPython sessions, used for beamline control at SLAC.

By combining these systems, we can create agent skills that teach AI assistants how to help users operate LCLS beamlines through hutch-python.

---

## Part 1: Agent Skills Format

### 1.1 Overview

A skill is a directory containing a `SKILL.md` file that provides metadata and instructions for an AI agent. The format uses progressive disclosure to minimize context usage while maximizing capability.

**Directory Structure:**
```
my-skill/
├── SKILL.md              # Required: metadata + instructions
├── scripts/              # Optional: executable code
├── references/           # Optional: detailed documentation
└── assets/               # Optional: templates and resources
```

### 1.2 SKILL.md Structure

Every skill starts with a YAML frontmatter followed by Markdown instructions.

**Required Fields:**
```yaml
---
name: skill-name
description: A description of what this skill does and when to use it.
---
```

**Optional Fields:**
```yaml
---
name: hutch-python-operations
description: Help users operate LCLS beamlines using hutch-python.
license: Apache-2.0
compatibility: Requires hutch-python environment and LCLS access
metadata:
  author: SLAC
  version: "1.0"
allowed-tools: Bash(xxxpython:*) Read
---
```

**Field Constraints:**

| Field | Required | Constraints |
|-------|----------|-------------|
| `name` | Yes | 1-64 characters, lowercase with hyphens only, no leading/trailing hyphens |
| `description` | Yes | 1-1024 characters, describes what and when |
| `license` | No | License name or reference |
| `compatibility` | No | Max 500 characters, environment requirements |
| `metadata` | No | Arbitrary key-value mapping |
| `allowed-tools` | No | Space-delimited list of pre-approved tools |

### 1.3 Progressive Disclosure Architecture

Skills use three-level context management:

1. **Discovery Phase (~100 tokens)** - Only `name` and `description` loaded at startup
2. **Activation Phase (<5000 tokens recommended)** - Full SKILL.md loaded when task matches
3. **Execution Phase (on-demand)** - Scripts and references loaded only when needed

**Recommendation:** Keep main instructions under 500 lines; move detailed content to `references/`.

### 1.4 Optional Directories

**scripts/** - Executable code (Python, Bash, JavaScript):
- Should be self-contained or clearly document dependencies
- Include helpful error messages

**references/** - On-demand documentation:
- `REFERENCE.md` - Detailed technical reference
- Domain-specific files (e.g., `scanning.md`, `daq.md`)

**assets/** - Static resources:
- Templates, configuration examples
- Lookup tables, schemas

### 1.5 Integration with Agents

Skills are integrated via XML format in agent prompts:

```xml
<available_skills>
  <skill>
    <name>hutch-python-operations</name>
    <description>Help users operate LCLS beamlines using hutch-python.</description>
    <location>/path/to/skills/hutch-python-operations/SKILL.md</location>
  </skill>
</available_skills>
```

### 1.6 Validation

Use the `skills-ref` CLI to validate skills:

```bash
# Validate a skill directory
skills-ref validate path/to/skill

# Generate XML for agent prompts
skills-ref to-prompt path/to/skill-a path/to/skill-b
```

---

## Part 2: hutch-python Overview

### 2.1 Purpose and Architecture

**hutch-python** is the launcher and configuration system for LCLS interactive IPython sessions. It provides:

- Automatic loading and configuration of beamline control hardware
- Configured IPython environment for interactive beamline operation
- Experiment-specific code and parameter management
- Integration with DAQ (Data Acquisition) systems
- Device organization from the happi database

### 2.2 Configuration System (conf.yml)

Configuration is defined in YAML format:

```yaml
hutch: xpp                    # Hutch name (e.g., xpp, mfx, cxi)
db: /path/to/happi_db.json    # Path to happi device database
load: xpp.beamline            # Modules to import
load_level: STANDARD          # Device loading depth
daq_type: lcls1               # DAQ type (lcls1, lcls2, nodaq)
```

**Complete Configuration Options:**

| Key | Type | Purpose |
|-----|------|---------|
| `hutch` | string | Hutch name - used for device filtering, banner, DAQ setup |
| `db` | string | Path to happi database (JSON file) |
| `load` | string/list | Modules to import (e.g., "xpp.beamline") |
| `load_level` | string | Device loading depth: UPSTREAM, STANDARD, or ALL |
| `experiment` | dict | Override experiment: {proposal: str, run: str} |
| `obj_config` | string | YAML file for object customization |
| `daq_type` | string | 'lcls1', 'lcls1-sim', 'lcls2', or 'nodaq' |
| `daq_host` | string | Collection host for LCLS2 DAQ |
| `daq_platform` | dict | Platform selection {default: int, hostname: int} |
| `exclude_devices` | list | Device names to skip loading |
| `additional_devices` | dict | Additional devices with search criteria |

### 2.3 Key Modules

| Module | Purpose |
|--------|---------|
| `cli.py` | Command-line interface (--cfg, --exp, --debug, --sim) |
| `load_conf.py` | Main configuration loader and startup orchestration |
| `happi.py` | Device loading from happi database |
| `namespace.py` | Groups objects into namespaces by type |
| `plan_wrappers.py` | Bluesky plan wrapping for scans |
| `exp_load.py` | Experiment-specific code loading |
| `qs_load.py` | Questionnaire object loading |

### 2.4 Namespaces and Object Access

After startup, these namespaces are available:

| Namespace | Alias | Description |
|-----------|-------|-------------|
| `motors` | `m` | All EpicsMotor objects |
| `slits` | `s` | All Slits objects |
| `all_objects` | `a` | Everything loaded |
| `user` | `x` | User/experiment object |
| `sim` | - | Simulated hardware |
| `RE` | - | RunEngine for bluesky plans |
| `daq` | - | Data acquisition control |
| `bp`, `bps`, `bpp` | - | Bluesky plans, plan specs, preprocessors |
| `re` | - | Ready-to-run scan functions |
| `{hutch}_beampath` | - | Lightpath beamline status |

### 2.5 Entry Points

**Main Command:**
```bash
xxxpython                     # Launch interactive session (xxx = hutch name)
xxxpython --cfg /path/conf.yml   # Custom configuration
xxxpython --exp expname          # Override experiment
xxxpython --sim                  # Simulated DAQ
xxxpython --debug                # Debug mode
xxxpython script.py              # Run script instead of interactive
```

---

## Part 3: Key hutch-python Concepts for Agent Training

### 3.1 Startup Sequence

The startup sequence is critical for understanding what's available:

1. Initialize system (logging, debug mode, simulation mode)
2. Read conf.yml configuration
3. Create virtual module (`{hutch}.db`)
4. Setup RunEngine (`RE`) for bluesky plans
5. Create DAQ object (LCLS1, LCLS2, or simulated)
6. Load bluesky defaults (`bp`, `bps`, `bpp`, `re`)
7. Load database devices from happi
8. Create lightpath (`{hutch}_beampath`)
9. Load cameras from camviewer
10. Load beamline modules (`from xxx.beamline import *`)
11. Select active experiment
12. Load questionnaire objects
13. Load experiment code (User class)
14. Create device groups (motors, slits, etc.)
15. Setup debugging objects
16. Launch IPython session

### 3.2 Device Loading (happi Database)

Devices are loaded from a happi database based on `load_level`:

- **UPSTREAM** - Devices upstream of the hutch
- **STANDARD** - Standard hutch devices (default)
- **ALL** - All available devices

```python
# Access devices by name
from xpp.db import my_motor

# Access by namespace
motors.my_motor
m.my_motor

# List all motors
list(motors)
```

### 3.3 Bluesky Integration and Scanning

hutch-python integrates with Bluesky for scanning:

```python
# Simple count
RE(count([daq]))

# Motor scan
RE(scan([daq], motor, start, stop, num_points))

# Using the re namespace (auto-wrapped)
re.scan([daq], motor, 0, 10, 11)

# Available plan sets
bp    # bluesky.plans
bps   # bluesky.plan_stubs
bpp   # bluesky.preprocessors
```

**Common Scan Types:**
- `scan` - Linear motor scan
- `count` - Simple counting
- `list_scan` - Scan through a list of positions
- `adaptive_scan` - Adaptive step sizing

### 3.4 DAQ Operations

Data Acquisition system control:

```python
# Check DAQ status
daq.state

# Configure DAQ
daq.preconfig(events=120)

# Run DAQ with Bluesky
RE(scan([daq], motor, 0, 10, 11))

# DAQ types available
# lcls1     - LCLS1 DAQ (default)
# lcls1-sim - Simulated LCLS1
# lcls2     - LCLS2 DAQ
# nodaq     - No DAQ integration
```

### 3.5 Motor Presets

Position presets for motors:

```python
# Move to preset position
motor.mv_tt()           # Move to 'tt' preset
motor.mv_out()          # Move to 'out' preset

# Add hutch preset (permanent)
motor.presets.add_hutch('sample', 42.1)

# Add experiment preset (temporary)
motor.presets.add_exp('sample', 42.1)

# List available presets
motor.presets.positions
```

**Preset File Location:** `/reg/g/pcds/pyps/apps/hutch-python/[hutch]/presets/`

### 3.6 Experiment Context

Experiments are identified by proposal and run numbers:

```python
# Current experiment info
user                    # or x - experiment object

# Override experiment
xxxpython --exp tstx010

# In conf.yml
experiment:
  proposal: ls2516
  run: 1
```

Experiment code lives in: `experiments/{expname}.py`

### 3.7 Lightpath

Beamline visualization and device status:

```python
# Access beampath
xpp_beampath            # Lightpath for xpp hutch

# Check device states
xpp_beampath.show()     # Display beamline status
```

---

## Part 4: Proposed Agent Skill for hutch-python

### 4.1 Recommended SKILL.md

```yaml
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

## Overview
hutch-python is the launcher for LCLS interactive IPython sessions. It provides access to beamline devices, scanning capabilities, and data acquisition.

## When to Use This Skill
- Users ask about LCLS beamline operations
- Questions about motor control, scanning, or DAQ
- Help with hutch-python configuration
- Device access and namespace navigation

## Quick Reference

### Launching Sessions
```bash
xxxpython                 # Interactive session (xxx = hutch)
xxxpython --cfg conf.yml  # Custom config
xxxpython --exp expname   # Override experiment
xxxpython --sim           # Simulated DAQ
```

### Essential Namespaces
| Namespace | Alias | Content |
|-----------|-------|---------|
| motors | m | All motors |
| slits | s | All slits |
| all_objects | a | Everything |
| user | x | Experiment object |

### Basic Scanning
```python
# Simple scan
RE(scan([daq], motor, start, stop, num_points))

# Count
RE(count([daq], num=10))
```

### Motor Operations
```python
# Move motor
motor.mv(position)
motor.mvr(relative_position)

# Move to preset
motor.mv_sample()
```

## For Detailed Information
See the reference files in the `references/` directory:
- [CONFIGURATION.md](references/CONFIGURATION.md) - conf.yml options
- [SCANNING.md](references/SCANNING.md) - Bluesky scanning details
- [DAQ.md](references/DAQ.md) - Data acquisition operations
- [NAMESPACES.md](references/NAMESPACES.md) - Object organization
```

### 4.2 Recommended References Directory

Create detailed reference files:

**references/CONFIGURATION.md** - Complete conf.yml documentation
**references/SCANNING.md** - Bluesky plans and scan types
**references/DAQ.md** - DAQ configuration and usage
**references/NAMESPACES.md** - Device access patterns
**references/PRESETS.md** - Motor preset system
**references/TROUBLESHOOTING.md** - Common issues and solutions

### 4.3 Recommended Scripts

**scripts/check_devices.py** - List available devices
**scripts/scan_helper.py** - Generate scan commands
**scripts/validate_config.py** - Validate conf.yml

---

## Part 5: Teaching Strategy

### 5.1 Priority Concepts (High to Low)

1. **Essential** - Must understand to be useful
   - Launching sessions (`xxxpython` command)
   - Namespace access (motors, slits, all_objects)
   - Basic motor operations (mv, mvr)
   - Simple scans with RunEngine

2. **Important** - Common operations
   - conf.yml structure
   - DAQ integration
   - Preset system
   - Device groups

3. **Advanced** - Power user features
   - Custom beamline modules
   - Experiment code
   - Lightpath
   - Debug mode

### 5.2 Common Workflows to Teach

1. **Start a Session**
   ```bash
   xxxpython --cfg /path/to/conf.yml
   ```

2. **Find and Move a Motor**
   ```python
   list(motors)          # See available motors
   my_motor.position     # Check current position
   my_motor.mv(10)       # Move to position
   ```

3. **Run a Scan**
   ```python
   daq.preconfig(events=100)
   RE(scan([daq], motor, 0, 10, 11))
   ```

4. **Save a Position**
   ```python
   motor.presets.add_hutch('reference', motor.position)
   ```

### 5.3 Error Handling Patterns

Teach agents to recognize and respond to:

- **Import errors** - Missing modules or devices
- **Connection errors** - Device communication issues
- **DAQ errors** - Acquisition system problems
- **Permission errors** - Access control issues

### 5.4 Key Files for Agent Reference

| File | Purpose |
|------|---------|
| `load_conf.py` | Startup orchestration |
| `happi.py` | Device database access |
| `utils.py` | Namespace utilities |
| `plan_wrappers.py` | Scan automation |
| `yaml_files.rst` | Configuration reference |
| `tutorial.ipynb` | Interactive examples |

---

## Appendix A: File Locations

**Agent Skills Repository:**
```
/sdf/data/lcls/ds/prj/prjdat21/results/cwang31/resources/agentskills/
├── docs/specification.mdx     # Full skill format specification
├── docs/what-are-skills.mdx   # Introduction to skills
├── docs/integrate-skills.mdx  # Integration guide
└── skills-ref/                # Reference implementation
```

**hutch-python Repository:**
```
/sdf/data/lcls/ds/prj/prjdat21/results/cwang31/proj-hutch-python/externals/hutch-python/
├── hutch_python/              # Source code
│   ├── cli.py                 # Command-line interface
│   ├── load_conf.py           # Configuration loader
│   ├── happi.py               # Device database
│   └── ...
└── docs/source/               # Documentation
    ├── yaml_files.rst         # Configuration reference
    ├── tutorial.ipynb         # Interactive tutorial
    └── ...
```

---

## Appendix B: Example Minimal Skill

The simplest valid skill for hutch-python:

```
hutch-python-basic/
└── SKILL.md
```

**SKILL.md:**
```yaml
---
name: hutch-python-basic
description: Basic hutch-python operations for LCLS beamline control. Use when users need help with motors, scanning, or data acquisition.
---

# hutch-python Basic Operations

## Launching
```bash
xxxpython  # Replace xxx with hutch name (xpp, mfx, cxi, etc.)
```

## Key Namespaces
- `motors` (or `m`) - All motor objects
- `slits` (or `s`) - All slit objects
- `all_objects` (or `a`) - Everything loaded

## Basic Scan
```python
RE(scan([daq], motor, start, stop, num_points))
```
```

---

## Appendix C: conf.yml Template

```yaml
# Minimal hutch-python configuration
hutch: xpp
db: /reg/g/pcds/pyps/apps/hutch-python/device_config/db.json
load: xpp.beamline
load_level: STANDARD
daq_type: lcls1

# Optional: exclude problematic devices
exclude_devices:
  - broken_motor

# Optional: experiment override
experiment:
  proposal: ls2516
  run: 1
```

---

*Research completed: 2026-01-12*
*Sources: agentskills documentation, hutch-python source code and documentation*
