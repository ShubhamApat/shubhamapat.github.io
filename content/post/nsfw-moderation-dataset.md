---
title: "I Collected 150,000+ Images to Train a Content Moderation Model"
date: 2024-09-15
draft: false
description: "How I built a 150K image dataset across 6 content categories for Dip's content moderation system"
tags: ["dataset", "data-collection", "content-moderation", "machine-learning"]
cover:
    image: "/images/image-dataset.jpg"
    alt: "Large-scale image dataset collection for content moderation"
---

## The Challenge

Building a content moderation system for a social media app requires **training data**. Lots of it.

Dip needed to automatically detect:
- NSFW content
- Violence
- Drugs
- Weapons
- Hateful content
- Neutral content

The question: **where do you get 150,000+ labeled images?**

This was my task at ByteCitadel: collect, curate, and label a massive dataset for Dip's image moderation pipeline.

## The Plan

Six categories, ~30,000 images each:
- **Violence** (30K)
- **Drugs** (30K)
- **NSFW** (30K)
- **Neutral** (30K)
- **Hateful Memes** (30K)
- **Weapons** (30K)

Total: **180,000 images** (we collected extra for quality control)

## Data Sources

### 1. Kaggle Datasets

Kaggle has tons of public datasets. I found:
- **Violence datasets** - From academic papers on conflict detection
- **Drug-related imagery** - For medical/research purposes
- **Weapon detection datasets** - Security/computer vision research

```python
# Download Kaggle datasets
import kaggle

kaggle.api.dataset_download_files(
    'tapakah68/violence-detection-dataset',
    path='./datasets/violence/',
    unzip=True
)
```

### 2. Research Papers

Academic papers often release datasets alongside publications:

```python
# Search for datasets mentioned in papers
paper_datasets = [
    "Flames-Dataset",  # Violence detection
    "Drug-Image-Dataset",  # Drug classification
    "Meme-Classification-Dataset"  # Hateful memes
]

for dataset in paper_datasets:
    download_dataset(dataset)
```

### 3. Roboflow

Roboflow has high-quality, pre-labeled computer vision datasets:

```python
# Export from Roboflow
import roboflow

rf = roboflow.Roboflow(api_key="YOUR_API_KEY")
project = rf.workspace("workspace").project("weapon-detection")
dataset = project.version(1).download("coco")
```

### 4. Custom Collection

For edge cases, I scraped and labeled manually:

```python
# Custom scraping for underrepresented categories
def collect_images(query, num_images=1000):
    results = []
    for image_url in image_search_api.search(query, num_images):
        download_image(image_url, f"./datasets/{query}/")
        results.append(image_url)
    return results
```

## The Curation Process

### Quality Filtering

Not all downloaded images were usable:

```python
def filter_image_quality(image_path):
    # Check file size
    if os.path.getsize(image_path) < 10 * 1024:  # Less than 10KB
        return False, "Too small"

    # Verify it's actually an image
    try:
        img = Image.open(image_path)
        img.verify()
    except:
        return False, "Corrupted"

    # Check resolution
    img = Image.open(image_path)
    if img.width < 224 or img.height < 224:
        return False, "Low resolution"

    # Check if it's actually an image (not blank/black)
    if is_blank_image(img):
        return False, "Blank image"

    return True, "Valid"
```

### Deduplication

Images from different sources often overlapped:

```python
from imagehash import average_hash

def remove_duplicates(image_paths):
    hashes = {}
    unique_images = []

    for img_path in image_paths:
        # Calculate perceptual hash
        img = Image.open(img_path)
        hash_value = average_hash(img)

        if hash_value not in hashes:
            hashes[hash_value] = img_path
            unique_images.append(img_path)
        else:
            print(f"Duplicate found: {img_path}")

    return unique_images

# Found and removed ~15,000 duplicates across the dataset
```

### Class Balancing

Ensuring equal representation across categories:

```python
# Check class distribution
class_counts = {}
for category in categories:
    count = len(os.listdir(f'./datasets/{category}/'))
    class_counts[category] = count

print(class_counts)
# Expected: ~30K per class

# If imbalance > 10%, collect more data for minority classes
```

## Labeling Strategy

### Automated Labeling

For well-known datasets, labels were already provided:

```python
# Process COCO-formatted labels
import json

with open('annotations.json', 'r') as f:
    coco_data = json.load(f)

for annotation in coco_data['annotations']:
    image_id = annotation['image_id']
    category_id = annotation['category_id']
    image_path = f'images/{image_id}.jpg'
    category = id_to_category[category_id]
    move_to_category(image_path, category)
```

### Manual Labeling

For custom-collected images, I had to label manually:

```python
# Labeling tool for edge cases
def label_image(image_path):
    img = Image.open(image_path)
    img.show()

    label = input(f"What category is this image? ")
    # Categories: violence, drugs, nsfw, neutral, weapons, hateful_memes

    # Save label
    save_label(image_path, label)
```

> **"Labeling 10,000 images by hand teaches you to appreciate automated annotation tools."**

### Quality Validation

Double-checking labels for accuracy:

```python
def validate_labels(image_path, expected_label):
    img = Image.open(image_path)
    actual_label = manual_label(image_path)

    if expected_label != actual_label:
        print(f"MISLABELED: {image_path}")
        print(f"Expected: {expected_label}, Got: {actual_label}")
        correct_label(expected_label)
```

## Challenges

### 1. **Copyright & Legal Issues**

Using internet images requires care:
- **Academic use only** - Educational/research purposes
- **No redistribution** - Keep dataset internal
- **Fair use** - Transformative use for ML training

### 2. **Cultural Sensitivity**

What counts as "inappropriate" varies by culture:
- Western vs. Indian context
- Religious imagery
- Historical vs. contemporary content

### 3. **Data Imbalance**

Some categories were harder to find:
- **Weapons**: Plenty of gun images, fewer knives
- **Drugs**: Need diverse substances, not just weed
- **Violence**: Real violence vs. fictional content

### 4. **False Positives**

Training on misleading data:
- Photoshoot violence vs. real violence
- Medical imagery (drugs for legitimate use)
- Art depicting weapons (not real weapons)

## The Final Dataset

### Statistics

| Category | Images | Duplicates Removed | Final Count |
|----------|--------|-------------------|-------------|
| **Violence** | 32,000 | 2,000 | 30,000 |
| **Drugs** | 35,000 | 5,000 | 30,000 |
| **NSFW** | 38,000 | 8,000 | 30,000 |
| **Neutral** | 31,000 | 1,000 | 30,000 |
| **Hateful Memes** | 29,000 | 0 | 29,000 |
| **Weapons** | 33,000 | 3,000 | 30,000 |

**Total**: 198,000 → **179,000** (after quality filtering)

### Data Structure

```
dataset/
├── violence/
│   ├── img_00001.jpg
│   ├── img_00002.jpg
│   └── ...
├── drugs/
├── nsfw/
├── neutral/
├── hateful_memes/
└── weapons/
```

Each image has a corresponding metadata file:

```json
{
  "image_id": "img_00001",
  "category": "violence",
  "source": "kaggle_violence_dataset",
  "resolution": "1920x1080",
  "file_size": "245KB",
  "date_collected": "2024-09-15",
  "verified": true,
  "quality_score": 0.95
}
```

## Model Training Results

With this dataset, Dip's content moderation model achieved:

- **92% accuracy** across all categories
- **94% precision** for violence detection
- **89% precision** for NSFW detection
- **<3% false positive rate** on neutral images

```python
# Train multi-class classifier
from torchvision import models
import torch

model = models.resnet50(pretrained=True)
model.fc = torch.nn.Linear(2048, 6)  # 6 categories

# Train with data augmentation
train_loader = DataLoader(dataset, batch_size=32, shuffle=True)

for epoch in range(10):
    for batch in train_loader:
        images, labels = batch
        outputs = model(images)
        loss = criterion(outputs, labels)
        loss.backward()
        optimizer.step()
```

## Lessons Learned

### 1. **Quality Over Quantity**

30K high-quality images > 100K mediocre ones

### 2. **Source Diversity Matters**

Using multiple sources prevents model bias

### 3. **Documentation is Critical**

Track:
- Data sources
- Preprocessing steps
- Quality checks
- Labeling methodology

### 4. **Expect Legal Complexity**

Using internet images requires legal review

### 5. **Human Validation is Essential**

Automated filters catch obvious cases, but humans catch edge cases

## Key Takeaways

1. **Data collection is 80% of ML** - Garbage in, garbage out
2. **Multiple sources** reduce bias and improve generalization
3. **Quality filtering** is as important as data collection
4. **Legal considerations** matter for production systems
5. **Document everything** for reproducibility

This dataset became the foundation for Dip's image moderation system, processing millions of images daily and keeping the platform safe for users.

The 179,000 images across 6 categories now power real-time content moderation for thousands of users. It's a reminder that **machine learning is only as good as your training data**.

---

*This dataset collection was part of building Dip's content moderation pipeline. If you need help with dataset creation or content moderation, feel free to reach out.*
