# Crystal Alignment with QExpress in hutch-python

This document describes the procedure for obtaining crystal alignment information at a resonant energy (E_R) from a higher non-resonant energy (E_NR), using the QExpress package within hutch-python at the RIX beamline.

---

## 1. Background: Why Transfer Alignment Between Energies?

In resonant inelastic X-ray scattering (RIXS), measurements are performed at resonant energies (e.g., the Cu L3-edge at ~930 eV) where the X-ray scattering cross-section changes dramatically. This makes it difficult to directly calibrate Bragg reflections at the resonant energy.

The solution is to:

1. **Calibrate at a non-resonant energy (E_NR)** where reflections are clean and predictable (e.g., 2 keV)
2. **Determine the crystal's UB orientation matrix** at E_NR
3. **Transfer** that calibration to the resonant energy (E_R) by recalculating expected diffractometer angles for the new wavelength

This approach is based on the Busing & Levy formalism for diffractometer orientation.

## 2. The qRIXS Diffractometer

The qRIXS diffractometer (R-000054615) provides the following axes:

| Axis | Range | Description |
|------|-------|-------------|
| Theta (th) | +90° / -120° | Sample rotation (incident angle) |
| 2-Theta (tth) | +10° / -170° | Detector arm angle |
| Phi | +/- 180° | Sample azimuthal rotation |
| Chi | +/- 10° | Sample tilt |
| X/Y | +/- 0.79 | Sample translation |
| Z | +/- 0.98 | Sample height |
| Detector Y | +/- 1.97 | Detector vertical offset |

The X-ray beam travels along the X direction. These axes are controlled as EpicsMotor objects through hutch-python.

## 3. The 4-Step Alignment Procedure

### Step A: Pre-beamtime Knowledge

Before arriving at the beamline, prepare:

1. **Unit cell parameters** (a, b, c, alpha, beta, gamma) — saved as a `sample` dictionary
2. **Sample orientation** on the mount (from Laue diffraction or single-crystal diffractometer)

In QExpress, the sample is defined as a Python dictionary:

```python
sample = {
    'name': 'YBCO',
    'lattice': {
        'a': 3.82, 'b': 3.89, 'c': 11.68,
        'alpha': 90, 'beta': 90, 'gamma': 90
    }
}
```

Sample data can also be stored as `.npy` files and loaded with `np.load()`.

### Step B: Calibrating Reflections at E_NR

Using the sample orientation from Step A, locate **at least two non-collinear reflections** at the non-resonant energy E_NR. For each reflection, record the Miller index (h, k, l) and the diffractometer angular positions (theta, 2-theta, chi, phi).

The reflections are stored as a multi-layer Python dictionary:

```python
reflections_reference = {
    0: {
        'hkl': [0, 0, 2],
        'angles': {'theta': ..., 'tth': ..., 'chi': ..., 'phi': ...}
    },
    1: {
        'hkl': [1, 0, 2],
        'angles': {'theta': ..., 'tth': ..., 'chi': ..., 'phi': ...}
    }
}
```

In the soft X-ray regime, only a few crystallographic reflections are typically accessible due to the long wavelength.

### Step C: Checking the Orientation at E_NR

Verify the orientation matrix by running `QE.align()` at E_NR and confirming that calculated motor positions match the measured ones for the calibration reflections. It is recommended to check at least one of the calibration reflections.

### Step D: Transferring from E_NR to E_R

Use `QE.reflection_conversion()` to generate new reference reflections at the resonant energy, then `QE.align()` to compute motor positions at E_R. This is described in detail in Section 5.

## 4. QExpress Package

QExpress (`QExpress_v1.py`) is a Python package that runs inside RIX hutch-python. It uses `hklpy` (a Python interface to diffractometer calculations based on the Busing/Levy formalism) for UB matrix computation and four-circle inverse calculations.

**Location:** `/cds/home/opr/rixopr/Documents/Lingjia/QExpress_v1.py`

**Prerequisites:**
- A set of non-collinear reflections from alignment calibration
- Known lattice parameters
- Access to RIX hutch-python

### 4.1 `align()` — Compute Motor Positions for a Target Reflection

The core function. Given a calibrated orientation, it returns the diffractometer angles for any target (h, k, l).

```python
import QExpress_v1 as QE

solution = QE.align(sample, xray_energy, reflections_reference, reflection_target)
```

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `sample` | dict | Keys: `'name'` (str), `'lattice'` (dict with a, b, c, alpha, beta, gamma) |
| `xray_energy` | float | Incident X-ray energy in keV |
| `reflections_reference` | dict | Calibrated reflections with Miller indices and angles |
| `reflection_target` | numpy array | Target reflection as [h, k, l] |

**Returns:** Diffractometer motor positions (theta, 2-theta, chi, phi) for the target reflection. Also displays the U and UB orientation matrices and lattice parameters.

**Example — YBCO at 2 keV:**

```python
import numpy as np
import QExpress_v1 as QE

xray_energy = 2.0  # keV
reflection_target = np.array([0.5, 0, 1.5])

solution = QE.align(sample, xray_energy, reflections_reference, reflection_target)
# Returns diffractometer angles for (0.5, 0, 1.5) at 2 keV
```

### 4.2 `reflection_conversion()` — Transfer Reflections to a New Energy

Generates a set of reference reflections for UB matrix calculation at a different (typically resonant) energy, based on a calibration performed at a higher energy.

```python
reflections_rxs = QE.reflection_conversion(sample, xray_energy, reflections_reference, rxs_energy)
```

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `sample` | dict | Sample definition (same as `align()`) |
| `xray_energy` | float | Original calibration energy in keV (E_NR) |
| `reflections_reference` | dict | Calibrated reflections at E_NR |
| `rxs_energy` | float | Target resonant energy in keV (E_R) |

**Returns:** A new `reflections_rxs` dictionary containing two reflections suitable for UB matrix calculation at E_R.

### 4.3 Energy Conversion Method

The energy conversion approach (developed May 2023) generates two virtual reflections at E_R:

**Reflection 1 — Specular reflection:**
- Uses `hklpy`'s `fourc.inverse` to find q1 = (h, k, l) at E_NR with tth = x1, th = 0.5*x1, chi = phi = 0
- Since this is specular, th = tth/2 holds at all energies
- The tth at E_R is recalculated using Bragg's law

**Reflection 2 — Orthogonal reflection:**
- Finds q2 = (h, k, l) at E_NR with tth = x2, th = 0.5*x1 +/- 90°, chi = phi = 0
- At E_R, theta of q2 is the theta of q1 offset by +/- 90°
- The tth of q2 at E_R is calculated from Bragg's law

The second reflection does not strictly need to be orthogonal — any non-zero angular offset works, but orthogonal is convenient.

These two reflections define the UB matrix at E_R when fed back into `align()`.

## 5. Complete Workflow in hutch-python

### 5.1 Basic Alignment at E_NR

```python
# 1. Enter RIX hutch python
#    $ rixpython

# 2. Import QExpress
import QExpress_v1 as QE
import numpy as np

# 3. Load sample and calibration data
sample = np.load('sample.npy', allow_pickle=True).item()
reflections_reference = np.load('reflections_reference.npy', allow_pickle=True).item()

# 4. Set calibration energy
xray_energy = 2.0  # keV (non-resonant)

# 5. Verify alignment at E_NR
reflection_target = np.array([0, 0, 2])
solution = QE.align(sample, xray_energy, reflections_reference, reflection_target)
# Displays U, UB matrices and motor positions
```

### 5.2 Transfer to Resonant Energy

```python
# 6. Convert reflections to resonant energy
rxs_energy = 0.93  # keV (Cu L-edge)
reflections_rxs = QE.reflection_conversion(sample, xray_energy, reflections_reference, rxs_energy)

# 7. Calculate motor positions at resonant energy
reflection_target = np.array([0.31, 0, 1.45])
solution = QE.align(sample, rxs_energy, reflections_rxs, reflection_target)
# Returns theta, 2-theta, chi, phi for the target reflection at Cu L-edge
```

### 5.3 Constant-Q Energy Scan

To perform an energy scan at a fixed Q position (e.g., for RIXS measurements across an absorption edge):

```python
# Prerequisites:
# - UB matrix defined at E_NR
# - Target Q position defined

Q_target = np.array([0.31, 0, 1.45])

# Scan from 920 eV to 950 eV in 5 eV steps
for energy_eV in range(920, 955, 5):
    energy_keV = energy_eV / 1000.0

    # Convert reflections to this energy
    refl = QE.reflection_conversion(sample, xray_energy, reflections_reference, energy_keV)

    # Get motor positions
    sol = QE.align(sample, energy_keV, refl, Q_target)

    # Move motors to calculated positions and collect data
    # (motor moves and DAQ integration via hutch-python)
```

This loop calls `reflection_conversion()` and `align()` at each energy step, returning the diffractometer angles needed to stay at the same Q while the energy changes.

### 5.4 Moving Motors and Collecting Data

Once QExpress returns motor positions, hutch-python handles the physical motion and data collection:

```python
# Move diffractometer axes (using hutch-python motor control)
theta_motor.mv(solution['theta'])
tth_motor.mv(solution['tth'])

# Or move multiple axes simultaneously
bps.mv(theta_motor, solution['theta'],
       tth_motor, solution['tth'],
       chi_motor, solution['chi'],
       phi_motor, solution['phi'])

# Collect data with DAQ
daq.preconfig(events=120, record=True)
RE(count([daq], num=1))
```

## 6. Known Issues and Notes

From the May 2023 development log:

- **`align()` re-run bug (fixed):** The function would fail on re-runs with a "sample exists" error. Root cause: the four-circle class was not being re-initialized. Fixed as of 05-08-2023.
- **Input format:** Sample and reflections data currently use numpy dictionaries (`.npy` files). Future versions may support text/CSV/free-format inputs.
- **qRIXS arm geometry:** Transformation for the qRIXS spectrometer arm geometry requires further collaboration (noted as non-trivial).
- **QExpress is simulator-only as of initial development:** Motor communications were not yet integrated at the time of the May 2023 log. The package calculates positions but the motor moves are performed separately through hutch-python.

## 7. References

- Busing, W. R. & Levy, H. A. (1967). "Angle Calculations for 3- and 4-Circle X-ray and Neutron Diffractometers." *Acta Cryst.* 22, 457-464.
- [hklpy](https://blueskyproject.io/hklpy/) — Python interface to diffractometer calculations
- qRIXS Diffractometer engineering drawing R-000054615
- QExpress development log, May 2023

---

*Document created: 2026-02-18*
*Sources: QExpress May 2023 development log, qRIXS diffractometer drawing, hutch-python skill references*
