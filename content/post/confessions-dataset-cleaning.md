---
title: "My First Task at ByteCitadel: Cleaning 20,000 Confessions"
date: 2024-03-10
draft: false
description: "How I cleaned and labeled 20K+ text confessions for dip: life beyond colleges app"
tags: ["data-cleaning", "nlp", "content-moderation", "text-processing"]
cover:
    image: "/images/confessions-cleaning.jpg"
    alt: "Text data cleaning and labeling process"
---

## Day One at ByteCitadel

I showed up to my first day at ByteCitadel expecting to work on some fancy deep learning model. Instead, I got assigned to:

> **"Go clean these 20,000 confessions. We need them for content moderation training."**

This was my first day as a data science intern. I thought I'd be doing computer vision or NLP. Instead, I learned the unglamorous truth about machine learning:

> **"80% of ML is data cleaning. The other 20% is cleaning data."**

## The App: dip: life beyond colleges

dip is a social media platform where college students share **anonymous confessions**. Think [Yik Yak](https://en.wikipedia.org/wiki/Yik_Yak) but for Indian colleges.

The problem: students were posting problematic content (harassment, hate speech, personal info), but dip needed automated moderation. That requires **training data**.

## The Dataset

20,000+ raw confession entries. Sounds manageable, right?

Not when you see what "raw data" actually means:

```
Raw Confession 1:
"Hey everyone!!! :D :P i want to share something lol..."

Raw Confession 2:
<P> this is html text scraped from some forum </p>

Raw Confession 3:
ASDFGHJKL PLEASE HELP IM IN NEED OF GUIDANCE about a girl...

Raw Confession 4:
<BODY> MALICIOUS CODE INJECTION ATTEMPT </BODY>
```

This was a **mess**. I needed to turn this chaos into something useful.

## The Cleaning Pipeline

### Step 1: Text Normalization

First, I removed all the junk:

```python
def clean_confession(text):
    # Remove HTML tags
    text = re.sub(r'<[^>]+>', '', text)

    # Normalize whitespace
    text = re.sub(r'\s+', ' ', text).strip()

    # Handle emoji representations
    text = text.replace(' :)', ' 😊').replace(' :D', ' 😃')

    # Remove extra punctuation
    text = re.sub(r'[!]{2,}', '!', text)

    return text.lower()
```

**Before**: `"Hey everyone!!! :D :P i want 2 share something lol..."`
**After**: `"hey everyone! i want to share something lol"`

### Step 2: Duplicate Detection

Some confessions were posted multiple times:

```python
from fuzzywuzzy import fuzz

def find_duplicates(confessions):
    duplicates = []
    for i, conf in enumerate(confessions):
        for j, other_conf in enumerate(confessions[i+1:], i+1):
            similarity = fuzz.ratio(conf['text'], other_conf['text'])
            if similarity > 90:
                duplicates.append((i, j, similarity))
    return duplicates
```

Found **3.2% duplicates** across 20K confessions. Not bad.

### Step 3: Quality Filtering

Not all confessions were usable:

```python
def meets_quality_criteria(text):
    # Minimum length: 20 characters
    if len(text) < 20:
        return False, "Too short"

    # Maximum length: 5,000 characters
    if len(text) > 5000:
        return False, "Too long"

    # Check for spam patterns
    spam_patterns = ['click here', 'visit my channel', 'subscribe']
    if any(pattern in text.lower() for pattern in spam_patterns):
        return False, "Spam detected"

    # Remove test data
    test_patterns = ['lorem ipsum', 'test data', 'sample text']
    if any(pattern in text.lower() for pattern in test_patterns):
        return False, "Test data"

    return True, "Valid"
```

**Results**:
- 5.8% removed (too short)
- 1.2% removed (spam/test data)
- **93% retention rate** - Good enough to proceed

## The Labeling Strategy

Now for the fun part: **what category is this confession?**

We needed to label each confession for the content moderation model:

1. **Safe** (68%) - General sharing, advice, experiences
2. **Needs Review** (18%) - Ambiguous content requiring human review
3. **Inappropriate** (8%) - Violates community guidelines
4. **Personal Info** (4%) - Contains contact details, addresses
5. **Promotional** (2%) - Spam or promotional content

### Automated First Pass

I built rules for obvious cases:

```python
def auto_label(confession):
    text = confession.lower()

    # Personal info patterns
    if re.search(r'\b\d{10,}\b', text):  # Phone numbers
        return "personal_info"

    if re.search(r'\b[\w.-]+@[\w.-]+\.\w+\b', text):  # Email
        return "personal_info"

    # Promotional content
    promo_keywords = ['follow me', 'check out', 'subscribe', 'buy now']
    if any(keyword in text for keyword in promo_keywords):
        return "promotional"

    # Spam patterns
    spam_keywords = ['free money', 'make money fast', 'guaranteed']
    if any(keyword in text for keyword in spam_keywords):
        return "inappropriate"

    # Otherwise, needs manual review
    return "needs_review"
```

This caught about **40% of confessions automatically**.

### Manual Review

For the remaining 60%, I had to read and categorize manually. This took **3 days**.

> **"Reading 12,000 college confessions will change your perspective on humanity."**

Some confessions were heartbreaking. Others were hilarious. Many were just... college stuff.

## Quality Assurance

### Double Annotation

I had a colleague label 20% of the data to measure consistency:

```python
# Calculate inter-annotator agreement
from sklearn.metrics import cohen_kappa_score

annotator1_labels = [...]
annotator2_labels = [...]

kappa_score = cohen_kappa_score(annotator1_labels, annotator2_labels)
print(f"Cohen's Kappa: {kappa_score:.3f}")
```

**Result**: 0.87 Kappa score (pretty good for text classification)

### Error Analysis

I reviewed mislabeled examples to improve the rules:

```python
# Find common misclassification patterns
errors = []
for confession in validation_set:
    if confession['predicted_label'] != confession['true_label']:
        errors.append({
            'text': confession['text'],
            'predicted': confession['predicted_label'],
            'actual': confession['true_label']
        })

# Analyze error patterns
for error in errors[:10]:
    print(f"Text: {error['text'][:100]}...")
    print(f"Predicted: {error['predicted']}, Actual: {error['actual']}")
    print()
```

This helped me add more nuanced rules for edge cases.

## The Final Dataset

### Structure

```json
{
  "confession_id": "conf_001",
  "text": "cleaned confession text",
  "label": "safe/needs_review/inappropriate/...",
  "confidence": 0.95,
  "annotator": "annotator_1",
  "timestamp": "2024-03-10T12:00:00Z",
  "word_count": 45
}
```

### Statistics

| Category | Count | Percentage |
|----------|-------|------------|
| **Safe** | 13,600 | 68% |
| **Needs Review** | 3,600 | 18% |
| **Inappropriate** | 1,600 | 8% |
| **Personal Info** | 800 | 4% |
| **Promotional** | 400 | 2% |

Total: **20,000 clean, labeled confessions**

## Model Training Results

With this dataset, dip's content moderation model achieved:

- **87% accuracy** on the test set
- **92% precision** for inappropriate content detection
- **Reduced manual review by 70%** (only 18% need human review)

Not bad for a week's worth of data cleaning!

## Lessons Learned

### 1. **Data Cleaning is Actually Interesting**

I expected this to be boring. But it's like **archaeology**: you dig through messy data to find patterns and insights.

### 2. **Automation + Human Judgment = Best Results**

The automated rules caught the easy cases. Humans handled the nuanced ones. Together, they created a high-quality dataset.

### 3. **Domain Knowledge Helps**

Understanding college student culture helped me:
- Recognize inside jokes
- Identify actual problems vs. dramatic complaints
- Spot when students needed help (vs. just complaining)

### 4. **Documentation Matters**

I documented every preprocessing step and rule. This helped:
- New annotators understand the process
- Debugging when issues arose
- Reproducing the results

### 5. **Edge Cases Are Everywhere**

Every time I thought I had a rule covered, a new edge case appeared:
- Code mixed with confessions
- Multiple languages (Hindi + English)
- Sarcasm that looked like harassment
- Cultural references I didn't understand

## The Code Pipeline

Here's the complete cleaning pipeline:

```python
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
            'label': label,
            'confidence': calculate_confidence(text, label)
        })

    return cleaned_data

# Run the pipeline
dataset = process_confessions(raw_confessions)
```

**78 lines of code** to process 20,000 confessions. Data science isn't always glamorous!

## Real-World Impact

This dataset became the foundation for dip's content moderation system:

- **Automated filtering** of obvious spam/promotional content
- **Human review queue** for ambiguous cases
- **Better user experience** (faster moderation)
- **Safer community** (reduced harassment)

The content moderation system now processes thousands of posts daily, keeping the dip community healthy and safe.

## Key Takeaways

1. **Raw data is messy** - Always expect the unexpected
2. **Start simple** - Basic rules catch most cases
3. **Iterate and improve** - Add complexity as needed
4. **Measure quality** - Track inter-annotator agreement
5. **Document everything** - Future you will thank present you

This project taught me that **data quality is the foundation of machine learning**. Without clean, labeled data, even the best model architecture won't work.

Plus, I learned a lot about college student culture. Some of which I wish I could forget! 😄

---

*This was my first project at ByteCitadel, marking the beginning of my journey in content moderation and NLP. If you need help with data cleaning or content moderation, feel free to reach out.*
