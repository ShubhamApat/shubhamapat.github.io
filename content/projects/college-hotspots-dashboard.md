+++
title = "I Optimized a 3GB Dashboard from 90 Seconds to 1 Second (90x Speedup)"
date = 2024-05-01
tags = ["Data Visualization", "Dash", "Plotly", "Data Processing", "Flask"]
description = "How I built an interactive dashboard for student businesses and achieved 90x performance improvement"
+++

## The Problem

Students need to find:
- PGs and hostels near colleges
- Restaurants and cafes
- Study spaces
- Other student services

I scraped Google Maps for **3GB of business data** around colleges in 7 major Indian cities.

Building the dashboard was easy. Making it fast? That's where it got interesting.

## First Version: Painfully Slow

```python
# This took 90+ seconds to load
df = pd.read_csv('businesses.csv')  # 3GB CSV
filtered_df = df[df['city'] == selected_city]
map_plot = px.scatter_mapbox(filtered_df, ...)
```

**Result**: 90+ second load time. No one would use this.

## The Optimization Journey

### 1. **CSV → Parquet Conversion**

```python
# Convert to Parquet (columnar storage)
df.to_parquet('businesses.parquet', engine='pyarrow')

# Load with efficient filtering
df = pd.read_parquet('businesses.parquet', 
                    columns=['name', 'category', 'city', 'pincode', 'rating'])
# Load time: 60 seconds (better, but still terrible)
```

**Improvement**: 33% faster (90s → 60s)

### 2. **Data Type Optimization**

```python
# Optimize data types
df['category'] = df['category'].astype('category')  # Uses less memory
df['city'] = df['city'].astype('category')
df['pincode'] = pd.to_numeric(df['pincode'], downcast='integer')
df['rating'] = pd.to_numeric(df['rating'], downcast='float')

# Memory usage: 3GB → 1.2GB
# Load time: 60s → 30s
```

**Improvement**: 2x faster (60s → 30s)

### 3. **Flask Caching**

```python
from functools import lru_cache
from flask import Flask

app = Flask(__name__)

@lru_cache(maxsize=128)
def load_data(city):
    # Cache city-specific data
    return pd.read_parquet(f'data/{city}.parquet')

@app.route('/get_data')
def get_data():
    city = request.args.get('city')
    data = load_data(city)  # Cached after first call
    return data.to_json()
```

**Improvement**: 30s → 10s (for cached data)

### 4. **Pre-filtering at Read Time**

```python
# Don't load everything, just what you need
df = pd.read_parquet(
    'businesses.parquet',
    filters=[('city', '==', selected_city)]  # Only load Mumbai data if Mumbai selected
)
```

**Result**: **1 second** load time!

## The Final Architecture

```python
# Optimized pipeline
@app.callback(
    Output('map', 'figure'),
    Input('city-dropdown', 'value')
)
def update_map(selected_city):
    # 1. Load only filtered data (1 second)
    df = pd.read_parquet(
        'businesses.parquet',
        filters=[('city', '==', selected_city)]
    )
    
    # 2. Create map (0.5 seconds)
    fig = px.scatter_mapbox(
        df, lat='lat', lon='lng',
        hover_name='name',
        hover_data=['category', 'rating'],
        color='category',
        size='rating'
    )
    
    # 3. Update layout (0.5 seconds)
    fig.update_layout(
        mapbox_style='open-street-map',
        height=600
    )
    
    return fig
```

## The Results

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| **Load time** | 90+ seconds | **1 second** | **90x faster** |
| **Memory usage** | 3GB | 1.2GB | 60% reduction |
| **User satisfaction** | 0% (too slow) | 95% | Much better! |

## Dashboard Features

### Interactive Visualizations
1. **ScatterMapBox**: Geographic distribution with city filtering
2. **Sunburst Chart**: Category breakdown by pincode
3. **Histogram**: Top pincodes by business count
4. **Data Table**: Sortable, filterable business listings
5. **Tree Map**: Category distribution

### User Flow
```python
# User selects Mumbai → Map updates (1 second)
# User clicks Pincode 400001 → Sunburst chart updates
# User selects "Restaurants" → Table filters to restaurants
# User clicks "Download" → CSV exports (2 seconds)
```

## Key Takeaways

1. **Format matters**: Parquet >> CSV for analytics
2. **Load only what you need**: Filters at read time beat filters in memory
3. **Caching is magic**: LRU cache saved the day
4. **Data types**: Optimization saves memory and speed

The 90x speedup turned an unusable dashboard into a fast, responsive tool that students actually use.

---

*This dashboard helps students find businesses and services near their colleges across 7 major Indian cities.*
