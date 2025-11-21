+++
title = "Real-Time Self-Driving Car with Lane Detection"
date = 2023-11-15
tags = ["Computer Vision", "Raspberry Pi", "OpenCV", "IoT"]
description = "Built a Raspberry Pi-based self-driving car capable of lane detection, obstacle avoidance, and path navigation using real-time image processing."
[cover]
image="images/autonomous_car.png"

+++

Developed an autonomous self-driving car using Raspberry Pi with advanced computer vision capabilities for lane detection and obstacle avoidance.

## Technical Implementation

### Image Processing Pipeline
- **Real-time image processing** with OpenCV
- Perspective transformation for bird's eye view
- Thresholding for lane detection
- Histogram-based lane tracking

### Hardware Integration
- Integrated multiple sensors:
  - IR sensors for obstacle detection
  - Ultrasonic sensors for distance measurement
  - Photoresistors for ambient light detection
- Real-time control logic for:
  - Steering adjustments
  - Acceleration control
  - Automatic braking

An autonomous car with image processing employs a sophisticated network of sensors to navigate its surroundings intelligently. Central to this system is a camera that captures real-time images, providing the foundational visual input for the car. These images undergo intricate image processing, facilitated by advanced algorithms, to recognize crucial elements such as lane markings, traffic signs, and obstacles. 

Complementing the camera, infrared (IR) sensors enhance the car's perceptual capabilities, especially in low-light conditions, by detecting obstacles through emitted and reflected infrared light.
 Ultrasonic sensors contribute to proximity sensing, utilizing sound waves to measure distances to nearby objects and aiding the car in navigation through confined spaces.
 Photodiode sensors play a pivotal role in maintaining the car's trajectory by detecting variations in light intensity and ensuring alignment with lane markings. 

The image processing model, running on a dedicated processing unit, synthesizes data from these sensors to generate a dynamic understanding of the environment. A sophisticated control system integrates this information, translating it into precise steering, acceleration, and braking commands. This amalgamation of image processing and sensor data enables the autonomous car to navigate predefined paths, respond to traffic signs, and adapt to its surroundings, marking a significant advancement in the realization of intelligent and safe autonomous transportation.

<img src="images/autonomous_car.png" | absURL>

How the Image Processing was implemented for lane detection:

Capturing Frames from the Camera

The camera is initialized with specific settings — frame width, height, brightness, etc.
Each frame is captured and converted from BGR to RGB (since OpenCV uses BGR by default).
```python
void Capture()
{
    Camera.grab();
    Camera.retrieve(frame);
    cvtColor(frame, frame, COLOR_BGR2RGB);
}
```

Perspective Transformation

We convert the front-facing view into a bird’s-eye (top-down) view using four defined source and destination points.

This simplifies the lane detection process since lane lines appear straight in this view.

```python
Matrix = getPerspectiveTransform(Source, Destination);
warpPerspective(frame, framePers, Matrix, Size(400,240));
```

Lane Line Isolation (Thresholding + Edges)

The system applies grayscale conversion, thresholding, and Canny edge detection to isolate lane markings.
```python
cvtColor(framePers, frameGray, COLOR_RGB2GRAY);
inRange(frameGray, 200, 255, frameThresh);
Canny(frameGray, frameEdge, 900, 900, 3, false);
add(frameThresh, frameEdge, frameFinal);
```

Histogram Analysis

To find lane positions, the code scans the bottom portion of the image column by column and computes a histogram of white pixel intensities.

```python
for(int i=0; i<400; i++)
{
    ROILane = frameFinalDuplicate(Rect(i,140,1,100));
    divide(255, ROILane, ROILane);
    histrogramLane.push_back((int)(sum(ROILane)[0]));
}
```

Detecting Lanes and Center Offset

The program identifies the maximum peaks on both sides of the histogram — these correspond to lane positions.

```python
LeftLanePos = distance(histrogramLane.begin(),
                       max_element(histrogramLane.begin(), histrogramLane.begin() + 150));

RightLanePos = distance(histrogramLane.begin() +250,
                        max_element(histrogramLane.begin() +250, histrogramLane.end()));
```
Then it calculates the lane center and compares it to the frame center (188 pixels):Then it calculates the lane center and compares it to the frame center (188 pixels):
```python
laneCenter = (RightLanePos - LeftLanePos)/2 + LeftLanePos;
Result = laneCenter - frameCenter;
```


Steering Decision via GPIO

Based on the Result value, specific GPIO pins are activated to signal movement direction.
Each pattern corresponds to a particular motion.

```python
if (Result == 0) { cout << "Forward"; }
else if (Result >0 && Result <10) { cout << "Right1"; }
else if (Result >=10 && Result <20) { cout << "Right2"; }
else if (Result >20) { cout << "Right3"; }
else if (Result <0 && Result >-10) { cout << "Left1"; }
else if (Result <=-10 && Result >-20) { cout << "Left2"; }
else if (Result <-20) { cout << "Left3"; }
```

Visualization and FPS

The system displays three live OpenCV windows:

>Original View (raw camera feed)

>Perspective View (bird’s-eye transformation)

>Final View (processed lane lines)

It also calculates and prints real-time FPS to monitor performance.

**Results**:

>Detects lanes in real-time at ~15–20 FPS on Raspberry Pi
>Outputs steering directions instantly
>Can be integrated with an RC car or motor driver for autonomous navigation