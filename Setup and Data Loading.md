---
title: "Setup and Data Loading"
teaching: 20
exercises: 10
---

## Overview

Before diving into subsurface imaging techniques, we need to ensure all required tools and libraries are properly installed and the data is ready to use.

In this episode, you'll learn how to:

- Set up your Python environment
- Install necessary packages (if needed)
- Understand the dataset format
- Load geophysical data (e.g., SEG-Y or GPR data) into Python for processing

---

## Learning Objectives

By the end of this episode, learners will be able to:

- Verify their Python and Jupyter environment is working
- Install and import required libraries such as NumPy and Matplotlib
- Understand the structure of the dataset provided (SEG-Y or other)
- Load and visualize the raw data in a Jupyter notebook

---

## 1. Environment Setup

> ⚠️ You should have Python 3.7+ installed. If not, we recommend [Anaconda](https://www.anaconda.com/) for easy setup.

```bash
conda create -n subsurface python=3.8
conda activate subsurface
conda install numpy matplotlib obspy jupyter
