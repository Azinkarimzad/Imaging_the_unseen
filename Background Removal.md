---
title: "Background Removal"
teaching: 10
exercises: 2
---

:::::: questions

- What is background removal in GPR data processing?

- Why is background removal needed for clearer interpretation?

- How can background removal be implemented programmatically?  

::::::

:::::: objectives

- Define the concept of background in GPR sections and explain its origin.
   
- Demonstrate how to remove background by subtracting the average trace.

- Evaluate the effect of background removal on data clarity and reflector visibility. 

::::::

:::::: keypoints

- Background in GPR sections often comes from consistent system responses, coupling effects, or horizontal banding.

- Removing the background enhances true reflections and reduces horizontal noise.

- A simple method is subtracting the mean trace across all traces. 

::::::

:::::: introduction

Ground Penetrating Radar (GPR) data often contain **background noise** that is unrelated to true subsurface reflections.  
This background usually appears as horizontal banding or low-frequency components repeated across all traces.  
It originates from several sources:  
- the direct wave and antenna ringing,  
- system electronics,  
- consistent ground coupling responses.  

If left untreated, this background can mask weak reflections, especially deeper ones.  

**Background removal** is a common preprocessing step that improves the visibility of hyperbolas and stratigraphic features.  
The simplest approach is to compute the **average trace** across all traces in a section and subtract it from each individual trace.  
This removes energy that is consistent everywhere (the "background") while preserving local variations that indicate subsurface features.  


## Example 1 — Removing background by subtracting the mean trace

```python
# Background removal by subtracting mean trace
def remove_background(data):
    background = np.mean(data, axis=1, keepdims=True)
    return data - background

# Apply it on your GPR section
data_bg_removed = remove_background(agc_result)

# Plot
plt.figure(figsize=(12, 6))
plt.imshow(data_bg_removed, cmap="gray", aspect="auto", origin="upper")
plt.title("GPR Section with Background Removed")
plt.xlabel("Trace Number")
plt.ylabel("Time Sample")
plt.colorbar(label="Amplitude")
plt.show()
```
Explanation:

Define the function

np.mean(data, axis=1, keepdims=True) computes the average amplitude across all traces, at each time sample.

The result is a "mean trace" representing the background that is common to all traces.

keepdims=True preserves the two-dimensional shape so subtraction works cleanly.

Subtract the background

data - background removes this mean trace from each column (trace) in the data section.

What remains are relative deviations, i.e. actual subsurface reflections.

Apply to processed section

In this case, agc_result is the section after Automatic Gain Control.

Applying background removal after AGC makes weaker reflectors clearer.

Plot

imshow shows the new radargram with background removed.

Horizontal banding should be reduced, and hyperbolic events should stand out.

## Example 2 — Visual comparison: before vs after background removal

This example shows the original section and the background-removed section, so learners can judge the effect directly.
```
import numpy as np
import matplotlib.pyplot as plt

def remove_background(data):
    # Estimate background as the mean across traces at each time sample
    background = np.mean(data, axis=1, keepdims=True)
    # Subtract the background from every trace
    return data - background, background

# Assume `agc_result` is your AGC-processed section with shape [samples, traces]
data_bg_removed, estimated_background = remove_background(agc_result)

# Plot original section
plt.figure(figsize=(12, 5))
plt.imshow(agc_result, cmap="gray", aspect="auto", origin="upper", interpolation="none")
plt.title("Original Section (after AGC, before background removal)")
plt.xlabel("Trace Number")
plt.ylabel("Time Sample")
plt.colorbar(label="Amplitude")
plt.show()

# Plot background-removed section
plt.figure(figsize=(12, 5))
plt.imshow(data_bg_removed, cmap="gray", aspect="auto", origin="upper", interpolation="none")
plt.title("Section after Background Removal (mean trace subtraction)")
plt.xlabel("Trace Number")
plt.ylabel("Time Sample")
plt.colorbar(label="Amplitude")
plt.show()

# Optional: visualize the estimated background (as an image)
plt.figure(figsize=(12, 5))
plt.imshow(np.repeat(estimated_background, agc_result.shape[1], axis=1),
           cmap="gray", aspect="auto", origin="upper", interpolation="none")
plt.title("Estimated Background (mean across traces, repeated laterally)")
plt.xlabel("Trace Number")
plt.ylabel("Time Sample")
plt.colorbar(label="Amplitude")
plt.show()
```
Explanation:

The estimated background is the mean across traces at each time sample; repeating it laterally shows the component being removed.

Comparing the two sections highlights how horizontal banding is reduced and diffractive features become clearer.

## Example 3 — Robust background removal using median trace
Mean subtraction can be sensitive to a few large amplitudes. A median is more robust when strong reflectors or outliers are present. This example subtracts the per-sample median across traces.
```
import numpy as np
import matplotlib.pyplot as plt

def remove_background_median(data):
    # Estimate background as the median across traces at each time sample (robust to outliers)
    background_med = np.median(data, axis=1, keepdims=True)
    # Subtract the median background from every trace
    return data - background_med, background_med

# Apply robust background removal
data_bg_removed_med, estimated_background_med = remove_background_median(agc_result)

# Plot result
plt.figure(figsize=(12, 5))
plt.imshow(data_bg_removed_med, cmap="gray", aspect="auto", origin="upper", interpolation="none")
plt.title("Section after Background Removal (median trace subtraction)")
plt.xlabel("Trace Number")
plt.ylabel("Time Sample")
plt.colorbar(label="Amplitude")
plt.show()
```
Explanation:

Median across traces at each time sample suppresses persistent horizontal components while resisting the influence of a few large events.

Resulting sections often show cleaner backgrounds when strong isolated reflectors are present.

::::::
