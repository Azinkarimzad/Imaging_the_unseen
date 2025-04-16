---
title: "Deconvolution"
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
from scipy.fft import rfft, irfft, rfftfreq

def spectral_decon(trace, fs, stab=0.01):
    """
    Apply frequency-domain (spectral) deconvolution to a trace.

    Parameters:
        trace (1D array): input trace
        fs (float): sampling frequency in Hz
        stab (float): stabilization constant

    Returns:
        deconvolved trace (1D array)
    """
    n = len(trace)
    f = rfftfreq(n, d=1/fs)
    spectrum = rfft(trace)
    amp = np.abs(spectrum)
    phase = spectrum / (amp + 1e-12)  # preserve phase

    # Invert the amplitude spectrum with stabilization
    inverse_amp = 1 / (amp + stab * np.max(amp))
    decon_spectrum = inverse_amp * spectrum

    # Reconstruct the time-domain signal
    decon_trace = irfft(decon_spectrum, n=n)
    return decon_trace

# --- Load SEG-Y GPR data ---
segy_file = "LINE01.sgy"
stream = _read_segy(segy_file, headonly=False)

# Build 2D data array [samples x traces]
n_traces = len(stream.traces)
n_samples = max(len(tr.data) for tr in stream.traces)
data = np.zeros((n_samples, n_traces))
for i, tr in enumerate(stream.traces):
    data[:len(tr.data), i] = tr.data

# --- Sampling frequency from header ---
try:
    dt_us = stream.binary_file_header.sample_interval_in_microseconds
    if dt_us == 0 or dt_us > 10:
        print(f"Header sample interval {dt_us} µs seems invalid. Overriding to 0.5 µs.")
        dt_us = 0.5
except:
    print("Sample interval not found. Using default of 0.5 µs.")
    dt_us = 0.5

fs = 1e6 / dt_us  # sampling frequency in Hz
print(f"Sampling rate: {fs/1e6:.2f} MHz")

# --- Apply spectral deconvolution ---
decon_data = np.zeros_like(data)
for i in range(n_traces):
    decon_data[:, i] = spectral_decon(data[:, i], fs=fs, stab=0.5)

# --- Plot comparison ---
fig, axs = plt.subplots(1, 2, figsize=(14, 6), sharey=True)

axs[0].imshow(data, cmap="gray", aspect="auto", origin="upper")
axs[0].set_title("Original GPR Data")
axs[0].set_xlabel("Trace")
axs[0].set_ylabel("Sample")

axs[1].imshow(decon_data, cmap="gray", aspect="auto", origin="upper")
axs[1].set_title("Spectral Deconvolved GPR Data")
axs[1].set_xlabel("Trace")

plt.tight_layout()
plt.show()


```






::::::
