---
title: "Optimizing NSFW Content Moderation: From 250ms to 180ms"
date: 2024-09-01
draft: false
description: "How I optimized NSFW detection inference time by 28% using OpenVINO, beating Ultralytics' own benchmarks"
tags: ["deep-learning", "model-optimization", "openvino", "performance", "content-moderation"]
cover:
    image: "/images/nsfw-moderation.jpg"
    alt: "NSFW content moderation pipeline optimization"
---

## Introduction

Content moderation at scale requires **real-time processing** with **high accuracy**. When working on the Dip social media app's NSFW detection pipeline, I faced a challenge: the inference time was too slow for real-time use.

This post shares how I optimized the pipeline from **250ms to 180ms** per image - a **28% improvement** that even beat Ultralytics' own benchmarks.

## The Problem

### Initial Performance
- **Inference Time**: 250ms per image (CPU)
- **Accuracy**: 97%
- **Target**: <200ms for real-time moderation
- **Benchmark**: Ultralytics' own performance standards

The pipeline used a dual-model approach:
1. **NudeNet** for NSFW detection
2. **MobileNet** for secondary classification

## Optimization Strategy

### 1. Model Conversion: PyTorch → ONNX

The first step was converting from PyTorch to ONNX format:

```python
# Convert PyTorch to ONNX
torch.onnx.export(
    model,
    dummy_input,
    "model.onnx",
    export_params=True,
    opset_version=11
)
```

**Benefits of ONNX**:
- Framework-agnostic format
- Better hardware acceleration
- Reduced inference overhead

### 2. ONNX → OpenVINO Conversion

Using OpenVINO's Model Optimizer (MO):

```bash
mo --input_model model.onnx \
   --output_dir openvino_model/ \
   --data_type FP16
```

**Why OpenVINO**:
- Intel's optimization toolkit
- Hardware-specific optimizations
- Significant performance gains on Intel CPUs

### 3. Post-Training Optimization (POT)

Applied OpenVINO's Post-Training Optimization Toolkit:

```python
from openvino.tools.pot import optimize_model

optimized_model = optimize_model(
    model=openvino_model,
    engine_config=engine_config,
    metric=metric
)
```

**POT Optimizations**:
- **Quantization**: Reduced precision (FP32 → FP16)
- **Graph optimization**: Removed unnecessary operations
- **Layer fusion**: Combined compatible operations
- **Pruning**: Removed redundant weights

## Performance Results

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| **Inference Time** | 250ms | 180ms | **28% faster** |
| **Accuracy** | 97% | 97% | Maintained |
| **Model Size** | Original | Reduced | ~40% smaller |
| **Memory Usage** | High | Optimized | ~30% reduction |

### Beat the Benchmark
- **Our Result**: 180ms
- **Ultralytics Benchmark**: ~200ms
- **Achievement**: 10% faster than the official benchmarks

## Technical Deep Dive

### Quantization Impact
- **FP32 → FP16**: ~2x speedup
- **Accuracy Impact**: <0.1% accuracy loss
- **Memory Savings**: 40-50% reduction

### Graph Optimizations Applied
1. **Constant Folding**: Pre-computed static values
2. **Dead Code Elimination**: Removed unused operations
3. **Batch Normalization Fused**: Combined with convolution layers
4. **ReLU Fused**: Merged activation functions

### Real-World Testing
- **Test Set**: 10,000 diverse images
- **CPU**: Intel Xeon (production-like environment)
- **Consistency**: 180ms ± 5ms across runs

## Key Learnings

### 1. Model Conversion Matters
- Converting between formats can yield significant gains
- ONNX is a great intermediate representation
- OpenVINO provides hardware-specific optimizations

### 2. Don't Trust Benchmarks Blindly
- Official benchmarks may not match your use case
- Test with your actual data and hardware
- Small changes can have big impacts

### 3. Hardware-Specific Optimizations
- Different models benefit from different optimizations
- Intel CPUs → OpenVINO optimization
- NVIDIA GPUs → TensorRT optimization
- ARM devices → specific toolchains

### 4. Trade-offs are Manageable
- Minimal accuracy loss for big speed gains
- 28% faster with <0.1% accuracy drop is acceptable
- Always measure, don't assume

## Code Implementation

The complete pipeline:

```python
import openvino as ov

# Load OpenVINO model
core = ov.Core()
model = core.compile_model("openvino_model.xml", "CPU")

# Inference
results = model(inputs)
```

**Simple, fast, and effective!**

## Impact on Production

### User Experience
- **Real-time moderation**: No lag in content upload
- **Better UX**: Smooth app performance
- **Scalability**: Handle more images per second

### Business Value
- **Server Costs**: Fewer resources needed
- **Latency**: Improved response times
- **User Retention**: Better app experience

## Conclusion

Model optimization is not just about code tweaks - it's about understanding the entire pipeline:
- **Format conversion** (PyTorch → ONNX → OpenVINO)
- **Quantization** for speed and size
- **Hardware-specific** optimizations
- **Measured improvements**, not assumptions

The 28% performance improvement demonstrates that with the right tools and techniques, we can achieve production-ready performance even with complex deep learning models.

### Takeaways
1. Always profile before optimizing
2. Test multiple optimization approaches
3. Hardware-specific toolchains matter
4. Measure real-world impact, not just benchmarks

**Result**: A 180ms NSFW detection pipeline ready for real-time content moderation at scale.

---

*This optimization work was part of the content moderation pipeline for the Dip social media app, enabling safer user-generated content.*
