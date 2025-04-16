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

:::::::::::::::::::::::::::::::::::::: Keypoints

- GPR uses EM waves to detect subsurface features.
- A trace is a time series of reflections from a point.
- SEG-Y files can be visualized using ObsPy
- .::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::: introduction

Ground Penetrating Radar (GPR) is a non-invasive geophysical method that uses electromagnetic waves to image the subsurface. It is widely used in engineering, archaeology, environmental studies, and geotechnical investigations.

GPR systems send short electromagnetic pulses into the ground and record the reflections that occur at interfaces of materials with different dielectric properties. The result is a set of time-based signals called *traces* that capture subsurface reflections at a specific location.

In this episode, we'll introduce how to read and visualize a GPR trace from SEG-Y data using Python and the `obspy` library.

:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: 
