---
title: "Collecting 1,400 University Degrees: A Global Dataset"
date: 2024-11-01
draft: false
description: "How I scraped 1,400 unique university degrees from 8 countries to build a comprehensive academic verification dataset"
tags: ["data-collection", "web-scraping", "academic-data", "dataset", "university"]
cover:
    image: "/images/university-degrees.jpg"
    alt: "Collection of university degree certificates from around the world"
---

## Introduction

Building systems for **degree verification** and **academic fraud detection** requires diverse, authentic examples. I collected a comprehensive dataset of **1,400 unique university degrees** from LinkedIn graduation posts across **8 countries**, creating a valuable resource for academic institutions and verification services.

## Project Vision

### Use Cases
1. **Academic Verification**: Automated degree authenticity checks
2. **Fraud Detection**: Identify fake or tampered certificates
3. **University Research**: Analyze degree formats and styles
4. **Machine Learning**: Train models for document verification
5. **Market Analysis**: Track educational trends globally

### Scope
- **1,400 unique degrees**
- **8 countries/regions**
- **Top universities worldwide**
- **Real, verified degree images**

## Geographic Coverage Strategy

### 1. **United States**
**Target Institutions**:
- All **Ivy League** schools (Harvard, Yale, Princeton, Columbia, etc.)
- **Top public universities** (UC Berkeley, UCLA, etc.)
- **Elite private institutions** (Stanford, MIT, etc.)
- **Business schools** (Wharton, HBS, etc.)

### 2. **United Kingdom**
**Focus Universities**:
- **Oxford** and **Cambridge**
- **Russell Group** universities
- **London institutions** (LSE, Imperial, King's College)
- **Top business schools** (London Business School)

### 3. **Europe**
**Coverage**:
- **Germany**: TUM, LMU, Heidelberg
- **France**: Sorbonne, Polytechnique
- **Switzerland**: ETH Zurich, EPFL
- **Netherlands**: TU Delft, University of Amsterdam
- **Italy**: Politecnico di Milano, University of Bologna

### 4. **India**
**Elite Institutions**:
- **IITs**: All 23 IITs
- **IIMs**: Top business schools
- **AIIMS**: Medical excellence
- **NITs** and other premier institutes

### 5. **South Korea**
**SKY Universities**:
- **Seoul National University (SNU)**
- **Korea University (KU)**
- **Yonsei University (YU)**
- Plus other top-tier institutions

### 6. **Japan**
**Leading Universities**:
- **University of Tokyo**
- **Kyoto University**
- **Tokyo Institute of Technology**
- **Osaka University**

### 7. **Australia**
**Group of Eight (Go8)**:
- **Australian National University (ANU)**
- **University of Melbourne**
- **University of Sydney**
- **UNSW Sydney**
- **Monash University**
- Plus others

### 8. **Singapore**
**Top Institutions**:
- **National University of Singapore (NUS)**
- **Nanyang Technological University (NTU)**
- **Singapore Management University (SMU)**

## Data Collection Methodology

### Source: LinkedIn Graduation Posts

#### Why LinkedIn?
- **Real content**: Students share genuine degree images
- **Global reach**: Access to worldwide universities
- **High quality**: Professionally taken photos
- **Diverse angles**: Various lighting and positions

#### Collection Process
```python
def collect_graduation_posts(keywords, universities):
    posts = []

    for university in universities:
        # Search for graduation posts
        search_queries = [
            f"{university} graduation 2024",
            f"{university} degree ceremony",
            f"{university} convocation",
            f"graduated from {university}"
        ]

        for query in search_queries:
            # Use LinkedIn scraper
            results = linkedin_scraper.search_posts(
                query=query,
                limit=50,
                date_range="2023-2024"
            )

            posts.extend(results)

    return posts
```

### Data Extraction

#### Image Download
```python
def download_degree_images(posts):
    degrees = []

    for post in posts:
        # Check if post contains degree image
        if has_degree_image(post):
            image_url = extract_image_url(post)

            # Download and store
            image_path = download_image(image_url)

            # Extract metadata
            degree_data = {
                'university': extract_university(post),
                'degree_type': extract_degree_type(post),
                'graduation_year': extract_year(post),
                'image_path': image_path,
                'source_post': post['url'],
                'country': determine_country(post)
            }

            degrees.append(degree_data)

    return degrees
```

#### Metadata Extraction
```python
def extract_degree_metadata(image_path, post_text):
    # Use OCR to extract text
    text = ocr.extract_text(image_path)

    # Parse key information
    metadata = {
        'university_name': parse_university(text),
        'degree_name': parse_degree(text),
        'graduation_date': parse_date(text),
        'student_name': parse_name(text),
        'program': parse_program(text),
        'honors': detect_honors(text)
    }

    return metadata
```

## Data Quality Control

### 1. **Duplicate Detection**
```python
from imagehash import average_hash
from PIL import Image

def detect_duplicates(degrees):
    # Use perceptual hashing
    hashes = {}
    unique_degrees = []

    for degree in degrees:
        image = Image.open(degree['image_path'])
        hash_value = average_hash(image)

        if hash_value not in hashes:
            hashes[hash_value] = degree
            unique_degrees.append(degree)
        else:
            # Similar degree found
            print(f"Duplicate detected: {degree['university']}")

    return unique_degrees
```

### 2. **Quality Validation**
```python
def validate_degree_quality(image_path):
    image = Image.open(image_path)

    # Check resolution
    if image.width < 800 or image.height < 600:
        return False, "Low resolution"

    # Check if it's a degree certificate
    confidence = detect_degree_confidence(image)
    if confidence < 0.8:
        return False, "Not a degree certificate"

    # Check for clarity
    sharpness = calculate_sharpness(image)
    if sharpness < 0.7:
        return False, "Blurry image"

    return True, "Valid"
```

### 3. **University Verification**
```python
def verify_university(degree):
    # Cross-reference with known universities
    university_list = load_university_database()

    if degree['university'] in university_list:
        return True, "Verified university"
    else:
        return False, "Unknown university"
```

## Dataset Statistics

### Country Distribution
| Country | Degrees | Top Universities |
|---------|---------|------------------|
| **USA** | 450 | Harvard, MIT, Stanford, Yale |
| **India** | 300 | IIT Delhi, IIT Bombay, IIM Ahmedabad |
| **UK** | 180 | Oxford, Cambridge, LSE |
| **Australia** | 150 | ANU, Melbourne, Sydney |
| **Canada** | 120 | Toronto, UBC, McGill |
| **Germany** | 80 | TUM, LMU, ETH Zurich |
| **Singapore** | 70 | NUS, NTU |
| **Others** | 50 | Various |

### Degree Types
- **Bachelor's**: 45%
- **Master's**: 35%
- **PhD**: 12%
- **MBA**: 5%
- **Other**: 3%

### Graduation Years
- **2024**: 40%
- **2023**: 35%
- **2022**: 15%
- **2021 and earlier**: 10%

## Sample Dataset

### Example Entry
```json
{
  "degree_id": "deg_0001",
  "university": "Stanford University",
  "country": "USA",
  "degree_type": "Bachelor of Science",
  "major": "Computer Science",
  "graduation_year": 2024,
  "honors": "Summa Cum Laude",
  "image_path": "degrees/usa/stanford_001.jpg",
  "image_size": "1200x1600",
  "format": "JPEG",
  "quality_score": 0.95,
  "verified": true,
  "date_collected": "2024-11-01"
}
```

## Applications

### 1. **Document Verification Systems**
```python
# Train ML model to verify degrees
def verify_degree(document_image):
    features = extract_features(document_image)
    prediction = degree_model.predict(features)

    return {
        'is_authentic': prediction[0] > 0.8,
        'confidence': prediction[0],
        'university': predict_university(features),
        'degree_type': predict_degree_type(features)
    }
```

### 2. **Fraud Detection**
- Identify altered or fake degrees
- Detect copied signatures
- Spot template inconsistencies
- Flag suspicious patterns

### 3. **Academic Research**
- Study degree formats evolution
- Compare international standards
- Analyze university branding
- Track educational trends

### 4. **Market Analysis**
- University popularity metrics
- Graduate placement rates
- Program demand analysis
- Geographic distribution

## Technical Implementation

### Image Processing Pipeline
```python
class DegreeDatasetProcessor:
    def __init__(self):
        self.ocr_engine = EasyOCR()
        self.hash_alg = average_hash

    def process_degree_image(self, image_path):
        # 1. Load and preprocess
        image = self.preprocess_image(image_path)

        # 2. Extract text
        text = self.ocr_engine.readtext(image)

        # 3. Parse metadata
        metadata = self.parse_metadata(text)

        # 4. Generate hash
        image_hash = self.hash_alg(image)

        # 5. Quality check
        quality = self.check_quality(image)

        return {
            'metadata': metadata,
            'hash': image_hash,
            'quality': quality,
            'text': text
        }
```

### Storage Structure
```
dataset/
├── degrees/
│   ├── usa/
│   │   ├── harvard/
│   │   ├── mit/
│   │   └── stanford/
│   ├── uk/
│   │   ├── oxford/
│   │   └── cambridge/
│   └── ...
├── metadata/
│   └── degrees_metadata.json
└── hashes/
    └── degree_hashes.pkl
```

## Challenges & Solutions

### 1. **Image Quality Variation**
**Challenge**: Graduation photos have varying quality
**Solution**: Implemented quality scoring and filtering

### 2. **Privacy Concerns**
**Challenge**: Student names visible in images
**Solution**: Anonymized data, removed PII in metadata

### 3. **Copyright Issues**
**Challenge**: Using degree images
**Solution**: Used for research/educational purposes only, not for redistribution

### 4. **Data Imbalance**
**Challenge**: Some countries/universities over-represented
**Solution**: Targeted collection for underrepresented institutions

## Legal & Ethical Considerations

### Data Usage Policy
- **Research/educational use only**
- **No commercial distribution**
- **Respect privacy**: Anonymized data
- **Academic integrity**: Promote verification, not fraud

### Compliance
- **GDPR**: European data protection
- **Copyright**: Educational fair use
- **University policies**: Respected each institution's IP

## Future Enhancements

### 1. **Expand Coverage**
- Add more countries (France, Italy, etc.)
- Include more universities per country
- Cover more degree types

### 2. **Enhanced Metadata**
- Add university logos for easier verification
- Include degree program details
- Track degree format changes over time

### 3. **Machine Learning Features**
- Train CNN for degree classification
- Build similarity detection models
- Create authenticity scoring

### 4. **API Access**
- Build REST API for researchers
- Implement batch verification
- Add real-time degree checking

## Impact

### Academic Institutions
- Better fraud detection
- Streamlined verification processes
- Research data for document security

### Students
- Credential verification services
- Fraud protection
- Academic integrity

### Research Community
- Public dataset for document analysis
- Benchmark for ML models
- Academic fraud detection research

## Technologies Used

- **Python**: Core language
- **Selenium**: Web scraping
- **EasyOCR**: Text extraction
- **PIL (Pillow)**: Image processing
- **imagehash**: Perceptual hashing
- **Pandas**: Data management
- **SQLite**: Metadata storage
- **NumPy**: Array operations

## Conclusion

Collecting 1,400 university degrees from 8 countries created a valuable dataset for academic verification and research. The comprehensive, high-quality dataset enables:

- **Automated degree verification**
- **Fraud detection systems**
- **Academic research**
- **Machine learning applications**

### Key Takeaways
- **Systematic collection** from reliable sources
- **Quality control** at every step
- **Ethical data handling** is crucial
- **Diverse representation** improves utility

The dataset contributes to academic integrity and helps build systems that protect students, universities, and employers from degree fraud.

**Future**: Expand to more countries, add video degrees, and build real-time verification APIs.

---

*This comprehensive degree dataset supports academic integrity and fraud detection research worldwide.*
