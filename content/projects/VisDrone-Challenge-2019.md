+++
title = "VisDrone Challenge"
date = 2025-11-20
tags = ["Deep Learning", "Object Detection", "Object Tracking"]
description="A deep dive into YOLO-based UAV vision: object detection, single-object tracking, multi-object tracking, and real-world traffic analytics"
cover = {image="/videos/traffic_analysis.gif"}

+++

When you work with drone footage, nothing is easy. You’ve got shaky cameras, tiny objects, crowded scenes, occlusion everywhere, and the constant joy of dealing with aerial perspectives where cars look the size of matchboxes. When I started exploring the VisDrone 2019 dataset, I didn’t realize I was stepping into one of the most challenging computer vision benchmarks out there.

But that challenge became fun pretty quickly.

I wanted to build something end-to-end — not just object detection, not just tracking, but a complete UAV vision system: detect objects in images, detect in videos, track one vehicle persistently, track everything simultaneously, and finally use all of that to analyze traffic like a real-world AI surveillance system.

So I split the project into four modules: object detection, single-object tracking, multi-object tracking, and traffic analysis. Even though they build on each other, each one required solving a different kind of problem, and each one taught me something new about computer vision.

**Object Detection on VisDrone Images and Videos**

The starting point of everything was object detection. You can’t track anything if you can’t detect it first. The [VisDrone2019 dataset](https://github.com/VisDrone/VisDrone-Dataset) is collected by the AISKYEYE team at Lab of Machine Learning and Data Mining , Tianjin University, China. The benchmark dataset consists of 288 video clips formed by 261,908 frames and 10,209 static images, captured by various drone-mounted cameras, covering a wide range of aspects including location (taken from 14 different cities separated by thousands of kilometers in China), environment (urban and country), objects (pedestrian, vehicles, bicycles, etc.), and density (sparse and crowded scenes). Note that, the dataset was collected using various drone platforms (i.e., drones with different models), in different scenarios, and under various weather and lighting conditions. These frames are manually annotated with more than 2.6 million bounding boxes of targets of frequent interests, such as pedestrians, cars, bicycles, and tricycles. Some important attributes including scene visibility, object class and occlusion, are also provided for better data utilization.

I finetuned YOLOv8 because its State-of-the-Art and simply perfect for fast prototyping. After the fine tuning entire detection step barely requires 10 lines of code, which still feels illegal to me:

```python
from ultralytics import YOLO

model = YOLO("yolov8m.pt")
results = model("image.jpg")

for result in results:
    boxes = result.boxes
    print(boxes)
```
YOLOv8 handles everything behind the scenes preprocessing, non-maximum suppression, post-processing which is why the model is so great for UAV pipelines. The model was surprisingly robust even with VisDrone’s tiny pedestrians and distant vehicles. When running detection on full drone videos, YOLO’s streaming API made the process extremely smooth:
```python
for frame in model.predict(source="video.mp4", stream=True):
    detections = frame.boxes.xyxy
```
<img src="/images/visdrone2.jpg">

By this point, I had a working detection system. But detection alone doesn’t feel smart. It’s just finding objects, not understanding them. To make the system behave “intelligently,” I needed tracking.

**Single Object Tracking With Click-to-Select Re-Identification**

Single-object tracking sounds trivial until you actually try it. The challenge is not following an object frame to frame! The challenge is keeping the same ID even when the detector switches IDs, the object gets occluded, or it shrinks to 6 pixels because the drone flew higher.

I wrote a custom tracking script where you can click on any bounding box in the first frame, and the system will track that object across the entire video. Here's the exact code for the click-to-select logic from my script:
```python
def select_vehicle(event, x, y, flags, param):
    if event == cv2.EVENT_LBUTTONDOWN:
        for box in param['boxes']:
            x1, y1, x2, y2 = map(int, box[:4])
            if x1 <= x <= x2 and y1 <= y <= y2:
                selected_vehicle_id = box[4]
                print("Selected vehicle:", selected_vehicle_id)
                break
```
This part alone makes the pipeline feel interactive. I can visually inspect the video, click a car I’m interested in, and the rest of the system revolves around that one click.

But UAV tracking is dirty. IDs switch. Objects disappear. So I added a re-identification mechanism. It remembers the last known position of the target, and if YOLO loses the ID, it automatically reselects the closest detected box to the previous location:
```python
def reselect_vehicle(current_boxes):
    global selected_vehicle_id, last_position
    if selected_vehicle_id not in [int(b.id) for b in current_boxes]:
        best, best_id = float('inf'), None
        for box in current_boxes:
            center = (int(box.xywh[0][0]), int(box.xywh[0][1]))
            distance = np.linalg.norm(np.array(center) - np.array(last_position))
            if distance < best:
                best, best_id = distance, box.id
        selected_vehicle_id = best_id
```
It’s a simple heuristic but in drone footage, simple heuristics can outperform complex algorithms.

By the end of this part, I had a smooth, stable single-object tracking system that can lock onto a specific vehicle even when YOLO momentarily fails.

**Multi-Object Tracking With YOLO + ByteTrack/SORT-Style Logic**
While single-object tracking feels satisfying, multi-object tracking is where things start looking like an actual surveillance system. The idea is to detect every object in every frame and maintain consistent identities through time.

VisDrone videos can contain 20–100 moving objects, and YOLOv8’s built-in tracker is already aligned with the concepts from algorithms like SORT and ByteTrack. YOLO gives you consistent box.id values through its built-in tracking:
```python
for frame in model.track(source="video.mp4", stream=True):
    for box in frame.boxes:
        print("ID:", box.id, "Coordinates:", box.xyxy)
```
<video controls width="100%" style="margin: 20px 0;">
    <source src="/MOT.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>
Behind the scenes, YOLO’s tracker does three main things:

>Associates detections across frames
>Keeps track of disappeared objects
>Handles ID reassignment

It uses the same ideas as SORT: IoU matching + Kalman filtering. ByteTrack improves this by also linking low-confidence detections, which matters a lot in drone footage because distant vehicles often get low scores.

Once MOT is working, you suddenly see dozens of tiny colored labels dancing around a UAV video — every person, bike, and car with its own persistent ID. This part became crucial for my next step: traffic analysis.

**Traffic Analysis: Turning UAV Tracking Into Real-World Insights**

Tracking alone is cool, but I wanted to turn raw detections into something meaningful — something that looks like real analytics. With consistent IDs from the MOT module, traffic analysis becomes almost trivial. You can count vehicles, measure how long they stay in the frame, estimate congestion, and get per-class statistics.

For example, counting the number of unique vehicles in a video becomes a dictionary operation:
```python
vehicle_count = set()

for result in model.track(source="video.mp4", stream=True):
    for box in result.boxes:
        vehicle_count.add(int(box.id))

print("Vehicles detected:", len(vehicle_count))
```
If you define lane regions or zones, you can compute how many cars pass through each zone. If you track bounding box displacement across frames, you can approximate velocity.

A typical displacement-based speed rough estimate looks like this:
```
speed = np.linalg.norm(np.array(curr_center) - np.array(prev_center)) / time_elapsed
```
Of course, you need scale calibration for real-world units, but for relative motion analysis, even pixel-based speed works well.

<video controls width="100%" style="margin: 20px 0;">
    <source src="/traffic_analysis.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>

This part of the project made the system feel like an actual drone-powered traffic monitor not just a computer vision demo.
