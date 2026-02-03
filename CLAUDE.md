export UV_CACHE_DIR='/sdf/data/lcls/ds/prj/prjdat21/results/cwang31/.UV_CACHE/'

## Communicating with hutch-python on rix-daq

When the SSH tunnel is active, Claude Code can send commands to the hutch-python session running on rix-daq.

### Quick Test
```bash
echo '{"code": "1+1"}' | nc localhost 9999
```

### Using the Python Client
```bash
python /sdf/data/lcls/ds/prj/prjdat21/results/cwang31/fun/test-hutch-python/ipython_client.py
```

### JSON Protocol
Send newline-terminated JSON to `localhost:9999`:
```json
{"code": "motor.position", "capture": true}
```

Response:
```json
{"status": "ok", "result": "10.5", "stdout": "", "stderr": "", "error": null}
```

### Setup Instructions
See `STEPS.md` for full setup instructions including:
- Starting the bridge in hutch-python
- Setting up the two-hop SSH tunnel (sdfiana025 → psdev → rix-daq)
