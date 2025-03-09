---
title: 'Building a Drowsiness Detection Web App from scratch - Pt2'
date: '2025-02-16T07:34:32+01:00'
draft: false
github_link: "https://github.com/AkanimohOD19A/DPL_Drowsiness_Detection"
description: "git init cohere_streamlit"
tags: ["data-science", "machine-learning", "cohere", "streamlit", "YOLOv6", "tutorial", "lifecycle"]
author: "AfroLogicInsect"
image: /images/blog-placeholder.png
---

# Pt2 - Building a Drowsiness Detection Web App from scratch

Cheers for making it to pt2; You've learnt how to Import datasets, augment that data and label the images, now we set up for real time predictions using the powerful YOLOv5 model and Google Colaboratory.

## Define Directories
Here, you would confirm the directories, because we would be navigating to them to train our model.
You should have your `images` and `labels` directory like so: 
> images
![image dir](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/y6hu1om8an50ob4u1klw.png)

> labels
![label dir](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zxie2b2kdrsht2dnr848.png)

Once you have confirmed the paths, we need to start training our data.

## Machine Learning
Not to be too literal, but we have got to the stage where the machine does all the work. While we can train this data on our local machine - would require a lot of computing energy, this tutorial shows you how to run this on Google Colab, which has a free tier for GPU.

### Data Directory
Let's Start:
1. Move your data folders into your Google Drive, create a folder and name it __data__, this will contain all data contents.

![Define the Classes](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ka3yv4n83i8h88yjj2d8.png)

2. Create a `dataset.yaml` file in your __data__ directory and paste the following:
```
## Creates the path to images for training and validation
train: /content/data/images
val: /content/data/images

## Defines the Classes
nc: 2  # number of classes
names: ['awake','drowsy']  # class names
```

### Launch Google Colab. 
You can do this from your parent folder - simply search for it, it looks like your __Jupyter__ notebook! Go the top right corner, that says __Connect__, select the __Change runtime type__ and choose 'GPU'.

![Set Runtime](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gauj4i8kkt5b9q9xqsin.png)

- Import Dependencies
```
import os
import shutil
import random
import torch
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
```
- Connect to your Drive
```
## This will mount your drive on the notebook
from google.colab import drive
drive.mount('/content/drive')
```
- Navigate to your image paths
```
images_path = "/content/drive/MyDrive/DROWSINESS/data/images"
```
- Clone the YOLO Model we would be training from
```
!git clone https://github.com/ultralytics/yolov5.git
## Navigate to the model
%cd yolov5/
## Install requirements
!pip install -r requirements.txt
## Download the YOLOv5 model
!wget https://github.com/ultralytics/yolov5/releases/download/v6.0/yolov5s.pt
```
- Start Training!
```
!python train.py --img 384 --batch 32 --epochs 1200 --data /content/drive/MyDrive/DROWSINESS/dataset.yaml --weights /content/yolov5/yolov5s.pt --nosave --cache
```
**NB**: You can change the number of epochs you would like to train for.

Now this will take a while.. Grab a cup of tea.

It's trained! Nice, it will save the trained model to your __weights__ folder with a last prefix, the next step is to test the validation of the predictions.

- Load Model and predict on sample image
```
## Load Modelmodel = torch.hub.load('ultralytics/yolov5', 'custom', path='runs/train/exp/weights/last.pt')
## sample Image
img = '/content/drive/MyDrive/DROWSINESS/data/images/awake_10.jpg'
## Plot prediction
result = model(img)
result.pandas().xyxy[0] ##Reference
%matplotlib inline
plt.imshow(np.squeeze(result.render(result)))
plt.show()
```
**NB** You may have trained multiplr times and this would mean your model would save with a different path, however it should have a 'last' prefix followed by '.pt'. So, check.

Now your model is tested, is it trusted? Maybe you want to train it again, this time with different parameters? But if you are satisfied - let see how we can download the model.

- Download the model/weights
```
from google.colab import files
files.download('/content/yolov5/runs/train/exp/weights/last.pt')
```
This will save the model to your local folder.

Here's a colab [notebook](https://colab.research.google.com/drive/12k-QCNNGLa38VC2xPcvWj7nhXIZWyE1h#scrollTo=tzkW_R_s_sGd) for reference.

The beauty of **YOLO** models to make predictions is not just that it is wicked efficient for computer vision, but that it is so light, we can actually serve it on a webpage. 

This is exactly what we would achieve in the final and **Pt 3** of this tutorial. See you next week, please share your questions and comments.