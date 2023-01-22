# TPC Signal Processing for the ICARUS T-600 Detector

## I. Introduction

## II. Algorithms

### A. Morphological Filtering

#### 1. Overview

#### 2. Mathematical Morphology

#### 3. Van-Herk Gil-Werman Algorithm

### B. Frequency Filters 

#### 1. Smooth Threshold Fourier Domain Filtering 

#### 2. Mollifiers and the Gibbs Phenomenon

### C. Adaptive Filtering

#### 1. Bilateral Filters

#### 2. Lee Filter and its Variants

### D. Template Matching

#### 1. Hough Transform

#### 2. Probabilistic Hough Transform

#### 3. General Hough-Transform based Methods

### E. Edge Detectors

#### 1. Intensity Gradient Filters 

#### 2. Canny Edge Detector

#### 3. Others

### F. Thresholding

#### 1. Hysteresis Thresholding 

 - Implementation via Fixed-point Iteration of Morphological Dilation
 - Implementation via Union-Find 

##### References 

 - [BBDT](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=5428863)
 - [Connected Components Labeling I](https://federicobolelli.it/pub_files/2017iciap_labeling.pdf)
 - [Connected Components Labeling II](https://www.osti.gov/servlets/purl/929013)
 - Artur Nowakowski and Wladyslaw Skarbek, [*Fast computation of thresholding hysteresis for edge detection*](https://doi.org/10.1117/12.674959), Proc. SPIE 6159, Photonics Applications in Astronomy, Communications, Industry, and High-Energy Physics Experiments IV, 615948 (26 April 2006);


## III. Region of Interest (ROI) Detection Chain

### A. Overview

### B. Parallelization

### C. Evaluation Metrics

#### 1. Fuzzy Similarity Metrics


## IV. Hit Finder

## V. Deconvolution

## VI. 3D Point Matching 

### A. Cluster3D 

## Useful References


 - Young, Gerbrands, and Villet., [*Fundamentals of Image Processsing*]() (2004) : best introduction to image processing algorithms. 
 - Michael Voss, Rafael Asenjo, James Reinders., [*ProTBB: C++ Parallel Programming with Threading Building Blocks*](https://link.springer.com/book/10.1007/978-1-4842-4398-5) (2019) : best introduction and reference to Intel Threading Building Blocks library and general parallel programming. 
 - Numerical Recipe



 
