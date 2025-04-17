---
title: "Intro and Theory Refresher"
teaching: 15
exercises: 5
---

:::::: questions

- What is Geophysics?
- What is Ground Penetrating Radar (GPR)?
- How is a GPR trace defined?
- How can we read and visualize GPR data using Python?

::::::

:::::: objectives

- Describe the basic principles of GPR.
- Explain what a trace is in GPR data.
- Demonstrate how to load and plot SEG-Y data using `obspy`.

::::::

:::::: keypoints

- GPR uses EM waves to detect subsurface features.
- A trace is a time series of reflections from a point.
- SEG-Y files can be visualized using ObsPy.

::::::

:::::: introduction

## What is geophysics?
In the broadest sense, the science of geophysics is the application of physics to investigations of the Earth, 
Moon and planets. The subject is thus related to astronomy.
To avoid confusion, the use of physics to study the interior of the Earth, from land surface to the inner 
core, is known as solid earth geophysics.


Ground Penetrating Radar (GPR) is a non-invasive geophysical method that uses electromagnetic waves to image the subsurface. It is widely used in engineering, archaeology, environmental studies, and geotechnical investigations.

GPR systems send short electromagnetic pulses into the ground and record the reflections that occur at interfaces of materials with different dielectric properties. The result is a set of time-based signals called *traces* that capture subsurface reflections at a specific location.

In this episode, we'll introduce how to read and visualize a GPR trace from SEG-Y data using Python and the `obspy` library.

To visualize Ground Penetrating Radar (GPR) data, we can use the `obspy` library, which supports the SEG-Y format.

Below is a simple example of how to load and display a single GPR trace:

```python
from obspy.io.segy.segy import _read_segy
import matplotlib.pyplot as plt

# Load SEG-Y using ObsPy (handles variable-length traces)
stream = _read_segy("LINE01.sgy", headonly=False)

# Access trace data
trace0 = stream.traces[0].data
n_traces = len(stream.traces)

# Plot first trace
plt.figure(figsize=(6, 4))
plt.plot(trace0)
plt.title("First Trace from LINE01.sgy")
plt.xlabel("Sample Index")
plt.ylabel("Amplitude")
plt.grid(True)
plt.show()
```



```python
import matplotlib.pyplot as plt
from obspy.io.segy.segy import _read_segy
import numpy as np

# Load SEG-Y file
segy_file = "LINE01.sgy"
stream = _read_segy(segy_file, headonly=False)

# Number of traces and max trace length
n_traces = len(stream.traces)
max_len = max(len(tr.data) for tr in stream.traces)

# Fill 2D array with seismic data
data = np.zeros((max_len, n_traces))
for i, tr in enumerate(stream.traces):
    data[:len(tr.data), i] = tr.data

# Plot
plt.figure(figsize=(12, 6))
plt.imshow(data, cmap="gray", aspect="auto", origin="upper", interpolation="none")
plt.title("Seismic Section (LINE01.sgy)")
plt.xlabel("Trace Number")
plt.ylabel("Time Sample")
plt.colorbar(label="Amplitude")
plt.show()

```


::::::
