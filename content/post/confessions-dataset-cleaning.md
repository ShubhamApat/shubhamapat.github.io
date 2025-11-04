---
title: "Cleaning 20,000+ Confessions: Building a Content Moderation Dataset"
date: 2024-03-10
draft: false
description: "My first task at ByteCitadel - cleaning and labeling 20K+ text confessions for dip: life beyond colleges app"
tags: ["data-cleaning", "nlp", "content-moderation", "text-processing"]
cover:
    image: "/images/confessions-cleaning.jpg"
    alt: "Text data cleaning and labeling process"
---

## Introduction

My journey at **ByteCitadel** started with a foundational task: cleaning and labeling 20,000+ text confessions for the dip: life beyond colleges app. This was my first project in the world of content moderation and data processing.

## Project Overview

### The App: dip: life beyond colleges
A social media platform for college students where users can share anonymous confessions, thoughts, and experiences.

### The Problem
- **Raw confessions**: Unstructured, messy text data
- **Volume**: 20,000+ entries
- **Purpose**: Train content moderation models
- **Challenge**: Data was unusable in its original state

## Data Cleaning Process

### 1. **Text Normalization**

#### Issues Found:
- Special characters and emojis scattered throughout
- Inconsistent capitalization
- Extra whitespaces and line breaks
- HTML tags from web scraping
- Emoji representations (like :), :D, etc.)

#### Solution:
```python
# Text cleaning pipeline
def clean_confession(text):
    # Remove HTML tags
    text = re.sub(r'<[^>]+>', '', text)

    # Normalize whitespace
    text = re.sub(r'\s+', ' ', text).strip()

    # Handle common emoji representations
    text = text.replace(' :)', ' 😊').replace(' :D', ' 😃')

    # Normalize capitalization
    text = text.lower()

    return text
```

### 2. **Duplicate Detection**

**Challenge**: Multiple users might post similar confessions
**Solution**:
- Used text similarity (fuzzy matching)
- Identified 3.2% duplicates
- Removed or merged similar entries

```python
from fuzzywuzzy import fuzz

# Check for similar confessions
similar_threshold = 85
for confession in confessions:
    if similarity_with_existing(confession) > similar_threshold:
        # Handle duplicate
        pass
```

### 3. **Quality Filtering**

**Criteria**:
- Minimum length: 20 characters
- Maximum length: 5,000 characters
- Remove spam/repetitive content
- Filter out test data

**Results**:
- 5.8% entries removed (too short)
- 1.2% entries removed (spam/test data)
- 93% retention rate

## Data Labeling Strategy

### Categories
We needed to label each confession for content moderation:

1. **Safe**: General sharing, advice, experiences
2. **Needs Review**: Ambiguous content requiring human review
3. **Inappropriate**: Violates community guidelines
4. **Personal Info**: Contains contact details, addresses
5. **Promotional**: Spam or promotional content

### Labeling Process

#### Approach: Rule-Based + Manual
1. **Automated rules** for clear cases
2. **Manual review** for ambiguous cases
3. **Consistency checks** across annotators

```python
# Rule-based initial labeling
def auto_label(confession):
    if contains_personal_info(confession):
        return "personal_info"
    elif is_promotional(confession):
        return "promotional"
    elif is_spam(confession):
        return "inappropriate"
    else:
        return "needs_review"  # Manual review
```

### Quality Assurance

#### Double Annotation
- 20% of data labeled by two annotators
- Measured inter-annotator agreement
- Target: >90% agreement

#### Error Analysis
- Reviewed mislabeled examples
- Refined rules iteratively
- Improved accuracy over time

## Data Structure

### Final Dataset Format
```json
{
  "confession_id": "conf_001",
  "text": "cleaned confession text",
  "label": "safe/needs_review/inappropriate/...",
  "confidence": 0.95,
  "annotator": "annotator_1",
  "timestamp": "2024-03-10T12:00:00Z"
}
```

### Stats
- **Total Confessions**: 20,000+
- **Safe**: 68%
- **Needs Review**: 18%
- **Inappropriate**: 8%
- **Personal Info**: 4%
- **Promotional**: 2%

## Challenges Faced

### 1. **Ambiguous Content**
- Some confessions weren't clearly safe or unsafe
- **Solution**: Created "Needs Review" category for human evaluation

### 2. **Sarcasm & Context**
- Text sarcasm doesn't translate well to rules
- **Solution**: Flagged for manual review

### 3. **Regional Language**
- Mix of English, Hindi, regional languages
- **Solution**: Identified but left for specialized models

### 4. **Scale**
- Processing 20K+ entries manually is impossible
- **Solution**: Hybrid automated + manual approach

## Impact & Results

### For dip App
- ✅ Clean dataset ready for ML training
- ✅ Reduced moderation workload by 70%
- ✅ Faster content review process
- ✅ Better user experience

### For Model Training
- **Baseline accuracy**: 87% with this dataset
- **Reduced false positives**: Clear labeling
- **Improved generalization**: Diverse content types

## Technical Stack

- **Python**: Core processing
- **Pandas**: Data manipulation
- **NLTK**: Text processing
- **FuzzyWuzzy**: String matching
- **Regex**: Pattern matching
- **Jupyter**: Analysis and exploration

## Lessons Learned

### 1. **Data Cleaning is 80% of the Work**
- Raw data is rarely usable
- Quality input = Quality output
- Don't skip this step!

### 2. **Hybrid Approach Works Best**
- Automation for efficiency
- Human judgment for quality
- Balance is key

### 3. **Document Everything**
- Track preprocessing steps
- Note assumptions
- Make it reproducible

### 4. **Start Simple, Iterate**
- Begin with basic rules
- Add complexity as needed
- Measure improvements

## Code Snippet

```python
# Complete cleaning pipeline
def process_confessions(raw_data):
    cleaned_data = []

    for confession in raw_data:
        # Clean text
        text = clean_confession(confession['text'])

        # Check quality
        if not meets_quality_criteria(text):
            continue

        # Auto-label
        label = auto_label(text)

        # Store
        cleaned_data.append({
            'id': confession['id'],
            'text': text,
            'label': label
        })

    return cleaned_data
```

## Conclusion

This project taught me the importance of **data quality** in machine learning. Clean, labeled data is the foundation of any successful AI system.

### Key Takeaways
- Invest time in data cleaning
- Hybrid automation + manual review
- Document your process
- Measure everything

This 20K confession dataset became the foundation for dip's content moderation system, processing thousands of user posts daily and keeping the community safe.

---

*This was my first project at ByteCitadel, marking the beginning of my journey in content moderation and NLP.*
