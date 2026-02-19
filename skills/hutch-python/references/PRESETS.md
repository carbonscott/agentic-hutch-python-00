# Motor Presets Reference

This document covers the motor preset system in hutch-python.

## Overview

Presets save named motor positions for easy recall. There are two types:
- **Hutch presets**: Permanent positions shared across experiments
- **Experiment presets**: Temporary positions for the current experiment

## Using Presets

### Moving to a Preset

```python
# Move to preset position
motor.mv_sample()       # Move to 'sample' preset
motor.mv_out()          # Move to 'out' preset
motor.mv_in()           # Move to 'in' preset

# The method name is mv_<preset_name>
```

### Listing Available Presets

```python
# See all presets for a motor
motor.presets.positions

# Check specific preset value
motor.presets.positions['sample']
```

## Creating Presets

### Hutch Presets (Permanent)

Hutch presets persist across experiments and are shared:

```python
# Add hutch preset at current position
motor.presets.add_hutch('sample', motor.position)

# Add hutch preset at specific position
motor.presets.add_hutch('reference', 42.5)

# Add with comment
motor.presets.add_hutch('sample', 10.0, comment='Sample center position')
```

### Experiment Presets (Temporary)

Experiment presets are specific to the current experiment:

```python
# Add experiment preset at current position
motor.presets.add_exp('my_sample', motor.position)

# Add with specific value
motor.presets.add_exp('edge', 25.3)
```

### Quick Add at Current Position

```python
# Add hutch preset at current position (shortcut)
motor.presets.add_hutch_here('sample')

# Add experiment preset at current position
motor.presets.add_exp_here('my_position')
```

## Removing Presets

```python
# Remove hutch preset
motor.presets.remove_hutch('old_preset')

# Remove experiment preset
motor.presets.remove_exp('temp_preset')
```

## Preset File Locations

### Hutch Presets

Stored in:
```
/reg/g/pcds/pyps/apps/hutch-python/{hutch}/presets/
```

### Experiment Presets

Stored in experiment directory, linked to current experiment.

## Preset File Format

Presets are stored in YAML format:

```yaml
# presets/motor_name.yml
sample:
  value: 42.5
  comment: Sample center position
out:
  value: -100.0
  comment: Motor retracted
in:
  value: 0.0
```

## Preset Naming Conventions

- Use lowercase names
- Use underscores for spaces: `sample_center`
- Common names:
  - `in` / `out` - Insert/retract positions
  - `sample` - Sample position
  - `tt` - Target transfer position
  - `reference` - Reference position

## Multiple Motors

### Moving Multiple to Presets

```python
# Move several motors to their presets
motor1.mv_sample()
motor2.mv_sample()
motor3.mv_sample()
```

### Coordinated Presets

For coordinated motion, create presets with same name on related motors:

```python
# All motors have 'alignment' preset
sample_x.mv_alignment()
sample_y.mv_alignment()
sample_z.mv_alignment()
```

## Preset History

```python
# View preset history/changes (if available)
motor.presets.history
```

## Best Practices

1. **Use hutch presets** for standard positions (beam, out, reference)
2. **Use experiment presets** for sample-specific positions
3. **Add comments** to document why positions were chosen
4. **Verify positions** before adding critical presets
5. **Clean up** experiment presets when done

## Common Patterns

### Alignment Workflow

```python
# Move to known position
motor.mv_reference()

# Align sample
# ... alignment procedure ...

# Save new sample position
motor.presets.add_exp_here('sample')
```

### Sample Change

```python
# Retract motors
motor1.mv_out()
motor2.mv_out()

# ... change sample ...

# Return to sample position
motor1.mv_sample()
motor2.mv_sample()
```

### Recording Current State

```python
# Save current positions of multiple motors
for motor in [m1, m2, m3]:
    motor.presets.add_exp_here(f'{motor.name}_backup')
```

## Troubleshooting

### Preset Not Found

```python
# Check available presets
motor.presets.positions

# Verify preset name spelling
```

### Cannot Save Preset

- Check write permissions to preset directory
- Verify motor name is valid for filesystem

### Preset Value Wrong

```python
# Update existing preset
motor.presets.add_hutch('sample', new_value)  # Overwrites
```
