# Neuropixels Opto Network Effects Dataset

[![DOI](https://zenodo.org/badge/1160242635.svg)](https://doi.org/10.5281/zenodo.18673526)


## Overview

This dataset contains neuropixels opto recordings from mouse cortex and stimulation data. This data supports analysis of spatially-resolved neural responses to optogenetic activation and inactivation across different power levels and emitter locations. Mice expressed DLX2.0-ChrimsonR-tdTomato (addgene #59171)

---

## Dataset Structure

```
data/
├── laser_calibration.json          # Calibration data (all lasers, mW units)
├── laser_calibration.mat           # Same data, MATLAB format
└── sessions/
    └── {subject}_{date}_en{en}_p{probe}/
        ├── session.json                    # Session metadata (see below)
        ├── spikes.times.npy                # Spike times (seconds)
        ├── spikes.clusters.npy             # Cluster ID per spike
        ├── clusters.templates.npy          # Mean waveform templates
        ├── clusters.channelPositions.npy   # Channel positions (x, y) in µm
        ├── clusters.templateDepths.npy     # Template depth (raw probe coords, µm)
        ├── clusters.depthsFromSurface.npy  # Depth from brain surface (µm)
        ├── clusters.qcPass.npy             # QC pass/fail boolean
        ├── clusters.presenceRatio.npy      # QC metric: presence ratio
        ├── clusters.isiViolationsRatio.npy  # QC metric: ISI violations ratio
        ├── clusters.amplitudeCutoff.npy    # QC metric: amplitude cutoff
        ├── clusters.troughToPeak.npy       # Spike width (ms)
        ├── clusters.peakChannel.npy        # Peak channel per cluster
        ├── clusters.waveformType.txt       # "RS" or "FS" (one per line)
        ├── clusters.KSLabel.csv            # Kilosort cluster labels
        ├── stim.times.npy                  # Stimulus onset times (seconds)
        ├── stim.emissionSites.npy          # Emitter ID (1–14)
        ├── stim.powerMW.npy               # Power at probe tip (mW)
        └── stim.durations.npy              # Pulse duration (seconds)
```


## Session Data

### Session Metadata (`session.json`)

Each session directory contains a metadata file with experimental and processing parameters:

```json
{
  "session_id": "AL0034_2024-12-05_en1_p0",
  "subject": "AL_0034",
  "date": "2024-12-05",
  "en": 1,
  "probe_index": 0,
  "top_chan": 142,
  "calibration": "quantifi",
  "power_level_scaling": 1.0,
  "sampling_rate": 30000,
  "n_spikes": 1234567,
  "n_clusters": 300,
  "n_channels": 384,
  "n_trials": 4200,
  "top_of_shank_um": 2130.0,
  "calibration_slope_mW": 0.001234,
  "calibration_intercept_mW": 0.000123,
  "spike_time_range_s": [0.0, 3600.0],
  "stim_time_range_s": [100.0, 3500.0],
  "n_qc_pass": 250,
  "n_qc_fail": 50,
  "qc_thresholds": {
    "presence_ratio_min": 0.8,
    "isi_violations_ratio_max": 0.5,
    "amplitude_cutoff_max": 0.1
  }
}
```

### Spike Data

| File | Shape | Type | Description |
|------|-------|------|-------------|
| `spikes.times.npy` | (N,) | float64 | Spike times in seconds |
| `spikes.clusters.npy` | (N,) | int32 | Cluster ID for each spike |

**N** = total number of spikes across all units

### Cluster Properties

| File | Shape | Type | Description |
|------|-------|------|-------------|
| `clusters.templates.npy` | (K, T, C) | float64 | Mean waveform templates |
| `clusters.channelPositions.npy` | (C, 2) | float64 | Channel positions (x, y) in µm |
| `clusters.templateDepths.npy` | (K,) | float64 | Template depth in raw probe coordinates (µm) |
| `clusters.depthsFromSurface.npy` | (K,) | float64 | Depth from brain surface (µm), computed using `top_chan` |
| `clusters.qcPass.npy` | (K,) | bool | QC pass/fail (presence_ratio>0.8, ISI<0.5, amp_cutoff<0.1) |
| `clusters.presenceRatio.npy` | (K,) | float64 | QC metric: fraction of session with spikes (NaN if no QC file) |
| `clusters.isiViolationsRatio.npy` | (K,) | float64 | QC metric: ISI violations ratio (NaN if no QC file) |
| `clusters.amplitudeCutoff.npy` | (K,) | float64 | QC metric: amplitude cutoff (NaN if no QC file) |
| `clusters.troughToPeak.npy` | (K,) | float64 | Spike width in milliseconds (NaN if no waveform data) |
| `clusters.peakChannel.npy` | (K,) | int32 | Peak channel index per cluster (-1 if unavailable) |
| `clusters.waveformType.txt` | K lines | text | "RS" (regular-spiking) or "FS" (fast-spiking), one per line |
| `clusters.KSLabel.csv` | (K, 2) | CSV | Kilosort cluster labels (cluster_id, KSLabel) |

**K** = number of clusters, **C** = number of channels, **T** = number of time samples in template

### Stimulation Data


| File | Shape | Type | Description |
|------|-------|------|-------------|
| `stim.times.npy` | (M,) | float64 | Stimulus onset times (seconds) |
| `stim.emissionSites.npy` | (M,) | int32 | Emitter ID (1–14) |
| `stim.powerMW.npy` | (M,) | float64 | Power at probe tip (mW), calibrated |
| `stim.durations.npy` | (M,) | float64 | Pulse duration (seconds) |

**M** = number of stimulation trials

---


## Loading Data

```python
import numpy as np
import json
import os

def load_session(session_dir):
    """Load a complete recording session from standardized files."""
    with open(os.path.join(session_dir, "session.json")) as f:
        meta = json.load(f)
    
    session = {
        "meta": meta,
        # Spikes
        "spike_times":    np.load(os.path.join(session_dir, "spikes.times.npy")),
        "spike_clusters": np.load(os.path.join(session_dir, "spikes.clusters.npy")),
        # Clusters
        "templates":           np.load(os.path.join(session_dir, "clusters.templates.npy")),
        "channel_positions":   np.load(os.path.join(session_dir, "clusters.channelPositions.npy")),
        "template_depths":     np.load(os.path.join(session_dir, "clusters.templateDepths.npy")),
        "depths_from_surface": np.load(os.path.join(session_dir, "clusters.depthsFromSurface.npy")),
        "qc_pass":             np.load(os.path.join(session_dir, "clusters.qcPass.npy")),
        "presence_ratio":      np.load(os.path.join(session_dir, "clusters.presenceRatio.npy")),
        "isi_violations":      np.load(os.path.join(session_dir, "clusters.isiViolationsRatio.npy")),
        "amplitude_cutoff":    np.load(os.path.join(session_dir, "clusters.amplitudeCutoff.npy")),
        "trough_to_peak":      np.load(os.path.join(session_dir, "clusters.troughToPeak.npy")),
        "peak_channel":        np.load(os.path.join(session_dir, "clusters.peakChannel.npy")),
        "stim_times":     np.load(os.path.join(session_dir, "stim.times.npy")),
        "emission_sites":  np.load(os.path.join(session_dir, "stim.emissionSites.npy")),
        "power_mW":        np.load(os.path.join(session_dir, "stim.powerMW.npy")),
        "stim_durations":  np.load(os.path.join(session_dir, "stim.durations.npy")),
    }
    
    # Waveform type (text file, one entry per line)
    wvf_path = os.path.join(session_dir, "clusters.waveformType.txt")
    if os.path.exists(wvf_path):
        with open(wvf_path) as f:
            session["waveform_type"] = np.array([line.strip() for line in f])
    else:
        session["waveform_type"] = np.full(session["templates"].shape[0], "", dtype=object)
    
    # Kilosort labels
    import pandas as pd
    kslabel_path = os.path.join(session_dir, "clusters.KSLabel.csv")
    if os.path.exists(kslabel_path):
        session["cluster_info"] = pd.read_csv(kslabel_path)
    
    return session

# Example usage
session = load_session('data/sessions/AL0034_2024-12-05_en1_p0')
m = session['meta']
print(f"Session: {m['session_id']}")
print(f"Spikes: {m['n_spikes']:,}  Clusters: {m['n_clusters']}  "
      f"Trials: {m['n_trials']}")
print(f"Power range: {m['power_range_mW'][0]:.4f} – {m['power_range_mW'][1]:.4f} mW")
```

---

## Analysis Notes

**Emission site geometry:** 14 sites spaced 100 µm apart starting at 70 µm from the probe tip: `[70, 170, 270, ..., 1370]` µm.

**Responsive neuron classification** (no p-value gate during classification, applied as post-filter):
- **Activated:** percent change > 200% (firing rate triples or more during stimulation)
- **Inactivated:** percent change < −50% AND baseline firing rate > 2 Hz
- Percent change = (stim_rate − pre_rate) / pre_rate, with pre-window of [−0.5, 0] s
- Significance post-filter: paired t-test p < 0.005

**QC thresholds:** presence_ratio > 0.8, ISI violations ratio < 0.5, amplitude cutoff < 0.1

---

## Contact

For questions about the dataset:  
**aladd38@uw.edu**  
**nsteinme@uw.edu**