+++
title = "Student ID Verification Pipeline for dip"
date = 2024-06-10
tags = ["Deep Learning", "Computer Vision", "YOLOv8", "Image Forgery Detection", "OCR"]
description = "Comprehensive ID verification pipeline using YOLOv8, UNet, and DenseNet121 for detecting forged ID cards with multiple validation layers."
+++

## Overview

Built a multi-stage pipeline for verifying student ID cards using advanced deep learning models to detect various types of forgery and tampering.

## Pipeline Architecture

### Test Dataset
- 97 manually collected and cleaned ID card images
- Diverse ID card formats and designs

### Model Components

#### 1. Face Detection (YOLOv8)
- High-accuracy face detection in ID cards
- Fast inference for real-time processing

#### 2. Gender Validation
- DeepFace-based gender classification
- **Accuracy**: 80% (with potential for improvement via dedicated training)

#### 3. Splicing Detection
- **Model**: Image Splicing Detector (image_splicer_quant.tflite)
- **Architecture**: DenseNet121 with ELA preprocessing
- **Training Data**: CASIA dataset
- **Preprocessing**: Error Level Analysis (ELA)
- **Performance**:
  - Train accuracy: 97.35%
  - Validation accuracy: 82.31%
- **Output**: Splicing score (threshold: 0.90)
- **Average test score**: 0.95

#### 4. Copy-Move Detection
- **Model**: New UNet (new_unet.tflite)
- **Training**: COMOFOD dataset
- **Training**: 30 epochs
- **Accuracy**: 98.7%
- **Output**: Copy-move score (threshold: 0.22)
- **Average test score**: 0.19

### OCR & Text Validation
- **File**: speed_uppp2.py
- **Features**:
  - OCR text extraction
  - Text consistency check
  - Keyword matching (case/special character validation)
  - Keyword match percentage calculation
  - ELA analysis

## Pipeline Flow
1. **Face Detection** (FaceDetection_YOLO.py)
2. **Gender Validation** (gender_validation.py)
3. **Splicing Detection** (splicing_detector.py)
4. **Copy-Move Detection** (copymove_detector.py)
5. **OCR & Text Analysis** (speed_uppp2.py)

## Validation Results
- Successfully processes diverse ID card formats
- Robust detection of multiple forgery types
- Multi-layer validation ensures high accuracy
- Real-time processing capability

## Technologies Used
- YOLOv8
- TensorFlow Lite
- UNet
- DenseNet121
- DeepFace
- OCR (Optical Character Recognition)
- Error Level Analysis (ELA)
- Python
- Computer Vision
