---
title: "AGC"
teaching: 10
exercises: 2
---

:::::: questions

- What isAGC

::::::

:::::: objectives

- Describe AGC

::::::

:::::: keypoints

- GPR AGC

::::::

:::::: introduction

Ground Penetrating Radar (GPR) write about AGC:

```python
import numpy as np
import matplotlib.pyplot as plt
from obspy.io.segy.segy import _read_segy

def apply_agc(data, window_len):
    """
    Apply automatic gain control (AGC) on a 2D seismic array.

    Parameters:
        data (ndarray): [samples x traces]
        window_len (int): Window length in samples

    Returns:
        ndarray: AGC-applied seismic section
    """
    agc_data = np.zeros_like(data)
    half_window = window_len // 2

    for i in range(data.shape[1]):  # loop over traces
        trace = data[:, i]
        agc_trace = np.zeros_like(trace)

        for j in range(len(trace)):
            start = max(j - half_window, 0)
            end = min(j + half_window, len(trace))
            window = trace[start:end]
            rms = np.sqrt(np.mean(window ** 2)) + 1e-12  # avoid divide by zero
            agc_trace[j] = trace[j] / rms

        agc_data[:, i] = agc_trace

    return agc_data

# --- Load SEG-Y ---
segy_file = "LINE01.sgy"
stream = _read_segy(segy_file, headonly=False)

# --- Build data array ---
n_traces = len(stream.traces)
max_len = max(len(tr.data) for tr in stream.traces)
data = np.zeros((max_len, n_traces))
for i, tr in enumerate(stream.traces):
    data[:len(tr.data), i] = tr.data

# --- Apply AGC ---
window_length = 50  # samples (adjust based on data)
agc_result = apply_agc(data, window_length)

# --- Plot ---
plt.figure(figsize=(12, 6))
plt.imshow(agc_result, cmap="gray", aspect="auto", origin="upper")
plt.title("Seismic Section with AGC (window = {} samples)".format(window_length))
plt.xlabel("Trace Number")
plt.ylabel("Time Sample")
plt.colorbar(label="Amplitude (AGC-normalized)")
plt.show()

```






::::::
