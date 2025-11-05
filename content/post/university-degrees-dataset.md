---
title: "I Collected 1,400 University Degrees to Build a Verification Dataset"
date: 2024-11-01
draft: false
description: "How I scraped 1,400 unique university degrees from 8 countries to build a comprehensive academic verification dataset"
tags: ["data-collection", "web-scraping", "academic-data", "dataset", "university"]
cover:
    image: "/images/university-degrees.jpg"
    alt: "Collection of university degree certificates from around the world"
---

## The Goal

Building systems for **degree verification** and **academic fraud detection** requires authentic examples. I collected degrees from LinkedIn graduation posts across 8 countries.

## Coverage

**1,400 unique degrees** from:
- **USA**: Ivy League, top universities, business schools
- **India**: IITs, IIMs, AIIMS
- **UK**: Oxford, Cambridge, Russell Group
- **Australia**: Group of Eight (Go8)
- **Europe**: Top institutions (ETH Zurich, TUM, etc.)
- **Asia**: NUS, NTU, Seoul National, University of Tokyo

## Collection Method

```python
def collect_graduation_posts(keywords, universities):
    posts = []
    for university in universities:
        search_queries = [
            f"{university} graduation 2024",
            f"{university} degree ceremony",
            f"graduated from {university}"
        ]
        
        for query in search_queries:
            results = linkedin_scraper.search_posts(
                query=query,
                limit=50,
                date_range="2023-2024"
            )
            posts.extend(results)
    return posts
```

## Quality Control

### Deduplication
Used perceptual hashing to remove duplicates:

```python
from imagehash import average_hash

hash_value = average_hash(image)
if hash_value not in seen_hashes:
    seen_hashes.add(hash_value)
    unique_degrees.append(degree)
```

**Removed**: ~15% duplicates

### Validation
- **Resolution check**: Minimum 800x600
- **Format check**: JPEG/PNG only
- **Content verification**: OCR-based degree detection

## The Dataset

### Structure
```json
{
  "degree_id": "deg_0001",
  "university": "Stanford University",
  "country": "USA",
  "degree_type": "Bachelor of Science",
  "graduation_year": 2024,
  "image_path": "degrees/usa/stanford_001.jpg",
  "quality_score": 0.95,
  "verified": true
}
```

### Statistics
- **USA**: 450 degrees
- **India**: 300 degrees
- **UK**: 180 degrees
- **Australia**: 150 degrees
- **Others**: 320 degrees

## Applications

### 1. **Document Verification**
Train ML models to verify degree authenticity

### 2. **Fraud Detection**
Identify fake or tampered certificates

### 3. **Academic Research**
Study degree formats and university branding

### 4. **Market Analysis**
Track educational trends and university popularity

## Lessons Learned

### 1. **Source Quality Matters**
LinkedIn posts provided authentic, high-quality degree images

### 2. **Global Coverage is Hard**
Some countries had fewer graduation posts

### 3. **Privacy Considerations**
Anonymized student information, used for research only

### 4. **Documentation is Critical**
Tracked sources, preprocessing, and validation steps

## Impact

This dataset enables:
- **Automated degree verification** systems
- **Fraud detection** for educational credentials
- **Academic research** on document security
- **Market analysis** of educational trends

## Key Takeaway

Large-scale data collection from social media requires careful curation, quality control, and ethical considerations.

---

*This comprehensive degree dataset supports academic integrity and fraud detection research worldwide.*
