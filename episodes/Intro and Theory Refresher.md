---
title: "Intro and Theory Refresher"
teaching: 15
exercises: 5
---

:::::: questions

- What are the core components of a GPR system?

- How does GPR work to detect subsurface changes?

- What does a GPR trace represent?

- How can we process and visualize SEG-Y GPR data using Python?

- What are the core hardware and data-logging components of a GPR system, and how do they interact during acquisition?

- Through which physical contrasts does GPR detect subsurface changes, and how do antenna frequency and bandwidth control resolution and depth?

- What does a single GPR trace represent in time and in depth, and how is a radargram constructed from adjacent traces?

- How can we read, inspect metadata, and visualize SEG-Y GPR data in Python using ObsPy without modifying the raw samples?

::::::

:::::: objectives

- Understand the structure and functionality of a GPR system.

- Explain the concept of GPR signal reflection and trace acquisition.

- Interpret what a single GPR trace and a 2D radargram represent.

- Demonstrate loading and visualizing GPR SEG-Y data using Python and obspy.

- Identify the console, antenna, and encoder roles in a modern GPR system and relate them to sampling, time window, and spatial sampling.

- Explain reflection generation from dielectric permittivity contrasts and the implications for amplitude, polarity, and travel time.

- Interpret a trace as a one-dimensional time series and a radargram as a two-dimensional section built by lateral stacking of traces.

- Demonstrate loading SEG-Y, inspecting basic headers, and plotting trace-level and section-level views using ObsPy and matplotlib.
::::::

:::::: keypoints

- GPR uses EM waves to detect subsurface features.
- A trace is a time series of reflections from a point.
- SEG-Y files can be visualized using ObsPy.
- GPR emits broadband electromagnetic pulses; reflections arise at contrasts in relative permittivity, conductivity, or magnetic permeability.
- A GPR trace is amplitude versus two-way travel time at a single surface position; a radargram is a collection of traces along a profile.
- SEG-Y is a common container for GPR; ObsPy can read variable-length traces and expose headers needed for plotting and basic QC.
- Antenna frequency controls depth–resolution trade-off: lower frequency penetrates deeper with coarser resolution; higher frequency resolves finer targets at shallower depths.

::::::

:::::: introduction

## What is geophysics?
In the broadest sense, the science of geophysics is the application of physics to investigations of the Earth, 
Moon and planets. The subject is thus related to astronomy.
To avoid confusion, the use of physics to study the interior of the Earth, from land surface to the inner 
core, is known as solid earth geophysics.
Applied geophysics’ covers everything from experiments to determine the thickness of the crust (which is 
important in hydrocarbon exploration) to studies of shallow structures for engineering site investigations, 
exploring for groundwater and for minerals and other economic resources, to trying to locate narrow 
mine shafts or other forms of buried cavities, or the mapping of archaeological remains, or locating buried 
pipes and cables – but where in general the total depth of investigation is usually less than 100 m.

Ground penetrating radar (GPR) is now a well-accepted geophysical technique. The method uses radio 
waves to probe “the ground”. 
In its earliest inception, GPR was primarily applied to natural geologic materials. Now GPR is equally well 
applied to a host of other media such as wood, concrete, and asphalt.
The most common form of GPR measurements deploys a transmitter and a receiver in a fixed geometry, 
which are moved over the surface to detect reflections from subsurface features. Ground penetrating 
radar systems are conceptually simple; the objective is to measure field amplitude versus time after 
excitation. The heart of a GPR system is the timing unit, which controls the generation and detection of 
signals. Most GPRs operate in the time domain. 
GPRs are normally characterized by their center frequency. The common range varies between 50MHz to 
2.5 GHz. 

Most GPR consist of three components: a console, an antenna, and an encoder. The first two are 
mandatory. The third is typical for 99 percent of GPR users that work in the utility locating, civil 
engineering, concrete scanning, and archaeology industries. The console is the brains of the system. This 
data logger communicates with both the encoder and the antenna to initiate a signal and record the 
responses. The antenna is where the GPR signal is produced. The encoder, also referred to as a survey 
wheel, controls for locational accuracy of the GPR data. As the wheel turns a fraction of a turn, the console 
will initialize the antenna. Most GPR antenna actually consist of a transmitter and a receiver, and once it 
is initialized by the console, the transmitter will produce the signal. The receiver will record responses for 
a defined amount of time and once that “time window” has reached its maximum time, the console will 
terminate the receiver from recording any more data. 

The GPR antenna produces an electromagnetic pulse at the ground surface. This pulse, which is in the
form of a wave, travels into the subsurface and will continue until one of two things happen: the signal 
strength totally decays, or the wave encounters a change in the physical properties of the subsurface 
material. This change can be a small target such as a buried pipe or a large geological layer such a bedrock. 
When the wave encounters a change in material properties, some of the wave’s energy will reflect back 
to the ground surface and some will transmit further into the ground if any energy is left. This produces a 
one-dimensional view just below the antenna. If multiple of these one-dimensional views are collected 
next to each other (i.e. the GPR is pushed along the ground surface in a line), then a twodimensional view 
of the subsurface will be generated by the GPR. This two-dimensional view is equivalent to looking at the 
wall of an excavation trench. It is a vertical profile of the subsurface directly beneath the path that the 
GPR was pushed.





![Schematic of a GPR setup]({{ site.baseurl }}/site/GPR-method-3.1.jpg){alt='Diagram of a GPR system showing console, antenna, and encoder.'}




In this episode we will read and visualize a GPR trace from SEG-Y data using Python and the ObsPy library. The aim is rapid inspection for quality control and for building intuition on trace and section appearance prior to any processing.

Example data context

Assume LINE01.sgy is a single profile recorded with an air-coupled or ground-coupled system. The file may contain variable-length traces. The sampling interval, number of samples, and textual or binary headers carry acquisition metadata.

Example 1: Plot a single trace

Example 2: Build a radargram

Example 3: Inspect metadata

Example 4: Add a time axis

Example 5: Apply a gain

## Example 1 — Plot a single trace for a quick look

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

What this code does, step by step:

Import readers and plotting: brings in ObsPy’s private SEG-Y reader and matplotlib.

Read the SEG-Y stream: _read_segy parses the file into a SEGYFile-like object with a .traces list. headonly=False ensures sample arrays are read, not just headers.

Select the first trace: stream.traces[0].data returns a one-dimensional array of amplitudes for the first position along the line.

Count traces: len(stream.traces) gives the number of positions acquired.

Plot the raw samples: the x-axis is sample index, not time, because a sample interval has not been applied; the y-axis is recorded amplitude in instrument units.

Interpretation note: a clear early-time direct wave or ring-down may dominate the first part of the trace; deeper reflections appear later in the sample index.

Add-on checks on performance:

Confirm whether amplitudes are signed and whether clipping is visible.

convert sample index to time using the trace header sample interval.

Look for saturation or DC drift that might require dewow in further processing.

## Example 2 — Build a simple radargram image from all traces

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
What this code does, step by step:

Read the file and collect traces: same reader as Example 1.

Determine array geometry: n_traces is the lateral dimension; max_len is the longest trace length.

Allocate a section matrix: data is a two-dimensional array with shape (time samples, trace number).

Populate the matrix: for each trace, copy its samples into the corresponding column; shorter traces remain zero-padded at late times.

Display the radargram: imshow renders amplitude as image intensity.

origin="upper" places early time at the top of the image.

aspect="auto" allows traces to fill the figure width.

cmap="gray" uses a neutral colour map suited to signed amplitudes.

interpolation="none" avoids smoothing that could hide thin reflectors.

Interpretation note: continuous subhorizontal events suggest layering; symmetric hyperbolas indicate compact objects. Polarity is arbitrary at this stage and depends on plotting conventions.

Add-on checks on performace:

Verify whether zero padding at late times creates a visible edge; in later processing you may want to trim to a common time window.

If position ticks are uneven, later steps should remap traces to true distance before velocity analysis.

To relate time samples to actual time, multiply sample index by the per-trace sample interval from headers.

## Example 3 — Inspect acquisition metadata
SEG-Y files store much more than amplitudes. Each trace contains a header with information such as the number of samples, sample interval, and trace sequence number. Reading these values is a first quality-control step.

```
from obspy.io.segy.segy import _read_segy

stream = _read_segy("LINE01.sgy", headonly=True)

# Access the first trace header
tr0 = stream.traces[0]

print("Number of samples:", tr0.header.number_of_samples)
print("Sample interval (microseconds):", tr0.header.sample_interval_in_ms_for_this_trace)
print("Trace sequence number:", tr0.header.trace_sequence_number_within_line)

```
Explanation:

headonly=True avoids loading amplitudes and reads only headers.

number_of_samples × sample_interval gives the total time window.

Trace sequence numbers are useful for checking completeness of the dataset.

Inspecting headers ensures that later visualizations are correctly scaled.

## Example 4 — Adding a time axis to a trace

Plots by sample index are useful, but a time axis makes the physics clearer. We can build a time vector from the sample interval stored in the header.
```
import matplotlib.pyplot as plt
from obspy.io.segy.segy import _read_segy
import numpy as np

stream = _read_segy("LINE01.sgy", headonly=False)
tr0 = stream.traces[0]

# Extract sample interval (microseconds) and build time axis
dt = tr0.header.sample_interval_in_ms_for_this_trace  # in microseconds
n_samples = len(tr0.data)
time = np.arange(n_samples) * dt * 1e-6  # convert to seconds

# Plot trace with time axis
plt.figure(figsize=(6, 4))
plt.plot(time, tr0.data)
plt.title("First Trace with Time Axis")
plt.xlabel("Two-way travel time [s]")
plt.ylabel("Amplitude")
plt.grid(True)
plt.show()
```
Explanation:

The x-axis now shows two-way travel time in seconds.

This connects the data to subsurface depth, once a velocity model is assumed.

Early samples correspond to shallow reflections; later samples correspond to deeper features.

## Example 5 — Applying a simple gain

Near-surface reflections are often strong, while deeper reflections appear weak. A gain function can emphasize later arrivals. This example applies a simple exponential gain to one trace.

```
import numpy as np
import matplotlib.pyplot as plt
from obspy.io.segy.segy import _read_segy

stream = _read_segy("LINE01.sgy", headonly=False)
tr0 = stream.traces[0]

# Build sample index array
n_samples = len(tr0.data)
time = np.arange(n_samples)

# Apply exponential gain
gain = np.exp(time / (0.2 * n_samples))  # adjust denominator to control strength
trace_gain = tr0.data * gain

# Compare original and gained traces
plt.figure(figsize=(6, 4))
plt.plot(time, tr0.data, label="Original")
plt.plot(time, trace_gain, label="With exponential gain", alpha=0.7)
plt.title("Trace with and without Gain")
plt.xlabel("Sample Index")
plt.ylabel("Amplitude")
plt.legend()
plt.grid(True)
plt.show()
```
Explanation:

The exponential gain increases amplitudes at later times.

This helps reveal weak deeper events that might otherwise be invisible.

The trade-off is that noise at late times is also amplified.

Students learn the importance of balancing clarity and noise.
::::::
