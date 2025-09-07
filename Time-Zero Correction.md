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

- What does time-zero represent in a GPR trace?  
- Why do system delays and antenna geometry cause time-zero to be shifted?  
- How can first breaks be detected automatically in traces?  
- What happens to interpretation if time-zero is not corrected?  
- How can we compare sections before and after time-zero correction? 

::::::

:::::: objectives

- Define time-zero and explain its importance for accurate depth conversion.  
- Identify common causes of time-zero shifts in GPR data.  
- Demonstrate an automatic time-zero correction workflow using first-break detection.  
- Visualize the distribution of first-break picks and assess their consistency.  
- Compare radargrams before and after correction to evaluate improvements.

::::::

:::::: keypoints

- Time-zero is the instant when the transmitted pulse enters the ground.  
- System delays and geometry shift the apparent time-zero away from zero samples.  
- Automatic methods detect first breaks using the signal envelope and amplitude thresholds.  
- Aligning traces to a common first break ensures consistent interpretation.  
- Time-zero correction is essential for reliable time-to-depth conversion and structural imaging.

::::::

:::::: introduction


## Time-Zero in GPR

In Ground Penetrating Radar (GPR), **time-zero** is the reference point corresponding to the instant when the transmitted electromagnetic pulse leaves the antenna and enters the medium. Ideally, this should appear at the start of every recorded trace. In practice, traces often show a shift in time-zero due to:

- electronic delays in the system trigger and recording,
- physical separation between transmitting and receiving antennas,
- coupling effects between antennas and the ground surface.

These shifts cause reflections to appear later than their true arrival time. If left uncorrected, such misalignment leads to errors in estimating depths and comparing events across traces.

**Time-zero correction** realigns all traces so that their effective start time corresponds to the true wave entry. This process improves consistency and ensures that travel times can be correctly converted to depth using velocity models.

In this lesson we use an **automatic first-break detection method**. The Hilbert transform is applied to compute the signal envelope, and a threshold relative to the maximum amplitude is used to pick the first significant arrival in each trace. The median of these first breaks defines the common alignment point. Traces are then shifted to this reference, and the section is trimmed so the new time axis begins at zero.

This correction is essential before velocity analysis, migration, or quantitative interpretation of GPR data. It is a standard preprocessing step in both engineering and archaeological applications.

::::::

## Example 1 — Automatic Time-Zero Correction using First-Break Detection

The following code implements an automatic method to detect first breaks in GPR traces and realign all traces so that their time-zero is consistent. This approach uses the Hilbert transform to compute the signal envelope and an amplitude threshold to identify the first arrival.

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
Step-by-step explanation

First-break detection

The Hilbert transform computes the analytic signal; its absolute value gives the envelope.

The code sets a threshold at 5% (threshold_ratio=0.05) of the maximum envelope.

The first sample exceeding this threshold (after min_sample) is picked as the first break.

Automatic alignment

All traces are analyzed, producing a list of first-break indices (shifts).

The median of these indices defines the target sample for alignment.

Each trace is shifted so that its first break matches this target.

Section trimming

Samples above the target are removed, so the new section begins at a common time-zero.

Quality control plots

A histogram of first breaks shows their distribution and helps check consistency.

A trace overlay (original vs corrected) illustrates the alignment.

Vertical lines on sample traces visualize first-break picks.

Side-by-side radargrams show the effect of correction at the section scale.

Interpretation

After correction, all traces share the same starting point, enabling accurate depth conversion.

Misalignments due to system delays are removed, improving reflector continuity.

The method is automatic but can be adjusted (e.g., threshold ratio, minimum sample) to fit data characteristics.


## Example 2 — Manual Time-Zero Correction using a Reference Trace

In some cases, automatic picking may not be reliable (e.g. noisy data, strong early clutter).  
A simpler approach is to pick a **reference trace** manually and align all other traces to its first break.

```
# --- Pick a reference trace manually ---
ref_idx = 500  # example: middle trace as reference
ref_trace = data[:, ref_idx]

# Detect first break on reference trace
ref_fb = detect_first_break(ref_trace, threshold_ratio=0.05, min_sample=5)
print(f"Reference trace index: {ref_idx}, First break sample: {ref_fb}")

# Detect first breaks on all traces
all_fb = [detect_first_break(data[:, i], threshold_ratio=0.05, min_sample=5) for i in range(n_traces)]

# Align traces to the reference first break
corrected_manual = np.zeros_like(data)
for i in range(n_traces):
    shift = all_fb[i] - ref_fb
    trace = data[:, i]
    shifted_trace = np.zeros_like(trace)
    if shift >= 0:
        shifted_trace[:len(trace)-shift] = trace[shift:]
    else:
        shifted_trace[-shift:] = trace[:len(trace)+shift]
    corrected_manual[:, i] = shifted_trace

# Trim so time-zero begins at the reference first break
corrected_manual = corrected_manual[ref_fb:, :]

# Plot before and after
fig, axs = plt.subplots(1, 2, figsize=(14, 6), sharey=False)
axs[0].imshow(data, cmap="gray", aspect="auto", origin="upper")
axs[0].set_title("Before Manual Time-Zero Correction")
axs[0].set_xlabel("Trace")
axs[0].set_ylabel("Sample")

axs[1].imshow(corrected_manual, cmap="gray", aspect="auto", origin="upper")
axs[1].set_title(f"After Manual Correction (Aligned to Trace {ref_idx})")
axs[1].set_xlabel("Trace")

plt.tight_layout()
plt.show()
```
Explanation

A reference trace is chosen manually (here: index 500).

Its first break defines the target time-zero.

All other traces are shifted relative to this reference.

Useful when automatic methods pick inconsistent breaks due to noise or strong clutter.

Less robust if the reference trace itself is distorted, so careful selection is required.


## Example 3 — Visualizing First-Break Picks Across the Section

To check the reliability of time-zero correction, it is useful to overlay first-break picks on top of the raw section.

```
# Detect first breaks on all traces (reuse detect_first_break function)
first_breaks = [detect_first_break(data[:, i], threshold_ratio=0.05, min_sample=5) for i in range(n_traces)]

# Plot section with first-break picks overlaid
plt.figure(figsize=(12, 6))
plt.imshow(data, cmap="gray", aspect="auto", origin="upper")
plt.scatter(np.arange(n_traces), first_breaks, color='red', s=10, label='First breaks')
plt.title("Raw Section with First-Break Picks")
plt.xlabel("Trace Number")
plt.ylabel("Sample Index")
plt.legend()
plt.show()
```
Explanation

Each red point marks the picked first break for one trace.

A consistent horizontal band of picks indicates reliable automatic detection.

Large scatter suggests noise or variable coupling; parameters (threshold, minimum sample) may need adjusting.

This visualization helps students evaluate whether the correction will be successful before applying shifts.
