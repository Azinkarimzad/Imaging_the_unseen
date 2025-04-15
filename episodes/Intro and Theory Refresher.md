---
title: "Intro and Theory Refresher"
teaching: 15
exercises: 5
---

:::::::::::::::::::::::::::::::::::::: 
## Questions

- What is Ground Penetrating Radar (GPR)?
- How is a GPR trace defined?
- How can we read and visualize GPR data using Python?

::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::: 
## Objectives

- Describe the basic principles of GPR.
- Explain what a trace is in GPR data.
- Demonstrate how to load and plot SEG-Y data using `obspy`.

::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::: 
## Keypoints

- GPR uses EM waves to detect subsurface features.
- A trace is a time series of reflections from a point.
- SEG-Y files can be visualized using ObsPy.

::::::::::::::::::::::::::::::::::::::::::::::::

## Introduction

Ground Penetrating Radar (GPR) is a non-invasive geophysical method that uses electromagnetic waves to image the subsurface. It is widely used in engineering, archaeology, environmental studies, and geotechnical investigations.

GPR systems send short electromagnetic pulses into the ground and record the reflections that occur at interfaces of materials with different dielectric properties. The result is a set of time-based signals called *traces* that capture subsurface reflections at a specific location.

In this episode, we'll introduce how to read and visualize a GPR trace from SEG-Y data using Python and the `obspy` library.

:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: instructor

Explain that GPR is similar in concept to seismic reflection but operates at much higher frequencies (typically 10 MHz–2.6 GHz), allowing for finer resolution and shallower depth penetration.

::::::::::::::::::::::::::::::::::::::::::::::::

## What is a Trace?

A trace is a single time series recording of the reflected radar signal at a given antenna position. In SEG-Y files, traces are typically stored in a sequential format with headers and data samples.

Each trace can be thought of as a 1D signal of amplitude over time.

Let's look at a practical example of how to load and visualize a GPR trace using ObsPy.

::::::::::::::::::::::::::::::::::::: challenge 

## Challenge 1: Visualize a Trace

Use the following Python code to read a SEG-Y file and plot the first trace.

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
