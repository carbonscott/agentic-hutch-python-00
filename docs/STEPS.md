# IPython <-> Claude Code Bridge Setup

## Overview

This bridge enables Claude Code to send commands to a hutch-python (or any IPython) session and receive results back.

```
Claude Code (node A)          SSH Tunnel              IPython (node B, e.g. rix-daq)
     |                           |                           |
     |--- localhost:9999 ------->|--- forwarded ------------>| bridge server
     |<-- response --------------|<-- response --------------|
```

## Files

| File | Where to run | Purpose |
|------|--------------|---------|
| `ipython_bridge.py` | In hutch-python/IPython | Server - listens for commands |
| `ipython_client.py` | On Claude Code's host | Client - sends commands |

---

## Step 1: Start the Bridge (on rix-daq)

SSH to the hutch-python node and start your session:
```bash
ssh rixopr@rix-daq
rix3   # or just: ipython
```

In the IPython session, load the bridge:
```python
%run /sdf/data/lcls/ds/prj/prjdat21/results/cwang31/fun/test-hutch-python/ipython_bridge.py
```

You should see:
```
IPython bridge listening on localhost:9999
```

The bridge is now running. You can continue using IPython normally.

---

## Step 2: Create SSH Tunnel (from Claude Code's host)

### Option A: Direct tunnel (if you have direct SSH access)

On the machine where Claude Code runs, open a terminal and create a tunnel:
```bash
ssh -L 9999:localhost:9999 rixopr@rix-daq -N
```

- Enter your password once
- The terminal will appear to hang (this is normal - the tunnel is open)
- Keep this terminal open

### Option B: Two-hop tunnel (via psdev)

If you need to go through an intermediate host (e.g., `sdfiana025 → psdev → rix-daq`):

**Terminal 1** (on sdfiana025, where Claude Code runs):
```bash
ssh -L 9999:localhost:9998 psdev
```
Keep this open.

**Terminal 2** (on psdev, after Terminal 1 connects):
```bash
sshlogin -L 9998:localhost:9999 rixopr@rix-daq
```
Keep this open.

This creates a chain: `sdfiana025:9999 → psdev:9998 → rix-daq:9999`

Now connections to `localhost:9999` on sdfiana025 will reach the bridge on rix-daq.

---

## Step 3: Test the Connection

### Option A: Quick test with netcat
```bash
echo '{"code": "1+1"}' | nc localhost 9999
```

Expected response:
```json
{"status": "ok", "result": "2", "stdout": "", "stderr": "", "error": null}
```

### Option B: Test with Python client
```python
from ipython_client import IPythonClient

client = IPythonClient('localhost', 9999)
print(client.is_alive())  # Should print: True
print(client.execute('1+1'))  # Should show result: '2'
```

---

## Step 4: Use from Claude Code

Claude Code can now send commands:
```python
# Check motor position
result = client.execute('motor_x.position')
print(result['result'])

# Move motor
result = client.execute('motor_x.mv(10)')
print(result['stdout'])

# Run a scan
result = client.execute('RE(bp.scan([det], motor_x, 0, 10, 11))', timeout=300)
print(result['stdout'])
```

---

## Stopping the Bridge

In the IPython session:
```python
stop_bridge()
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Connection refused" | Bridge not running. Run `%run ipython_bridge.py` in IPython |
| Timeout | SSH tunnel not open, or wrong port |
| "Not in IPython session" | Run the bridge inside IPython, not as a standalone script |
| Bridge hung | `Ctrl+C` in IPython, then `stop_bridge()` and restart |

---

## Quick Reference

```
# On rix-daq (IPython):
%run /sdf/data/lcls/ds/prj/prjdat21/results/cwang31/fun/test-hutch-python/ipython_bridge.py
stop_bridge()  # to stop

# On Claude Code host (terminal):
ssh -L 9999:localhost:9999 rixopr@rix-daq -N

# Test:
echo '{"code": "1+1"}' | nc localhost 9999
```
