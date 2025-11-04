+++
title = "Remote Sensing-Based Mangrove Classification"
date = 2024-02-20
tags = ["Remote Sensing", "Machine Learning", "Google Earth Engine", "Random Forest"]
description = "Developed a mangrove classification model for India using Sentinel-2A multispectral imagery and Random Forest classifier with 99.03% accuracy."
+++

## Overview

Final semester project focused on developing a high-accuracy land cover classification system for mangrove ecosystems using satellite imagery and cloud-based geospatial platforms.

## Challenge

Mangroves serve as vital coastal ecosystems, but their spatial distribution is often poorly mapped due to:
- Persistent cloud cover
- Limited accessibility to remote areas
- Inconsistent data quality

## Solution

### Data Collection Strategy
- **Geo point-based sampling**: Over 1,100 manually labeled points across India's coastal regions
- Used Global Mangrove Watch data for reference
- Separated into mangrove/non-mangrove classes

### Feature Engineering
- **Spectral bands**: B2, B3, B4, B8, B11 from Sentinel-2A
- **Indices**: NDVI (Normalized Difference Vegetation Index), NDWI (Normalized Difference Water Index)
- Multi-spectral analysis for robust classification

### Model Performance
- **Algorithm**: Random Forest Classifier
- **Accuracy**: 99.03% on test data
- **Generalization**: Strong performance on unseen regions

### Deployment
- **Google Earth Engine** tile exports for classified imagery
- **Custom Streamlit web application** for real-time visualization
- Interactive classification by user clicks on map

## Technologies Used
- Google Earth Engine (GEE)
- Sentinel-2A Multispectral Imagery
- Random Forest
- Streamlit
- Python
- Geospatial Analysis
