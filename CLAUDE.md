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

## Environment Setup

This project uses [uv](https://github.com/astral-sh/uv) for Python package management.

**Install dependencies:**
```bash
source .env && uv sync
```

The `.env` file contains:
- `UV_CACHE_DIR` - Cache location for uv packages

## Running Scripts

Always source `.env` before running to set the UV cache directory:

```bash
source .env && uv run <script.py>
```

## Agent Skills Directory

```bash
export AGENT_SKILL_DIR='/sdf/data/lcls/ds/prj/prjdat21/results/cwang31/resources/agentskills'
```

## Project Skills

This project includes a hutch-python skill for teaching AI agents about LCLS beamline operations:

**Skill Location:** `skills/hutch-python/`

When helping users with hutch-python, motor control, scanning, DAQ operations, or beamline configuration, refer to the skill files:
- `skills/hutch-python/SKILL.md` - Main instructions
- `skills/hutch-python/references/CONFIGURATION.md` - conf.yml options
- `skills/hutch-python/references/SCANNING.md` - Bluesky scanning
- `skills/hutch-python/references/DAQ.md` - Data acquisition
- `skills/hutch-python/references/NAMESPACES.md` - Device organization
- `skills/hutch-python/references/PRESETS.md` - Motor presets
