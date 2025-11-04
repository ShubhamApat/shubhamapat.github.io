+++
title = "College Hotspots Dashboard - Interactive Data Visualization"
date = 2024-05-01
tags = ["Data Visualization", "Dash", "Plotly", "Data Processing", "Flask"]
description = "Built a comprehensive interactive dashboard from scratch using Dash by Plotly, analyzing 3GB of business data around colleges across major Indian cities."
+++

## Overview

Developed an interactive data visualization dashboard from scratch to analyze student-focused businesses around colleges and universities across major Indian cities.

## Dataset
- **Source**: Scraped from Google Maps
- **Size**: 3 GB of comprehensive business data
- **Coverage**: Ahmedabad, Gandhinagar, Mumbai, Pune, Kolkata, Hyderabad, Bangalore
- **Content**: All business information available on Google Maps

## Data Pipeline
1. **Scraping**: Automated data collection from Google Maps
2. **Cleaning**: Data preprocessing and validation
3. **Feature Extraction**: Relevant business attributes
4. **Data Visualization**: Interactive charts and maps

## Dashboard Features

### Interactive Visualizations
1. **ScatterMapBox**
   - City-wise filtering
   - Clickable pincode bubbles
   - Geographic distribution of businesses

2. **Sunburst Chart**
   - Updates based on selected pincode
   - Shows business categories and quantities
   - Hierarchical data representation

3. **Histogram**
   - Top 50 pincodes per city
   - Based on number of businesses
   - Clickable to filter other visualizations

4. **Data Table**
   - Detailed business listings
   - Category filtering
   - Sortable columns

5. **Tree Map**
   - Business category distribution
   - City-wise analysis

### Download Feature
Users can download complete business data for any selected pincode

## Performance Optimization

### Initial Challenge
- **Problem**: 90+ second loading time
- **Cause**: Large 3GB dataset with inefficient processing

### Optimization Techniques
1. **Flask Caching**: Implemented intelligent caching strategies
2. **CSV to Parquet Conversion**: More efficient file format
3. **Data Type Optimization**: Reduced memory footprint
4. **Query Optimization**: Faster data access patterns

### Result
- **Before**: 90+ seconds
- **After**: 1 second
- **Improvement**: 90x speedup

## Technologies Used
- Python
- Dash by Plotly
- Flask
- Pandas
- Data Processing
- Interactive Visualization
- Caching
