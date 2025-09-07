---
title: "AGC"
teaching: 10
exercises: 2
---

:::::: questions

- Why do GPR signals decrease in amplitude with increasing two-way travel time?

- What is the principle of Automatic Gain Control (AGC) in signal processing?

- How does the choice of window length influence the effect of AGC on a radargram?

- How can AGC be implemented in Python to enhance deeper reflections in SEG-Y data?

- What are the potential benefits and drawbacks of applying AGC to GPR data?

::::::

:::::: objectives

- Explain the physical reason for signal decay in GPR traces and why amplitude balancing is needed.

- Describe the concept of AGC as a sliding-window normalization method.

- Demonstrate the step-by-step implementation of AGC on a 2D data section using Python.

- Evaluate how window length selection affects signal visibility and noise amplification.

- Compare radargrams before and after AGC to interpret the improvement in deeper reflections.

::::::

:::::: keypoints

- GPR amplitudes decay with travel time due to geometric spreading, absorption, and scattering.

- Automatic Gain Control (AGC) rescales amplitudes within a moving window to balance early and late signals.

- Window length controls the behavior: short windows emphasize local contrasts but may boost noise, long windows smooth variations but may underrepresent weak reflections.

- AGC does not recover true amplitudes; it enhances relative visibility for interpretation.

- In Python, AGC can be implemented by normalizing each trace sample by the RMS of its surrounding window.

::::::

:::::: introduction

Automatic Gain Control (AGC) in GPR

One of the main challenges in Ground Penetrating Radar (GPR) interpretation is that recorded amplitudes decrease with increasing two-way travel time. This decay occurs because of several factors: geometric spreading of the wavefront, intrinsic absorption by the medium, and scattering losses. As a result, reflections from deeper structures often appear weak compared to strong early arrivals near the surface.

Automatic Gain Control (AGC) is a signal-processing technique designed to correct this imbalance. Instead of applying a single global gain function, AGC adjusts amplitudes locally using a sliding window. Within each window, amplitudes are normalized relative to their average energy (root mean square, RMS). This ensures that both shallow and deep events are displayed with more balanced strength.

The effect of AGC depends strongly on the window length:

Short windows enhance small-scale features but may also amplify random noise.

Long windows provide smoother balancing but may underemphasize weak events.

It is important to remember that AGC modifies amplitudes for visualization and interpretation rather than preserving true reflection strength. In this lesson, we will implement AGC in Python, apply it to a SEG-Y radargram, and compare sections before and after processing to see how deeper reflections become clearer.

## Example 1 — Applying Automatic Gain Control (AGC) to a radargram

apply_agc normalizes each sample by the RMS amplitude of its local window.
The window length determines how local the gain is.
After applying AGC, deeper reflections become more visible, although noise may also be amplified.
The plotted radargram now shows both shallow and deep events with balanced amplitudes for interpretation.

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

## Example 2 — Compare radargram before and after AGC

This example loads the SEG Y profile, builds the section matrix, applies AGC with a chosen window, and plots the original and the AGC result in separate figures for direct visual comparison.

```
import numpy as np
import matplotlib.pyplot as plt
from obspy.io.segy.segy import _read_segy

def apply_agc(data, window_len):
    n_samples, n_traces = data.shape
    agc_data = np.zeros_like(data)
    half_window = window_len // 2
    for i in range(n_traces):
        for j in range(n_samples):
            start = max(0, j - half_window)
            end = min(n_samples, j + half_window)
            rms = np.sqrt(np.mean(data[start:end, i]**2)) + 1e-12
            agc_data[j, i] = data[j, i] / rms
    return agc_data

# Load and assemble section
stream = _read_segy("LINE01.sgy", headonly=False)
n_traces = len(stream.traces)
max_len = max(len(tr.data) for tr in stream.traces)

data = np.zeros((max_len, n_traces))
for i, tr in enumerate(stream.traces):
    data[:len(tr.data), i] = tr.data

# Apply AGC
window_length = 50  # samples
agc_data = apply_agc(data, window_length)

# Plot original section
plt.figure(figsize=(12, 6))
plt.imshow(data, cmap="gray", aspect="auto", origin="upper", interpolation="none")
plt.title("Original Section (no gain)")
plt.xlabel("Trace Number")
plt.ylabel("Time Sample")
plt.colorbar(label="Amplitude")
plt.show()

# Plot AGC section
plt.figure(figsize=(12, 6))
plt.imshow(agc_data, cmap="gray", aspect="auto", origin="upper", interpolation="none")
plt.title(f"AGC Applied Section (Window = {window_length} samples)")
plt.xlabel("Trace Number")
plt.ylabel("Time Sample")
plt.colorbar(label="Normalized Amplitude")
plt.show()

```

Explanation:

The first figure shows the unscaled radargram where early strong arrivals can mask deeper reflections.

The second figure shows the AGC result where deeper events are more balanced against near-surface energy.

Use the same color map and plotting parameters so differences are due to AGC only.

## Example 3 — Window length sensitivity study
AGC behavior depends on the window length. This example applies AGC with several window lengths and shows separate figures so students can judge the trade-offs.

```
import numpy as np
import matplotlib.pyplot as plt
from obspy.io.segy.segy import _read_segy

def apply_agc(data, window_len):
    n_samples, n_traces = data.shape
    agc_data = np.zeros_like(data)
    half_window = window_len // 2
    for i in range(n_traces):
        for j in range(n_samples):
            start = max(0, j - half_window)
            end = min(n_samples, j + half_window)
            rms = np.sqrt(np.mean(data[start:end, i]**2)) + 1e-12
            agc_data[j, i] = data[j, i] / rms
    return agc_data

# Load and assemble section (once)
stream = _read_segy("LINE01.sgy", headonly=False)
n_traces = len(stream.traces)
max_len = max(len(tr.data) for tr in stream.traces)

data = np.zeros((max_len, n_traces))
for i, tr in enumerate(stream.traces):
    data[:len(tr.data), i] = tr.data

# Try several window lengths (in samples)
window_list = [20, 50, 100, 200]

for w in window_list:
    agc_w = apply_agc(data, w)

    plt.figure(figsize=(12, 6))
    plt.imshow(agc_w, cmap="gray", aspect="auto", origin="upper", interpolation="none")
    plt.title(f"AGC Section — Window = {w} samples")
    plt.xlabel("Trace Number")
    plt.ylabel("Time Sample")
    plt.colorbar(label="Normalized Amplitude")
    plt.show()

```

Explanation and guidance:

Short windows, for example 20 samples, strongly equalize local amplitudes. Small features pop out, but late-time noise can be amplified.

Intermediate windows around 50 to 100 samples often give a good balance for many GPR datasets.

Long windows such as 200 samples produce smoother results that preserve broader trends but may underemphasize weak reflectors.

A practical rule of thumb is to select a window corresponding to a few to several periods of the dominant frequency:

$$
\text{window\_len} \approx k \times \frac{1}{f_c\,\Delta t}
$$

where f_c is the antenna center frequency and Δt is the sample interval; choose k between roughly three and ten.

::::::
