---
title: 'Building a Drowsiness Detection Web App from scratch - pt1'
date: '2025-02-16T07:34:32+01:00'
draft: false
github_link: "https://github.com/AkanimohOD19A/DPL_Drowsiness_Detection"
description: "git init cohere_streamlit"
tags: ["data-science", "machine-learning", "cohere", "streamlit", "YOLOv6", "tutorial", "lifecycle"]
author: "AfroLogicInsect"
image: blog-images/blogs-placeholder.png
---

Have you ever wondered how to build a web app that can detect if you are feeling sleepy or alert? Do you want to learn how to use computer vision and machine learning to create a fun and useful project? Then this blog post is for you! In this post, we will see how to build a drowsiness detection web app from scratch, using _Python_, _LabelImg_, _PyTorch_ and _Streamlit_. 

This template is also reproducible for problems of binary classification. The stages include:
1. Import Dataset
2. Data Augmentation
3. Image Labelling
4. Buidling web application with Streamlit
5. Deploying web application


This will help you learn how to capture and process images from your webcam, how to train a deep neural network to recognize facial landmarks, which is a measure of how open or closed your eyes are. You will also learn how to deploy your web app on Streamlit, so you can share it with your friends and family. By the end of this post, you will have a working drowsiness detection web app that can detect when you need a break or a nap. Sounds exciting, right? Let's get started!

1. Import Dataset
Simply visit this [drive](https://drive.google.com/drive/folders/18RdVVSIW135g3t4zPi9Ys6-Enw7nu8Qh?usp=sharing) and save the images in a set directory. The drive contains two major paths, to the *drowsy* and *awake* images. The next step is necessary only if you want more dataset.

2. Data Augmentation
Use the following script to augment. First update your _requirements.txt_ file and install the __Augmentor__ module.
```
## data_augment.py

import Augmentor

img_list = ["./path/to/your/awake_images/", "./path/to/your/drowsy_images/"]

for image_folder in img_list:
    p = Augmentor.Pipeline(image_folder)

    ##Features
    p.rotate90(probability=0.5)
    p.rotate270(probability=0.5)
    p.flip_left_right(probability=0.8)
    p.flip_top_bottom(probability=0.3)
    p.crop_random(probability=1, percentage_area=0.5)
    p.resize(probability=1.0, width=360, height=240)

    ##Sample output
    p.sample(100)
```

this script simply gets your image folder and creates (according to the provided weights) a slight distortion of the original file with 100 outputs. Freeing our hands for the next step of labelling.

3. Image Labelling 
With the [LabelImg package](https://github.com/HumanSignal/labelImg) we can create grids around our subject input and staple a label to it. From the project directory, here's how we get started, 
- Add the following to your _requirements.txt_ file:
```
lxml
PyQt5
```

- Open cmd, create a labelImg directory and navigate to it.
```
cd [labelImg directory]
pyrcc4 -o libs/resources.py resources.qrc
For pyqt5, pyrcc5 -o libs/resources.py resources.qrc
```

Now, enter the following command to launch the program.
```
python labelImg.py
python labelImg.py [IMAGE_PATH] [PRE-DEFINED CLASS FILE]
```
> [IMAGE_PATH] is your directory's path
> [PRE-DEFINED CLASS FILE] is a path to a .txt file with predefined classes, one comes with the module by default.

That's it! start labelling, it will take a while, so take your time, we will see next week for how we can make real-time predictions and deploy to the internet.