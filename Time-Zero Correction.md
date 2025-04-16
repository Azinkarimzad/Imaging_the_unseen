---
title: "Time-Zero Correction"
teaching: 10
exercises: 2
---
---
title: "Background Removal"
teaching: 10
exercises: 2
---

:::::: questions

- What is bakground removal

::::::

:::::: objectives

- Describe background removal

::::::

:::::: keypoints

- GPR bkr

::::::

:::::: introduction

Ground Penetrating Radar (GPR) write about bkr:

```python
import numpy as np
import matplotlib.pyplot as plt
from obspy.io.segy.segy import _read_segy
from scipy.signal import hilbert
import seaborn as sns

def detect_first_break(trace, threshold_ratio=0.05, min_sample=5):
    """
    Detect the first break index using an amplitude threshold on the envelope,
    starting the search from a specified minimum sample.
    """
    envelope = np.abs(hilbert(trace))
    threshold = threshold_ratio * np.max(envelope)
    for i in range(min_sample, len(envelope)):
        if envelope[i] > threshold:
            return i
    return min_sample  # fallback

def time_zero_correction_auto(data, threshold_ratio=0.05, min_sample=5):
    """
    Automatically estimate the target sample (median of all first breaks),
    align all traces to it, and trim the section to re-zero time.
    """
    n_samples, n_traces = data.shape
    shifts = [detect_first_break(data[:, i], threshold_ratio, min_sample) for i in range(n_traces)]

    # Estimate target sample using median of detected first breaks
    target_sample = int(np.median(shifts))
    print(f"Estimated target sample for alignment: {target_sample}")

    corrected = np.zeros_like(data)
    for i in range(n_traces):
        shift = shifts[i] - target_sample
        trace = data[:, i]
        shifted_trace = np.zeros_like(trace)
        if shift >= 0:
            shifted_trace[:len(trace)-shift] = trace[shift:]
        else:
            shifted_trace[-shift:] = trace[:len(trace)+shift]
        corrected[:, i] = shifted_trace

    # Trim the top 'target_sample' rows so time-zero starts from there
    corrected_trimmed = corrected[target_sample:, :]
    return corrected_trimmed, shifts, target_sample

# --- Load GPR (SEGY) data ---
segy_file = "LINE01.sgy"
stream = _read_segy(segy_file, headonly=False)

# Build 2D data array [samples x traces]
n_traces = len(stream.traces)
max_len = max(len(tr.data) for tr in stream.traces)
data = np.zeros((max_len, n_traces))
for i, tr in enumerate(stream.traces):
    data[:len(tr.data), i] = tr.data

# --- Auto Time-Zero Correction ---
threshold_ratio = 0.05
min_sample = 5
corrected_data, shifts, target_sample = time_zero_correction_auto(data, threshold_ratio, min_sample)

# --- First-Break Shift Histogram ---
plt.figure(figsize=(10, 3))
sns.histplot(shifts, bins=20, kde=True)
plt.title("Distribution of First-Break Shifts (Samples)")
plt.xlabel("Sample")
plt.ylabel("Frequency")
plt.tight_layout()
plt.show()

# --- Overlay Original vs Corrected for One Trace ---
plt.figure(figsize=(10, 4))
idx = 1000
plt.plot(data[:, idx], label="Original", alpha=0.6)
aligned = np.zeros_like(data[:, idx])
shift = shifts[idx] - target_sample
if shift >= 0:
    aligned[:len(aligned) - shift] = data[:, idx][shift:]
else:
    aligned[-shift:] = data[:, idx][:len(aligned) + shift]
plt.plot(aligned[target_sample:], label="Corrected", linestyle="--")
plt.title(f"Trace {idx} - Original vs Auto-Aligned")
plt.xlabel("Sample")
plt.ylabel("Amplitude")
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()

# --- Plot First-Break Picks on Sample Traces ---
plt.figure(figsize=(10, 6))
for i in range(5):
    idx = i * (n_traces // 5)
    plt.plot(data[:, idx], label=f"Trace {idx}")
    plt.axvline(x=shifts[idx], color='r', linestyle='--')
plt.axhline(target_sample, color='k', linestyle=':', label='Auto Target Sample')
plt.title("First-Break Picks on Sample Traces")
plt.xlabel("Sample")
plt.ylabel("Amplitude")
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()

# --- Full Section: Before vs After Correction ---
fig, axs = plt.subplots(1, 2, figsize=(14, 6), sharey=False)

axs[0].imshow(data, cmap="gray", aspect="auto", origin="upper")
axs[0].set_title("Before Time-Zero Correction")
axs[0].set_xlabel("Trace")
axs[0].set_ylabel("Sample")

axs[1].imshow(corrected_data, cmap="gray", aspect="auto", origin="upper")
axs[1].set_title(f"After Auto Time-Zero Correction (Aligned to Sample {target_sample})")
axs[1].set_xlabel("Trace")

plt.tight_layout()
plt.show()

```






::::::
