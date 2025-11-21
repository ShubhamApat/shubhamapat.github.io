+++
title = "Remote Sensing-Based Mangrove Classification"
date = 2025-04-20
tags = ["Remote Sensing", "Machine Learning", "Google Earth Engine", "Random Forest"]
description = "Developed a mangrove classification model for India using Sentinel-2A multispectral imagery and Random Forest classifier with 99.03% accuracy."
[cover]
image="images/mangrove_outcome.png"
+++

## Overview


<img src="images/mangrove_plantation.jpg" | absURL>

Final semester project focused on developing a high-accuracy land cover classification system for mangrove ecosystems using satellite imagery and cloud-based geospatial platforms. The motivation behind this project was a research about mangroves that I was part of, where I learned to fly a drone over mangrove plantations for their monitoring and plantation.

Mangroves serve as vital coastal ecosystems, but their spatial distribution is often poorly mapped due to:
- Persistent cloud cover
- Limited accessibility to remote areas
- Inconsistent data quality

The core objective is to develop a geospatial classification model capable of differentiating
mangrove and non-mangrove classes using high-resolution satellite imagery. The focus
is limited to the Indian subcontinent, where coastal zones are vulnerable to ecological
degradation. The classification model is designed to be scalable, cloud-based, and capable
of handling spatially distributed data.

To manually label and validate training samples using Global Mangrove Watch data Over
570 points were manually marked for each class (mangrove = 1, non-mangrove = 0). The
labeling process was guided by GMW overlays and satellite base maps. Non-mangrove
classes included forests, rivers, wastelands, mudflats, and urban backgrounds to ensure the
classifier learned meaningful feature distinctions.

To compute vegetation indices and spectral bands for model input features
The following indices and bands were used to improve model learning:
-NDVI = (B8 - B4) / (B8 + B4) – to highlight vegetation density
-NDWI = (B3 - B8) / (B3 + B8) – to separate water features
-MNDWI = (B3 - B11) / (B3 + B11) – to enhance mudflat and wetland detection
-Raw bands: B2 (Blue), B3 (Green), B4 (Red), B8 (NIR), B11 (SWIR)

To export the classification result as tile overlays for deployment Since GEE restricts
model API deployment, the final classification image was exported as a map tile overlay
using ee.Image.getMapId(), which allows integration with custom web applications.

To build an interactive click-to-classify frontend using Streamlit and Folium A responsive web application was created using Streamlit with embedded Folium maps. Key
functionalities include:
-Interactive satellite and terrain map views
-Real-time click-to-classify feature using EE classification values
-Visualization of mangrove (green) and non-mangrove (red) areas
-Area estimation using pixel-wise aggregation of classified data

**Study Area and Dataset Selection**

The classification was geographically focused on the coastal regions of India, encompassing
known mangrove ecosystems such as:

- Sundarbans
- Gulf of Khambhat
- Gulf of Mannar
- Pichavaram
- Andaman & Nicobar Islands

These areas were selected based on their ecological significance and confirmed mangrove
presence using Global Mangrove Watch (GMW) overlays.
The primary dataset used was Sentinel-2A Level-2A Surface Reflectance imagery. 

<img src="/images/mangrove_data_sampling.png">

For Data Sampling, a total of ˜570 points per class were manually marked.

>Mangrove (class = 1): Points placed over dense mangrove patches, verified with
GMW overlays and true-color images.

>Non-mangrove (class = 0): Points included:
- Open water
- Urban areas
- Forest
- Wetlands
- Barren land

>Points were stored using custom Google Earth Engine scripts along with spatial
metadata.

Each point was enriched with the following features:
Spectral Bands

- B2 (Blue, 10m)
- B3 (Green, 10m)
- B4 (Red, 10m)
- B8 (Near Infrared, 10m)
- B11 (Short Wave Infrared, 20m, resampled)

Vegetation Indices
- NDVI = B8−B4/B8+B4 (indicates chlorophyll and biomass)
- NDWI = B3−B8/B3+B8 (highlights water bodies)

This resulted in a 7-dimensional feature vector for each point: [B2, B3, B4, B8, B11,
NDVI, NDWI].

**Model Training**

A Random Forest classifier was trained using the smileRandomForest module in GEE.

<img src="/images/gee-training.png">
<img src="/images/gee.png">
Figure 3.4.1: Screenshot of the GEE script used for training the Random Forest classifier
and exporting results. The console output on the right summarizes training-validation
split, dataset balance, and feature selection used for classification.

- Train-test split:
    - 70% training
    - 30% testing
- Advantages of RF:
    - Resistant to overfitting
    - Handles non-linear relationships
    - Offers feature importance analysis
The model achieved an accuracy of 99.03%, validated using test points near the Mumbai
coastline.

**Model Deployment**

Deployed using Streamlit, Google Earth Engine (GEE), and Folium for interactive mapping Initialized using GEE Python API, Tile layered generated with getMapId( ).

I added Click & Classify functionality allows user to click a point on map and it captures Latitude and Longitude, how does it work? so Lats & Longs are converted to GEE point and Classification value (0 or 1) gets fetched using reduceRegion( ).

<left>
<img src="/images/mangrove_deployment1.png", width=1200>
</left>
<right>
<img src="/images/mangrove_deployment2.png", width=1200>
</right>

**Overall Architecture**

<img src="/images/mangrove_architecture.png">

**Outcome**

The strong outcome validates the effectiveness of the supervised learning approach, the
geo-point sampling strategy, and the value of vegetation indices like NDVI and NDWI in
distinguishing mangroves from surrounding land covers.

<img src="/images/mangrove_outcome.png">
(a) Global Mangrove Watch
Reference
(b) Manually Labeled GeoPoints
(c) Final Classified Overlay
Figure 4.1.2: Comparison of classification process: 
(a) Reference from Global Mangrove
Watch, 
(b) manually annotated geo-points used for training the classifier, and 
(c) final
classification result overlaid as tile using the trained model.