---
title: "I Made YOLOv8 Detection 3.7x Faster and Beat Ultralytics' Own Benchmark"
date: 2024-09-01
draft: false
description: "How I optimized YOLOv8's detection from 650ms to 180ms using OpenVINO, even beating Ultralytics' own benchmarks"
tags: ["deep-learning", "model-optimization", "openvino", "performance", "content-moderation"]
cover:
    
---

Every Social Media app needs Content moderation. At ByteCitadel I was given the task to make the Image Moderation pipeline to not let users to post NSFW content. But content moderation at scale is **hard**. You need to process hundreds of images per second, maintain high accuracy, and do it all in <200ms per image. That's the sweet spot for "real-time" content filtering.

When I was working on the [Dip social media app](https://dip.chat)'s NSFW detection pipeline, we hit a bottleneck: the whole pipeline was taking **650ms per image** on CPU. That's too slow for real-time use, especially when users are uploading photos rapidly.

When you're working on a pipeline with Image input and its prediction, the resource heavy and time consuming constraints to look out for are image resizing, preprocessing the input before the model prediction, and model's inference speed!!! Now why my pipeline was taking 650ms per image on CPU? I used two models, one [nudenet detector](https://github.com/notAI-tech/NudeNet/blob/v3/README.md) and other was [nsfw classifier](https://github.com/alex000kim/nsfw_data_scraper). The nsfw classifier was a mobilenetv2small.tflite and the detector was yolov8m.pt.

<img src="/images/nudenet.png">
<center>Nudenet detector output</center>

Things you should know for edge optimizations for your model at production levels, first what framework is your model saved in and which format is the best for your purpose!
Here:

 ON GPU, for running the model, [TensorRT](https://developer.nvidia.com/tensorrt#:~:text=NVIDIA%C2%AE%20TensorRT%E2%84%A2%20is,high%20throughput%20for%20production%20applications.) is currently the fastest format, it is an NVIDIA library specifically designed to maximize inference performance on NVIDIA GPUs through a series of aggressive optimizations. But if you're using CPU then TensorRT is not useful at all because it is a GPU specific library. 

 On CPU, for running model, consider [ONNX-runtime](https://onnx.ai/) or if using intel CPU use [OpenVINO](https://github.com/openvinotoolkit/openvino). OpenVINO is typically the top performer on Intel CPUs, as it is specifically tailored to leverage Intel's architecture and instruction sets, whereas ONNX Runtime offers excellent, highly portable CPU performance across various hardware (including non-Intel CPUs) and is a strong general-purpose choice for cross-platform deployment. But if your main goal is to deploy the model on a mobile app, use [tflite](https://ai.google.dev/edge/litert)!!!

<img src="/images/comparison_of_models.png">

## The Optimization Strategy

Anytime I have to reduce latency of any, I used python's time library on every function of the pipeline to understand which is the most time consuming. In my pipeline, both the models had different input sizes, the detector's input size was 640x640, and classifier's input size was 224x224 so I needed to do resizing two times using **opencv-python**

```python 
import cv2
import imutils

image = cv2.imread('image.png')

cv2.imshow('Original Image', image)
cv2.waitKey(0)
```

One thing you should know about opencv is how ridiculously slow it is at resizing an image, and top of it I had to do it twice! So the zero step I took was running both the models in parallel using threading, and reduce reads and writes by making the code modular! Adding abstraction to code seriously helps you find out the unnecessary calls happening to other unwanted functions for one task. 

As i made my code modular and ran both models in parallel, the latency of whole pipeline reduced to **650ms to 580ms**!

### 1. PyTorch → ONNX Conversion

First step was converting from PyTorch to [ONNX](https://onnx.ai/) format. ONNX is great because it's framework-agnostic and supports hardware acceleration.

```python
torch.onnx.export(
    model,
    dummy_input,
    "model.onnx",
    export_params=True,
    opset_version=11
)
```

**Result**: 580ms to 350ms, but still not enought fast.

<img src="/images/yolov8.png">

As you can see in the above table here, the YOLOv8m model has minimum latency of 234.7ms on ONNX-Runtime. With that latency and add ons of classifier and image resizing, my overall latency was 350ms! I still needed to optimize it further to run it under 200ms but I how do I best the ultralytics own benchmarks on cpu?

### 2. ONNX → OpenVINO with Model Optimizer

Intel's [OpenVINO](https://www.intel.com/content/www/us/en/developer/tools/openvino-toolkit/overview.html) (Open Visual Inference and Neural network Optimization) is specifically designed for CPU optimization. The Model Optimizer (MO) tool converts ONNX models to OpenVINO's intermediate representation.

```bash
mo --input_model model.onnx \
   --output_dir openvino_model/ \
   --data_type FP16
```

**Result**: Another speedup. We got from 350 to 250ms!

### 3. Post-Training Optimization Toolkit (POT)

This is where the magic happened. OpenVINO's [POT](https://docs.openvino.ai/latest/pot_introduction.html) applies advanced optimizations:

- **Quantization**: Converting FP32 to INT8
- **Graph optimization**: Removing unnecessary operations
- **Layer fusion**: Combining compatible layers
### Why Quantization Works

[Quantization](https://pytorch.org/docs/stable/quantization.html) reduces precision from 32-bit floats to 16-bit (or 8-bit) floats. This reduces memory bandwidth and computation time.

```python
# FP32: 4 bytes per parameter
# FP16: 2 bytes per parameter
# INT8: 1 byte per parameter
```

```python
from openvino.tools.pot import optimize_model

optimized_model = optimize_model(
    model=openvino_model,
    engine_config=engine_config,
    metric=metric
)
```

**Result**: Finally we got the latency to **180ms**!!

## The Numbers

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| **Inference Time** | 650ms | 180ms | **3.7x faster** |
| **Accuracy** | 97% | 97% | Maintained |
| **Model Size** | 100% | 60% | 40% smaller |

## The Sweet Victory

Here's the kicker: we beat **Ultralytics' own benchmark**. Their official docs claimed ~234.7ms for NudeNet inference. We got it down to **180ms**.

> **"180ms vs Ultralytics' 200ms = 10% faster"**

## The Technical Details

The accuracy impact was not even <0.1%.

### Graph Optimizations Applied

1. **Constant Folding**: Pre-computed values at compile time
2. **Dead Code Elimination**: Removed unused operations
3. **Batch Norm Fusion**: Combined batch normalization with convolution layers
4. **ReLU Fusion**: Merged activation functions with parent layers

Different hardware requires different optimization strategies:
- **Intel CPUs** → OpenVINO
- **NVIDIA GPUs** → [TensorRT](https://developer.nvidia.com/tensorrt)
- **Apple Silicon** → [Core ML](https://developer.apple.com/documentation/coreml)
- **ARM devices** → [TFLite](https://www.tensorflow.org/lite)

### **The 80/20 Rule**

Most of the optimization gains came from two things:
- Model conversion (PyTorch → ONNX → OpenVINO)
- Quantization (FP32 → FP16)

These two steps gave us 85% of our performance improvement.

### **Trade-offs are Manageable**

3.7x faster with <0.1% accuracy loss is a **no-brainer** in production. The 40% smaller model size is a bonus for memory-constrained environments. The 180ms NSFW detection pipeline now powers real-time content moderation for thousands of users daily. It's a reminder that with the right tools and techniques, you can achieve production-ready performance even with complex deep learning models.

---

*This optimization was part of building Dip's content moderation pipeline. If you found this useful, you can reach out for consulting work on model optimization and performance tuning.*
