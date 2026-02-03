# Safety Measures for IPython Bridge

This document outlines strategies to prevent accidental motor movements or other dangerous operations when using the IPython bridge with Claude Code.

## The Risk

Claude Code can send arbitrary Python code to hutch-python via the bridge. A malformed or misunderstood command could:
- Move motors to wrong positions
- Damage equipment
- Interrupt running experiments

## Safety Options

### 1. Read-Only Client Mode (Simplest)

Create a bridge mode that only allows "safe" operations:
- Whitelist: only allow `.get()`, `.position`, `.read()` methods
- Reject any code containing `.mv(`, `.set(`, `=` assignment to motor attributes

### 2. Confirmation Protocol

Require a two-step process for writes:
```python
# Step 1: Claude proposes
{"code": "motor.mv(10)", "mode": "propose"}
# Response: "Proposed: move motor from 5.0 to 10.0. Confirm with token: abc123"

# Step 2: Human confirms (copy-paste the token)
{"code": "motor.mv(10)", "confirm": "abc123"}
```

### 3. Bridge-Level Restrictions

Modify `ipython_bridge.py` to:
- Parse the code and reject dangerous patterns before execution
- Maintain a whitelist of allowed commands
- Log all commands to a file for audit

Example implementation:
```python
DANGEROUS_PATTERNS = ['.mv(', '.set(', '.move(', '.put(']

def _execute(self, request):
    code = request.get('code', '')

    # Safety check
    if any(p in code for p in DANGEROUS_PATTERNS):
        return {'status': 'rejected', 'error': 'Write operations not allowed'}

    # ... rest of execution
```

### 4. Soft Limits in Bridge

Before any motor move, the bridge could:
- Check if move is within soft limits
- Require explicit `force=True` for moves > X units
- Show current position and delta before executing

### 5. User Approval Hook (Claude Code Level)

Configure Claude Code hooks to require confirmation before running any command containing motor keywords. This operates outside the bridge itself.

## Recommendation

Start with **Option 1 or 3**: Add a simple code filter in the bridge that rejects anything that looks like a motor write. This approach is:
- Easy to implement
- Transparent (clear error messages)
- Can be toggled off when write access is explicitly needed

## Current Status

- [ ] Implement chosen safety measure
- [ ] Test with simulated motors
- [ ] Document how to enable/disable write access
