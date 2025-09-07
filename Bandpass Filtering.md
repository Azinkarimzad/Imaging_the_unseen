---
title: "Bandpass Filtering"
teaching: 10
exercises: 2
---

:::::: questions
- What is a bandpass filter in the context of GPR signal processing?
- Why apply a bandpass filter to GPR traces before interpretation?
- How do sampling frequency and Nyquist frequency constrain the choice of cutoffs?
- How does zero-phase filtering affect waveform shape compared with causal filtering?
- How can frequency spectra guide the selection of passband limits?
::::::

:::::: objectives
- Define bandpass filtering and relate it to antenna bandwidth and recorded noise.
- Compute sampling and Nyquist frequencies from SEG Y headers and validate cutoffs.
- Apply a zero-phase Butterworth bandpass to a 2D GPR section without altering your code.
- Compare radargrams and trace spectra before and after filtering to assess effectiveness.
- Describe trade-offs when tightening or relaxing the passband.
::::::

:::::: keypoints
- GPR data often contain low-frequency drift and high-frequency noise; a bandpass suppresses both.
- Cutoffs must lie within (0, Nyquist); selecting them near the antenna’s effective bandwidth improves SNR.
- Zero-phase forward-backward filtering (filtfilt) preserves arrival timing and polarity.
- Overly narrow passbands may distort wavelets or remove useful signal.
- Frequency spectra of representative traces help justify passband choices quantitatively.
::::::

:::::: introduction
GPR antennas radiate broadband pulses with finite bandwidth. Recorded sections typically include
low-frequency components from coupling or instrumentation and high-frequency noise from electronics
or environment. A **bandpass filter** retains a chosen frequency band while attenuating energy below
and above it. In practice, cutoffs should respect the **Nyquist frequency** and bracket the dominant
signal band suggested by the antenna type and the observed spectra.

This lesson applies a **zero-phase Butterworth bandpass** to a 2D section. Zero-phase filtering via
forward-backward application avoids phase distortion, so reflector timing is preserved. We compute
sampling and Nyquist frequencies from SEG Y headers, apply the filter trace-by-trace, and evaluate the
result using both images and amplitude spectra.
::::::

## Example 1 — Zero-phase Butterworth bandpass on a 2D GPR section

The code below reads a SEG Y profile, derives the sampling frequency, applies a fourth-order
Butterworth bandpass with cutoffs as percentages of Nyquist, and compares sections and spectra
before and after filtering.

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
Explanation

Design the filter
The cutoffs are normalized by Nyquist to design a fourth-order Butterworth bandpass. This shape is smooth
in magnitude and, with forward-backward application, zero-phase in the output.

Validate cutoffs
The code checks that both cutoffs fall strictly between zero and Nyquist. Invalid values raise a clear error.

Apply zero-phase filtering
Each trace is filtered with filtfilt, which runs the filter forward and backward to cancel phase shift while
doubling the effective order.

Compute sampling parameters
The sample interval from the SEG Y binary header gives fs in hertz and nyquist = fs/2. These determine the
allowable band.

Inspect results in time and frequency
Side-by-side images show structural changes after filtering. A frequency spectrum of a representative trace
quantifies which bands were attenuated or retained.

Interpretation
Suppressing very low and very high frequencies clarifies reflections within the passband. If reflections weaken
or look distorted, the passband is too narrow; relax the cutoffs toward the data’s dominant band.


## Example 2 — Comparing Multiple Passbands

It is often unclear which cutoff frequencies best enhance reflections. In this example, we apply three different passbands and compare their effects on both the radargrams and frequency spectra.

```python
# Define three passband settings as fractions of Nyquist
passbands = [
    (0.02 * nyquist, 0.80 * nyquist),  # wide band
    (0.05 * nyquist, 0.50 * nyquist),  # medium band
    (0.10 * nyquist, 0.30 * nyquist)   # narrow band
]

fig, axs = plt.subplots(1, 3, figsize=(18, 6), sharey=True)

for i, (lowcut, highcut) in enumerate(passbands):
    filtered = bandpass_filter(data, lowcut, highcut, fs)
    axs[i].imshow(filtered, cmap="gray", aspect="auto", origin="upper")
    axs[i].set_title(f"{lowcut/1e6:.2f}–{highcut/1e6:.2f} MHz")
    axs[i].set_xlabel("Trace")
    if i == 0:
        axs[i].set_ylabel("Sample")

plt.tight_layout()
plt.show()

# Plot spectra for the middle trace across all three passbands
plt.figure(figsize=(10, 5))
for (lowcut, highcut) in passbands:
    filtered = bandpass_filter(data, lowcut, highcut, fs)
    plot_frequency_spectrum(filtered[:, middle_trace_idx], fs,
                            title=f"{lowcut/1e6:.2f}–{highcut/1e6:.2f} MHz")
plot_frequency_spectrum(data[:, middle_trace_idx], fs, title="Unfiltered")
plt.title("Frequency Spectra for Different Passbands")
plt.legend()
plt.tight_layout()
plt.show()

```
Explanation

Wide passband keeps nearly all signal but lets noise through.

Medium passband balances suppression of noise with preservation of reflections.

Narrow passband reduces noise strongly but can distort wavelets and weaken hyperbolas.

Comparing spectra confirms which frequencies remain in each case.

## Example 3 — Filtering Individual Traces for QC

Before filtering the entire dataset, it is useful to test the filter on a few representative traces. This example applies the same bandpass to three traces and compares them with the originals.

```
# Select three traces across the profile
trace_indices = [n_traces // 4, n_traces // 2, 3 * n_traces // 4]

lowcut = 0.05 * nyquist
highcut = 0.95 * nyquist

plt.figure(figsize=(12, 8))

for i, idx in enumerate(trace_indices, 1):
    original = data[:, idx]
    filtered = bandpass_filter(data[:, [idx]], lowcut, highcut, fs).flatten()
    
    plt.subplot(3, 1, i)
    plt.plot(original, label="Original", alpha=0.6)
    plt.plot(filtered, label="Filtered", linestyle="--")
    plt.title(f"Trace {idx} before and after bandpass")
    plt.xlabel("Sample")
    plt.ylabel("Amplitude")
    if i == 1:
        plt.legend()

plt.tight_layout()
plt.show()

```
Explanation

Original vs filtered traces plotted directly help confirm that filtering removes unwanted frequencies without destroying arrivals.

Students can visually check if the wavelet shape is preserved.

This is a simple quality-control step before applying the filter to the full dataset.
