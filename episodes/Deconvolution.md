---
title: "Deconvolution"
teaching: 10
exercises: 2
---

:::::: questions
- What is deconvolution in the context of GPR processing?  
- Why do recorded GPR signals differ from the true earth response?  
- How does spectral deconvolution recover sharper reflections?  
- What is the role of the stabilization parameter in deconvolution?  
- How can we evaluate the effect of deconvolution on traces and sections?  
::::::

:::::: objectives
- Define deconvolution and explain its purpose in GPR signal processing.  
- Implement frequency-domain (spectral) deconvolution on individual traces and full sections.  
- Assess the impact of stabilization on the recovered signal.  
- Compare radargrams before and after deconvolution to evaluate reflector clarity.  
- Use single-trace examples to illustrate wavelet shortening.  
::::::

:::::: keypoints
- GPR records are a convolution of the source wavelet and subsurface reflectivity.  
- Deconvolution aims to remove the source wavelet, recovering a spike-like reflectivity.  
- Spectral deconvolution divides the spectrum by its amplitude, with stabilization to avoid noise blow-up.  
- Spiking deconvolution sharpens arrivals but requires careful parameter tuning.  
- Stabilization controls the trade-off between resolution gain and noise amplification.  
::::::

:::::: introduction
Ground Penetrating Radar (GPR) does not record the true reflectivity of the subsurface. Instead, each trace
represents the **convolution** of the antenna source wavelet with reflections from interfaces in the ground.
This convolution blurs the reflectivity, producing wavelets rather than sharp spikes.

**Deconvolution** is a signal processing step that attempts to undo this effect by compressing the wavelet
into something closer to a spike. The goal is improved temporal resolution, so that thin layers and closely
spaced reflectors can be better distinguished.

A common approach is **spectral deconvolution**. In the frequency domain, the observed trace spectrum is
divided by its amplitude spectrum (with a small stabilization factor added to prevent division by near-zero
values). The phase is preserved. After inverse Fourier transform, the resulting trace shows sharpened
reflections.

The stabilization constant is critical: too small and noise dominates; too large and resolution gains are
lost. In practice, deconvolution is applied trace-by-trace across the section and results are compared to
the original radargram to confirm improved clarity without excessive noise.

::::::

## Example 1 — Spectral Deconvolution on a 2D GPR Section

The following code implements spectral deconvolution for each trace of a SEG-Y GPR section and compares
the radargram before and after processing.

```python
import numpy as np
import matplotlib.pyplot as plt
from obspy.io.segy.segy import _read_segy
from scipy.fft import rfft, irfft, rfftfreq

def spectral_decon(trace, fs, stab=0.01):
    """
    Apply frequency-domain (spectral) deconvolution to a trace.
    """
    n = len(trace)
    f = rfftfreq(n, d=1/fs)
    spectrum = rfft(trace)
    amp = np.abs(spectrum)
    phase = spectrum / (amp + 1e-12)  # preserve phase

    # Invert amplitude spectrum with stabilization
    inverse_amp = 1 / (amp + stab * np.max(amp))
    decon_spectrum = inverse_amp * spectrum

    # Back to time domain
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

# --- Sampling frequency ---
try:
    dt_us = stream.binary_file_header.sample_interval_in_microseconds
    if dt_us == 0 or dt_us > 10:
        print(f"Header sample interval {dt_us} µs seems invalid. Overriding to 0.5 µs.")
        dt_us = 0.5
except:
    print("Sample interval not found. Using default of 0.5 µs.")
    dt_us = 0.5

fs = 1e6 / dt_us
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
````
Explanation:

Fourier transform each trace → separate amplitude and phase.

Invert amplitude spectrum, with stabilization constant to prevent division by zero.

Recombine with phase and inverse transform → sharpened trace.

Apply across all traces to produce deconvolved section.

Compare radargrams before and after → reflections appear narrower and better resolved.

## Example 2 — Spiking Deconvolution on a Single Trace

Spiking deconvolution uses the autocorrelation of a trace to estimate its wavelet and design a filter that compresses it into a spike.

```
from scipy.linalg import toeplitz, solve_toeplitz

def spiking_decon(trace, stab=0.1, filter_length=30):
    """
    Spiking deconvolution using Wiener filter design.
    """
    # Autocorrelation
    autocorr = np.correlate(trace, trace, mode="full")
    autocorr = autocorr[autocorr.size // 2:]

    # Toeplitz system
    R = toeplitz(autocorr[:filter_length])
    r = np.zeros(filter_length)
    r[0] = 1.0  # desired spike

    # Stabilization
    R += stab * np.eye(filter_length)

    # Solve Wiener-Hopf equations
    f = np.linalg.solve(R, autocorr[:filter_length])
    decon = np.convolve(trace, f, mode="same")
    return decon

# Apply to one representative trace
trace_idx = n_traces // 2
original_trace = data[:, trace_idx]
decon_trace = spiking_decon(original_trace, stab=0.1, filter_length=30)

plt.figure(figsize=(10, 4))
plt.plot(original_trace, label="Original", alpha=0.7)
plt.plot(decon_trace, label="Spiking Deconvolved", linestyle="--")
plt.title(f"Trace {trace_idx} before and after Spiking Deconvolution")
plt.xlabel("Sample")
plt.ylabel("Amplitude")
plt.legend()
plt.grid(True)
plt.show()

```
Explanation

Estimate wavelet from autocorrelation of the trace.

Solve Wiener equations to design filter that compresses wavelet into a spike.

Convolve filter with trace → spike-like reflectivity series.

Useful for single traces or small windows where spectral method is unstable.

## Example 3 — Effect of Stabilization on Spectral Deconvolution

The stabilization constant (stab) controls the balance between resolution gain and noise. This example applies different values and compares results.

```
stab_values = [0.01, 0.1, 1.0]

fig, axs = plt.subplots(1, 3, figsize=(18, 6), sharey=True)

for i, stab in enumerate(stab_values):
    decon_data_test = np.zeros_like(data)
    for j in range(n_traces):
        decon_data_test[:, j] = spectral_decon(data[:, j], fs=fs, stab=stab)
    axs[i].imshow(decon_data_test, cmap="gray", aspect="auto", origin="upper")
    axs[i].set_title(f"Spectral Deconvolution\nstab={stab}")
    axs[i].set_xlabel("Trace")
    if i == 0:
        axs[i].set_ylabel("Sample")

plt.tight_layout()
plt.show()

```

Explanation

Small stab (0.01): strong wavelet compression but high noise amplification.

Moderate stab (0.1): balanced improvement; often practical.

Large stab (1.0): stable but reflections less sharpened.

Students can see the trade-off and understand why stabilization is critical.
