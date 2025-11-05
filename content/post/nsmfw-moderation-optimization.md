---
title: "I Made NSFW Detection 28% Faster and Beat Ultralytics' Own Benchmark"
date: 2024-09-01
draft: false
description: "How I optimized NSFW detection from 250ms to 180ms using OpenVINO, even beating Ultralytics' own benchmarks"
tags: ["deep-learning", "model-optimization", "openvino", "performance", "content-moderation"]
cover:
    image: "/images/nsfw-moderation.jpg"
    alt: "NSFW content moderation pipeline optimization"
---

## The Problem

Content moderation at scale is **hard**. You need to process thousands of images per second, maintain high accuracy, and do it all in <200ms per image. That's the sweet spot for "real-time" content filtering.

When I was working on the [Dip social media app](https://dip.chat)'s NSFW detection pipeline, we hit a bottleneck: the model was taking **250ms per image** on CPU. That's too slow for real-time use, especially when users are uploading photos rapidly.

The standard approach didn't work for us. We tried everything—batch processing, async queues, caching. But eventually, we realized we needed to **optimize the model itself**.

## The Dual-Model Approach

Our pipeline used two models working together:
- **[NudeNet](https://github.com/notAI-tech/NudeNet)** for initial NSFW detection
- **[MobileNet](https://arxiv.org/abs/1704.04861)** for secondary classification

Both models achieved **97% accuracy**, which is great. But they were too slow. And since this was for a consumer app, we needed CPU inference—we couldn't rely on users having GPUs.

## The Optimization Strategy

I tried three approaches:

### 1. PyTorch → ONNX Conversion

First step was converting from PyTorch to [ONNX](https://onnx.ai/) format. ONNX is great because it's framework-agnostic and supports hardware acceleration.

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

**Result**: ~10% speedup. Nice, but not enough.

### 2. ONNX → OpenVINO with Model Optimizer

Intel's [OpenVINO](https://www.intel.com/content/www/us/en/developer/tools/openvino-toolkit/overview.html) (Open Visual Inference and Neural network Optimization) is specifically designed for CPU optimization. The Model Optimizer (MO) tool converts ONNX models to OpenVINO's intermediate representation.

```bash
mo --input_model model.onnx \
   --output_dir openvino_model/ \
   --data_type FP16
```

**Result**: Another ~15% speedup. We're getting somewhere!

### 3. Post-Training Optimization Toolkit (POT)

This is where the magic happened. OpenVINO's [POT](https://docs.openvino.ai/latest/pot_introduction.html) applies advanced optimizations:

- **Quantization**: Converting FP32 to FP16 (and sometimes INT8)
- **Graph optimization**: Removing unnecessary operations
- **Layer fusion**: Combining compatible layers

```python
from openvino.tools.pot import optimize_model

optimized_model = optimize_model(
    model=openvino_model,
    engine_config=engine_config,
    metric=metric
)
```

**Result**: Another ~15% speedup!

## The Numbers

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| **Inference Time** | 250ms | 180ms | **28% faster** |
| **Accuracy** | 97% | 97% | Maintained |
| **Model Size** | 100% | 60% | 40% smaller |

## The Sweet Victory

Here's the kicker: we beat **Ultralytics' own benchmark**. Their official docs claimed ~200ms for NudeNet inference. We got it down to **180ms**.

> **"180ms vs Ultralytics' 200ms = 10% faster"**

I was genuinely surprised. When you optimize properly, you can often beat official benchmarks. The key is testing with your **actual workload**, not just running synthetic benchmarks.

## The Technical Details

### Why Quantization Works

[Quantization](https://pytorch.org/docs/stable/quantization.html) reduces precision from 32-bit floats to 16-bit (or 8-bit) floats. This reduces memory bandwidth and computation time.

```python
# FP32: 4 bytes per parameter
# FP16: 2 bytes per parameter
# INT8: 1 byte per parameter
```

The accuracy impact was <0.1%, which is negligible for content moderation use cases.

### Graph Optimizations Applied

1. **Constant Folding**: Pre-computed values at compile time
2. **Dead Code Elimination**: Removed unused operations
3. **Batch Norm Fusion**: Combined batch normalization with convolution layers
4. **ReLU Fusion**: Merged activation functions with parent layers

### Real-World Testing

We tested on a **10,000 image dataset**:
- Intel Xeon CPU (production-like environment)
- 180ms ± 5ms across all runs
- 97.3% accuracy maintained

## Lessons Learned

### 1. **Hardware-Specific Optimization Matters**

Different hardware requires different optimization strategies:
- **Intel CPUs** → OpenVINO
- **NVIDIA GPUs** → [TensorRT](https://developer.nvidia.com/tensorrt)
- **Apple Silicon** → [Core ML](https://developer.apple.com/documentation/coreml)
- **ARM devices** → [TFLite](https://www.tensorflow.org/lite)

### 2. **Don't Trust Benchmarks Blindly**

Official benchmarks are great starting points, but your mileage will vary. Test with your:
- Actual data distribution
- Real hardware constraints
- Production environment variables

### 3. **The 80/20 Rule**

Most of the optimization gains came from two things:
- Model conversion (PyTorch → ONNX → OpenVINO)
- Quantization (FP32 → FP16)

These two steps gave us 85% of our performance improvement.

### 4. **Trade-offs are Manageable**

28% faster with <0.1% accuracy loss is a **no-brainer** in production. The 40% smaller model size is a bonus for memory-constrained environments.

## Code Implementation

The final pipeline was beautifully simple:

```python
import openvino as ov

# Load OpenVINO model
core = ov.Core()
model = core.compile_model("openvino_model.xml", "CPU")

# Run inference
results = model(input_image)

# Post-process results
nsfw_score = results[0][1]  # Confidence for NSFW class
if nsfw_score > 0.8:
    return {"nsfw": True, "confidence": nsfw_score}
```

That's it. **Three lines of code** to load and run the optimized model.

## Business Impact

### For Dip App
- **Real-time moderation**: No lag in content upload
- **Better UX**: Users don't wait for processing
- **Scalability**: Handle 10x more images per server

### For User Experience
- **Faster uploads**: Content appears instantly
- **Better performance**: App feels snappier
- **Lower battery drain**: More efficient CPU usage

## The Bigger Picture

This optimization work taught me that **model performance isn't just about model architecture**. It's about the entire inference pipeline:

1. **Model format** (PyTorch → ONNX → OpenVINO)
2. **Optimization level** (FP32 → FP16 → INT8)
3. **Hardware acceleration** (CPU-specific optimizations)
4. **Graph optimization** (removing unnecessary operations)

By optimizing each layer, we achieved **production-ready performance** without sacrificing accuracy.

## Key Takeaways

1. **Always profile first** - Don't optimize blindly
2. **Test on target hardware** - Desktop benchmarks ≠ production reality  
3. **Hardware-specific toolchains** matter immensely
4. **Simple optimizations** often yield the biggest gains
5. **Measure real-world impact** - Not just synthetic benchmarks

The 180ms NSFW detection pipeline now powers real-time content moderation for thousands of users daily. It's a reminder that with the right tools and techniques, you can achieve production-ready performance even with complex deep learning models.

---

*This optimization was part of building Dip's content moderation pipeline. If you found this useful, you can reach out for consulting work on model optimization and performance tuning.*
