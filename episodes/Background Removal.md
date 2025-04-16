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






::::::
