---
title: "Wrap-up and Discussion"
teaching: 10
exercises: 2
---

:::::: questions
- What are the main processing steps applied to GPR data in this course?  
- How do these steps improve interpretability of radargrams?  
- Which processing choices require the most careful parameter selection?  
- How can Python be used flexibly to test and evaluate different workflows?  
::::::

:::::: objectives
- Summarize the core processing steps learned: visualization, AGC, background removal, time-zero correction, bandpass filtering, and deconvolution.  
- Reflect on how each step alters the data and what geophysical problem it addresses.  
- Encourage critical evaluation of parameters and their impact on results.  
- Prepare learners to design their own GPR processing workflows for new datasets.  
::::::

:::::: keypoints
- GPR traces are raw recordings that must be processed for reliable interpretation.  
- Each processing step—AGC, background removal, time-zero correction, filtering, deconvolution—has a clear physical motivation.  
- Parameters (e.g., filter cutoffs, stabilization constants) control the trade-off between resolution and noise.  
- No single workflow is universal; effective GPR interpretation requires testing, visual inspection, and critical judgment.  
- Python offers a transparent environment for experimenting with and combining different methods.  
