+++
title = "Individual Identification of Mugger Crocodiles using YOLOv8"
date = 2025-10-10
tags = ["Deep Learning", "Computer Vision", "Research"]
[cover]
image= "images/mugger_cover.jpg"
+++

**Abstract**

>Individual identification contributes significantly towards investigating behavioral mechanisms of animals and understanding underlying ecological principles. Most studies employ invasive procedures for individually identifying organisms. In recent times, computer-vision techniques have served as an alternative to invasive methods. However, these studies primarily rely on user input data collected from captivity or from individuals under partially restrained conditions. Challenges in collecting data from free-ranging individuals are higher when compared to captive populations. However, the former is a far more important priority for real-world applications.


Individual Identification of mugger crocodile, is where my computer vision journey started. I was part of a deep learning research project under **Prof. Mehul Raval**, where we were provided a dataset of free-ranging muggger crocodiles collected using Unmanned Aerial Vehicle (UAV). The dataset contained total 160,000 images focusing on mugger's dorsal body. The data was collected from 160 individuals across 19 different locations along the western part of India.
This was an extension to an already done [research](https://india.mongabay.com/2022/11/identifying-individual-mugger-crocodiles-using-drone-technology-to-minimise-conflicts/).

<img src="images/mugger.png" width=300>

Using a CNN model, we aim to individually identify free ranging mugger crocodiles, Crocodylus palustris based on their dorsal scute patterns. Scutes are hard calcium bony plates called osteoderms (Fig. 1) that form a crocodile's dorsal body. It serves as a unique identifier based on its placement on the dorsal body. With this background, we hypothesize that the developed CNN-based IID for mugger crocodiles will provide a robust understanding of behavioral processes and physiological patterns at an individual level, which in turn will contribute towards species conservation. 

The dataset had 160,000 images and their annotations (bounding boxes of scute patterns), we decided to use YOLOv8 as it is the state of the art object detection model, known for its speed and accuracy. The size of dataset was ~450GB, for data preprocessing and model training we were assigned access to ParamShavak supercoputer which use NVIDIA A6000 GPU!

>**Only problem paramshavak did not have GUI, so we coded whole summer in pUTTY cli, doing image processing tasks on a machine where images couldn't open, very ironic right!**

**Data Preprocessing**

Standard data preprocessing steps before any object detection model training are image processing which includes, image resizing, normalization and data augmentation. Also annotation processing is very important step, Adjust bounding box coordinates to match the image transformations applied during preprocessing (e.g., resizing, cropping, flipping). Format Conversion, Convert annotations to the format expected by the chosen object detection framework (e.g., COCO, PASCAL VOC, YOLO format). 

As we were using YOLOv8, we resized every image to 640x640, applied normalization and converted annotations into YOLO txt format. Another step taken was out of 1000 images per class, we only used 250 images per class in training by taking every 4th frame, divided that train data into 90:10 ratio for train and validation. Other 750 images per class were taken as unseen testing data.

**Model Training**

Since we already had selected the model, next step was model configuration. Configured hyperparameters using  Adam optimizer, batch normalization and 100 training epochs.

For yolov8 training you need to prepare a YAML mapping file where you need to give path to train data, val data and mapping of classes.

```cli
yolo train model=yolov8m.pt data=mapping.yaml epochs=100 imgsz=640
```

This was first training version which acheived 97.5% mAP50, after that did hyperparameter tuning with [train-settings]("https://docs.ultralytics.com/modes/train/#train-settings") and the images-per-class which varied from 50,100,150,200,500 and even taking all 1000. But the final best accuracy acheived was with optimizer Adam, aumentations settings like perspective, flipud, fliplr, scale and 250 images per class whcih acheived **99.5% mAP50**.

For testing, we used new data collected six months after the training dataset. Because of the time gap, the weather conditions had changed significantly, creating a noticeable difference between the training and test images.

We used two parameters, True Positive Rate (TPR) and True Negative Rate (TNR), to validate the efficiency of the trained models. Using YOLO-v5l, TPR (re-identification of trained muggers) and TNR (differentiating untrained muggers as ‘unknown’) values at the 0.84 decision threshold were 88.8% and 89.6%, respectively. The trained model showed 100% TNR for the non-mugger species, the Gharial, Gavialis gangeticus, and the Saltwater crocodile, Crocodylus porosus. 


<img src="images/yolov8_output1.png" width=600 | absURL>
<img src="images/yolov8_output2.png" width=600 | absURL>


**Metrics**

<img src="images/confusion_matrix.png" width =600 | absURL>

This confusion matrix displays the predicted values on the y-axis and the true values on the x-axis. The diagonal line represents perfect predictions, where the predicted value matches the true value. 


<img src="images/other_metrics.png" | absURL>

precision(B): This metric shows some fluctuations but generally increases over time, indicating improving precision performance. recall(B): Similar to precision, the recall metric also improves as training progresses. mAP50(B): This metric, likely mean Average Precision with an IoU threshold of 0.5, starts low but steadily increases, suggesting better object detection/localization performance. mAP50-95(B): This metric, probably mean Average Precision averaged over multiple IoU thresholds (0.5 to 0.95), also improves consistently during training.

The performance of the CNN model was reliable and accurate while using only 250 images per individual for training purposes. Showing that a bounding box approach (YOLO-v5l model) with background elimination is a promising method to individually identify free-ranging mugger crocodiles. Our manuscript demonstrates that UAV imagery appears to be a promising tool for non-invasive collection of data from free-ranging populations. It can be used to train open-source algorithms for individual identification. Further, the identification method is entirely based upon dorsal scute patterns, which can be applied to different crocodilian species, as well.