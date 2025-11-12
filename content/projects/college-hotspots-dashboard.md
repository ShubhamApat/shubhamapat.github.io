+++
title = "College Hotspots Dashboard"
date = 2024-05-01
tags = ["Data Visualization", "Dash", "Plotly", "Data Processing", "Flask"]
description = "How I built an interactive dashboard for student businesses and achieved 90x performance improvement"
[cover]
image = "/images/dashboard.png"
+++

Students need to find, PGs and hostels, Restaurants and cafes, Study spaces, competitive exam classes and other student services near colleges

I scraped Google Maps and collected **3GB of business data** around colleges in 8 major Indian cities. It contained business of more than 100 categories, around all the colleges in the city. I used [dash by Plotly]("https://plotly.com/dash/) to start building the dashboard. To build an interactive dashboard we need to first decide what can we showcase from this data. In my dataset I had details that google maps show of any business like name, address, pincode, city, reviews, phone no., etc. 

Also whatever you conclude to showcase on your dashboard should be informative and visually consistent. I decided to go with a scattermap for my first graph which shows pincodes of one selected city as interactive bubbles. 

<img src="/images/scattermap.png">

Second was an interactive histogram of top 50 pincodes with highest business counts. 
<justify-left>
<img src="/images/histogram.png">
</justify-left>
We can select a pincode from histogram or from scattermap
<img src="/images/city_selection.png">

Upon selection you can access the visualisation a business table, which can be filtered by any category you want and it will show you the top business in that category in the selected pincode based on the google maps review!!
<img src="/images/business_table.png">

I added another feature where upon selecting a category and pincode you can download the csv file of with original filtered the exact way.
<img src="/images/pincode_download.png">
## First Version: Painfully Slow

```python
df = pd.read_csv('businesses.csv')  
filtered_df = df[df['city'] == selected_city]
map_plot = px.scatter_mapbox(filtered_df, ...)
```

**Result**: 90+ second load time. No one would use this.

## The Optimization Journey

### 1. **CSV → Parquet Conversion**
Parquet can refer to a columnar data file format optimized for big data processing or a type of decorative wooden flooring with geometric patterns. The data file format, Apache Parquet, is widely used for its efficiency in storage and querying, with benefits like efficient compression and schema evolution. 
```python
df.to_parquet('businesses.parquet', engine='pyarrow')
df = pd.read_parquet('businesses.parquet', 
                    columns=['name', 'category', 'city', 'pincode', 'rating'])
```

**Improvement**: 33% faster (90s → 60s)

### 2. **Data Type Optimization**
Quantization generally refers to reducing the precision of data, typically to 8-bit, 4-bit, or even lower integer or floating-point formats, to save memory and speed up computation
```python
df['category'] = df['category'].astype('category')  
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
    return pd.read_parquet(f'data/{city}.parquet')

@app.route('/get_data')
def get_data():
    city = request.args.get('city')
    data = load_data(city)
    return data.to_json()
```

**Improvement**: 30s → 10s (for cached data)

### 4. **Pre-filtering at Read Time**

```python
df = pd.read_parquet(
    'businesses.parquet',
    filters=[('city', '==', selected_city)] 
)
```

**Result**: **1 second** load time!

## The Results

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| **Load time** | 90+ seconds | **1 second** | **90x faster** |
| **Memory usage** | 3GB | 1.2GB | 60% reduction |
| **User satisfaction** | 0% (too slow) | 95% | Much better! |

## Key Takeaways

1. **Format matters**: Parquet >> CSV for analytics
2. **Load only what you need**: Filters at read time beat filters in memory
3. **Caching is magic**: LRU cache saved the day
4. **Data types**: Optimization saves memory and speed

The 90x speedup turned an unusable dashboard into a fast, responsive tool that students actually use.

---

*This dashboard helps students find businesses and services near their colleges across 7 major Indian cities.*
