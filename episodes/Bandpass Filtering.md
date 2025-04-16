---
title: "Bandpass Filtering"
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
from scipy.signal import butter, filtfilt
from obspy.io.segy.segy import _read_segy

def bandpass_filter(data, lowcut, highcut, fs, order=4):
    nyquist = 0.5 * fs
    low = lowcut / nyquist
    high = highcut / nyquist
    if not 0 < low < 1 or not 0 < high < 1:
        raise ValueError(f"Cutoff frequencies must be within (0, {nyquist:.2f} Hz). Got low={lowcut}, high={highcut}")
    b, a = butter(order, [low, high], btype='band')
    
    filtered = np.zeros_like(data)
    for i in range(data.shape[1]):
        filtered[:, i] = filtfilt(b, a, data[:, i])
    return filtered

def plot_frequency_spectrum(trace, fs, title):
    """
    Plot frequency amplitude spectrum of a single trace.
    """
    n = len(trace)
    freqs = np.fft.rfftfreq(n, d=1/fs)
    spectrum = np.abs(np.fft.rfft(trace))
    
    plt.plot(freqs / 1e6, spectrum, label=title)  # MHz
    plt.xlabel("Frequency (MHz)")
    plt.ylabel("Amplitude")
    plt.grid(True)

# --- Load SEG-Y GPR data ---
segy_file = "LINE01.sgy"
stream = _read_segy(segy_file, headonly=False)

# Build 2D data array [samples x traces]
n_traces = len(stream.traces)
n_samples = max(len(tr.data) for tr in stream.traces)
data = np.zeros((n_samples, n_traces))
for i, tr in enumerate(stream.traces):
    data[:len(tr.data), i] = tr.data

# --- Sampling rate ---
dt_us = stream.binary_file_header.sample_interval_in_microseconds  # microseconds
if dt_us == 0:
    raise ValueError("Sample interval is missing or zero in SEG-Y header. Set manually if needed.")

fs = 1e6 / dt_us  # Hz
nyquist = fs / 2
print(f"Sampling frequency: {fs/1e6:.2f} MHz, Nyquist: {nyquist/1e6:.2f} MHz")

# --- Dynamically determine bandpass range ---
lowcut = 0.05 * nyquist   # 5% of Nyquist
highcut = 0.95 * nyquist  # 95% of Nyquist
print(f"Applying bandpass filter: {lowcut/1e6:.2f}–{highcut/1e6:.2f} MHz")

# --- Apply bandpass filter ---
filtered_data = bandpass_filter(data, lowcut, highcut, fs)

# --- Plot before and after ---
fig, axs = plt.subplots(1, 2, figsize=(14, 6), sharey=True)

axs[0].imshow(data, cmap="gray", aspect="auto", origin="upper")
axs[0].set_title("Original GPR Data")
axs[0].set_xlabel("Trace")
axs[0].set_ylabel("Sample")

axs[1].imshow(filtered_data, cmap="gray", aspect="auto", origin="upper")
axs[1].set_title(f"Bandpass Filtered GPR\n({lowcut/1e6:.1f}–{highcut/1e6:.1f} MHz)")
axs[1].set_xlabel("Trace")

plt.tight_layout()
plt.show()

# --- Plot Frequency Spectra ---
middle_trace_idx = n_traces // 2
plt.figure(figsize=(10, 5))
plot_frequency_spectrum(data[:, middle_trace_idx], fs, title="Before Filter")
plot_frequency_spectrum(filtered_data[:, middle_trace_idx], fs, title="After Filter")
plt.title("Frequency Spectrum of Middle Trace")
plt.legend()
plt.tight_layout()
plt.show()


```






::::::
