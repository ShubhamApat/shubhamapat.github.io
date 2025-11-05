---
title: "I Trained a Model to Identify 160+ Individual Crocodiles with 98.5% Accuracy"
date: 2024-04-20
draft: false
description: "Building a YOLOv8 model for wildlife conservation - identifying individual mugger crocodiles by their back patterns"
tags: ["deep-learning", "computer-vision", "wildlife", "yolov8", "research"]
cover:
    image: "/images/crocodile-research.jpg"
    alt: "Mugger crocodile identification using UAV imagery"
---

## The Question

Can you tell crocodiles apart? Probably not—unless you're a wildlife biologist with decades of experience.

But what if I told you that **every mugger crocodile has a unique back scute pattern** (like human fingerprints)? And what if we could build a computer vision model to automatically identify individual crocodiles from drone footage?

That's exactly what I spent 6 months doing as a research intern at **Ahmedabad University** under Prof. Mehul Raval.

## The Challenge

Wildlife conservation is **data-starved**. Tracking individual animals requires:
- Expensive GPS collars
- Manual photo identification
- Lots of time and expertise
- Frequent human error

> **"160+ individual crocodiles, 1000 images each = 500GB of data"**

We had UAV (drone) imagery from Gujarat's western and southern regions. The footage was stunning—but analyzing it manually would take **years**.

## The Dataset

This wasn't your typical image classification dataset:

- **160 individual crocodiles** (each with unique ID)
- **1,000 images per crocodile**
- **500GB total** of high-resolution drone footage
- **Gujarat, India** - Western and Southern coastal regions

Each crocodile's back scute pattern was annotated by wildlife experts. Think of it like facial recognition, but for crocodile backs.

### The Infrastructure Problem

Training on 500GB of data requires serious compute:
- **Param Shavak Supercomputer** - A Linux-based HPC system
- **No GUI** - Command-line only
- **NVIDIA P5000 GPUs** - Multiple multi-day training runs
- **Custom CLI scripts** - For data processing

Since it was a research project with limited resources, I had to optimize everything. No room for inefficient code.

## The Model Architecture

### Why YOLOv8?

YOLOv8 (You Only Look Once version 8) is great for this use case:
- **Real-time detection** - Process video streams
- **High accuracy** - State-of-the-art performance
- **Multiple sizes** - We used YOLOv8-XL for best results

```python
# Load YOLOv8-XL model
model = YOLO('yolov8x.pt')

# Train on crocodile dataset
model.train(
    data='crocodile.yaml',
    epochs=100,
    imgsz=640,
    batch_size=16,
    device='0,1,2,3'  # 4 GPUs
)
```

### The Training Process

Training took **72 hours** per run on 4 P5000 GPUs:

1. **Data preprocessing** - Custom scripts for massive dataset
2. **Hyperparameter tuning** - Grid search across multiple runs
3. **Threshold optimization** - 0.82 was the sweet spot
4. **Multi-day training** - With checkpointing

The model learned to:
- Detect crocodiles in images
- Classify them into individual IDs (160 classes)
- Handle varying lighting/weather conditions

## The Results

### Performance Metrics

| Metric | Value |
|--------|-------|
| **mAP (Mean Average Precision)** | **98.50%** |
| **Threshold** | 0.82 |
| **Training Time** | 72 hours (4 GPUs) |
| **Inference Speed** | ~50ms per image |

### Visual Examples

The model correctly identified:
- Crocodiles in various poses
- Different lighting conditions
- Partial occlusions
- Water reflections

> **"98.5% accuracy on 160 individual crocodiles = better than human experts"**

## Technical Challenges

### 1. **Massive Dataset Management**

500GB of images doesn't fit in RAM. I built a streaming data loader:

```python
class CrocodileDataset(Dataset):
    def __init__(self, image_paths, labels):
        self.image_paths = image_paths
        self.labels = labels

    def __getitem__(self, idx):
        # Stream from disk (no loading all into memory)
        image = self.load_image(idx)
        label = self.labels[idx]
        return image, label
```

### 2. **Linux-Only Environment**

The Param Shavak supercomputer had **no GUI**. Everything had to be command-line:

```bash
# Submit training job
qsub -l gpu=4 -N crocodile_training train_model.sh

# Monitor progress
tail -f training.log
```

I couldn't use Jupyter notebooks or PyCharm. Just Vim and command-line tools.

### 3. **Hyperparameter Tuning**

With 160 classes and limited compute time, I had to be smart:

```python
# Parameter grid (tested iteratively)
param_grid = {
    'learning_rate': [0.001, 0.01, 0.1],
    'batch_size': [8, 16, 32],
    'optimizer': ['SGD', 'Adam'],
    'weight_decay': [0.0001, 0.001, 0.01]
}
```

### 4. **Threshold Optimization**

Finding the right confidence threshold is crucial:

```python
# True Positive/Negative rates at different thresholds
thresholds = np.arange(0.5, 0.95, 0.05)
for t in thresholds:
    tp, tn, fp, fn = calculate_confusion_matrix(predictions, t)
    tpr = tp / (tp + fn)  # True Positive Rate
    tnr = tn / (tn + fp)  # True Negative Rate
```

The **0.82 threshold** maximized TPR/TNR balance for wildlife monitoring.

## Real-World Applications

This model isn't just academic—it has concrete uses:

### 1. **Behavioral Studies**
- Track individual crocodile movements
- Understand migration patterns
- Study social behaviors

### 2. **Population Monitoring**
- Count populations without capture
- Estimate survival rates
- Track breeding success

### 3. **Conservation Impact**
- Monitor endangered species
- Assess habitat health
- Guide protection efforts

### 4. **Research Applications**
- Non-invasive monitoring
- Automated tracking systems
- Long-term studies

## Lessons Learned

### 1. **Domain Expertise Matters**

The wildlife biologists' annotation was invaluable. They knew:
- Which back scutes are unique identifiers
- How to handle ambiguous cases
- The behavioral context behind patterns

> **"Computer vision is only as good as your training data"**

### 2. **Infrastructure Limitations**

Linux-only environments force you to:
- Master command-line tools
- Write robust, reproducible code
- Think carefully about resources

### 3. **Multi-Day Training Requires Patience**

72-hour training runs mean:
- Bugs are expensive
- Checkpointing is critical
- You can't iterate quickly

### 4. **Conservation Needs Practical Solutions**

Academic models need to be:
- Deployable in field conditions
- Robust to real-world variations
- Useful for actual conservation

## The Bigger Picture

This project showed me that **computer vision can make a real difference** in conservation. We're not just building models for fun—we're building tools that help protect endangered species.

The 98.5% accuracy means wildlife researchers can:
- Replace manual identification (which takes hours)
- Process massive datasets automatically
- Track populations at scale

## Key Takeaways

1. **Deep learning + conservation = powerful combination**
2. **Infrastructure constraints push you to be creative**
3. **Domain expertise is irreplaceable**
4. **Real-world deployment needs practical engineering**
5. **Conservation impact makes technical challenges worthwhile**

This project taught me that **computer vision isn't just for tech companies**. It's a tool that can help solve some of the world's most pressing environmental challenges.

The 98.5% accuracy on 160 individual crocodiles proves that with enough data and the right approach, we can build models that rival expert human performance—while being faster, more consistent, and more scalable.

---

*This research was conducted at Ahmedabad University under Prof. Mehul Raval. If you're interested in conservation technology or need help with computer vision projects, feel free to reach out.*
