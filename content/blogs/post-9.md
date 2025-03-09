---
title: 'Building a Drowsiness Detection Web App from scratch - pt3'
date: '2025-02-16T07:34:32+01:00'
draft: false
github_link: "https://github.com/AkanimohOD19A/DPL_Drowsiness_Detection"
description: "git init cohere_streamlit"
tags: ["data-science", "machine-learning", "cohere", "streamlit", "YOLOv6", "tutorial", "lifecycle"]
author: "AfroLogicInsect"
image: blog-images/blogs-placeholder.png
---

In the final leg of this Tutorial, we start to build a webpage using __Streamlit__ that can perform three things:
1. Give some description of the Application
2. Take an image input
3. Provide predictions/output on #2

Streamlit is an open-source app framework for Machine Learning and Data Science teams. It is a Python-based library specifically designed for machine learning engineers that does this well and given all the resources from our previous work, we can set to build this service - all in python.

## Create a Simple Description of our service
Here we will interact with the markdown functions of _Streamlit_ library, first, create an `app.py` script in your local directory and paste the following.
```
## Import the Neccessary Libraries
### Don't forget to update your --requirements.txt file
import streamlit as st
from PIL import Image
import numpy as np
import torch
import io
import os

## Put some description of the service
st.title("DROWSINESS DETECTION")
st.subheader("An implementation of Machine Learning to determine when user might be feeling a little drowsy.")
st.markdown("---")
```

Next, in the same script we load our saved model - and yes! it has to be your folder directory. The following loads the trainned model as well as the torch hub that holds the YOLOv5 pre-trained model, then we evaluate with `model.eval()`.

```
## Load Model
#e.g run_model_path = './last_drowsy_v4.pt'
run_model_path = './model_name.pt'
model = torch.hub.load('ultralytics/yolov5', 'custom', path=run_model_path)
model.eval()
```

Great!, next, we create a function that takes our model and the input image to output the prediction as a bounding box, below is how the __predict_img__

```
def predict_img(img):
    ## Flatten out the image
    image = np.array(Image.open(img))
    
    ## predict
    result = model(image)

    ## Output
    output = io.BytesIO()
    out_image = np.squeeze(result.render())
    output_img = Image.fromarray(out_image)
    output_img.save(output, format='JPEG')
    result_img = output.getvalue()

    return st.image(result_img)
```
## Take an image input & Provide predictions/output
Phew, now, let's handle the dial for handling image uploads using the __file_uploader__ on the sidebar of our app:
```
img_file_buffer = st.sidebar.file_uploader("Upload an Image", type=["jpg", "jpeg", "png"])
    if img_file_buffer is not None:
        UPLOADED_IMAGE = img_file_buffer
        predict_img(UPLOADED_IMAGE)
```
Notice how we declared the various image formats our app can accept as well as how we handled for __if__ there are no Uploaded files.

That-is-it! Easy right?

Navigate to your terminal and run:
> streamlit run app.py

On the Top-Right corner, you should see an option to "Deploy App", please fire away and share how it turns out. The Author did and you can find the [repository](https://github.com/AkanimohOD19A/DPL_Drowsiness_Detection) and [deployed application](https://drowsiness-detection.streamlit.app/).

![Drowsiness Detection](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mdcplttit6ks4g84xa4t.png)

Yay! We made it to the finish line. Now take this approach and create your own web-services powered by ML.