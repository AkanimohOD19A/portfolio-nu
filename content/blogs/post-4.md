---
title: 'Deploying an Image Segmentation web application with YOLOv8 and Streamlit - pt1'
date: '2025-02-16T07:34:32+01:00'
draft: false
github_link: "https://github.com/AkanimohOD19A/medical_charges_v2"
description: "real-time: Image Segmentation web application with YOLOv8"
tags: ["data-science", "machine-learning", "streamlit", "YOLOv8", "tutorial", "lifecycle"]
author: "AfroLogicInsect"
image: static/images/blogs-placeholder.png
---

![Segmented Pieces](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pvgpuepoh09cat8bbph1.png)

We are going to create an object detecting web app, our goal is to to segment any image, note the distribution of the unique items in it, implement a slider view and ultimately deploy this on a web-application anyone can use.

First, let us have a basic understanding of the two building components, YOLOv8 is the newest state-of-the-art model developed by _Ultralytics_, that can be used for object detection, image classification, and instance segmentation tasks. YOLOv8 is designed to be fast, accurate, and easy to use,
Streamlit is a free and open-source framework to rapidly build and share beautiful machine learning and data science web apps. 

Let's start by creating a virtual environment from your IDE terminal.
`# pip install virtualenv`
`python -m venv [VIRTUALENVIRONMENTNAME]`

`pip install opencv streamlit-image-comparison pandas ultralytics streamlit
`

Or do this with conda
`conda create -n [VIRTUALENVIRONMENTNAME] python=3.x opencv streamlit-image-comparison pandas ultralytics streamlit`
`source activate [VIRTUALENVIRONMENTNAME] ##Activate`
and installing the requisite packages: **opencv, streamlit-image-comparison, pandas, ultralytics, streamlit**


Next, import the already tuned yolov8.pt model and introduce the streamlit with this code in your python script: 

```
from ultralytics import YOLO
import streamlit as st

# Downloads the model
model = YOLO('yolov8n.pt')
## Placed image in the same directory
img_file = 'bus.jpg' 
```

Then write a for-loop that goes through the image results and display each segment and predicted name.

```
## Run Model on image
results = model(img_file)  
img = cv2.imread(img_file) 

## Initiate Name bucket
names_list = [] 
 
## Handling the individual components in the results
for result in results:
    boxes = result.boxes.cpu().numpy()
    numCols = len(boxes)
    cols = st.columns(numCols) ## Dynamic Column Names
    ## Annotating the individual boxes
    for box in boxes:
        r = box.xyxy[0].astype(int)
        rect = cv2.rectangle(img, r[:2], r[2:], (255, 55, 255), 2)

    ## Use a slider to compare the various components
    st.markdown('')
    st.markdown('##### Slider of Uploaded Image and Segments')
    image_comparison(
        img1=img_file,
        img2=img,
        label1="Actual Image",
        label2="Segmented Image",
        width=700,
        starting_position=50,
        show_labels=True,
        make_responsive=True,
        in_memory=True
    )

    ## Add Slider to compare result and placed image
    st.markdown('##### Slider of Uploaded Image and Segments')
        image_comparison(
           img1=img_file,
           img2=img,
           label1="Actual Image",
           label2="Segmented Image",
           width=700,
           starting_position=50,
           show_labels=True,
           make_responsive=True,
           in_memory=True
       )

    ## Enumerate the individual result
    for i, box in enumerate(boxes):
        r = box.xyxy[0].astype(int)
        crop = img[r[1]:r[3], r[0]:r[2]] ## crop image 
        ## retrieve the predicted name
        predicted_name = result.names[int(box.cls[0])] 
        names_list.append(predicted_name)
        with cols[i]:
            ## Display the predicted name
            st.write(str(predicted_name) + ".jpg")
            st.image(crop)
```
![Image Slider](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tace81kbacuhtu4fy5ve.png)

Now, let's get the distribution of the items in a dataframe.

```
st.sidebar.markdown('#### Distribution of identified items')

df_x = pd.DataFrame(names_list)
summary_table = df_x[0].value_counts().rename_axis('unique_values').reset_index(name='counts')
st.sidebar.dataframe(summary_table)
```

![Image item counter](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bbrlc1xdgnqzz0ncqtwe.png)


With that done, it's time to launch this in your local browser with `streamlit run [NAMEOFAPP].py` in the terminal, then we move to actually deploying/sharing this site.

Create a requirements.txt with all the libraries you imported, it will look like this:

```
opencv-python-headless==4.7.0.72
streamlit-image-comparison==0.0.4
pandas==2.0.2
ultralytics==8.0.119
streamlit==1.23.1
```
 and packages.txt file with this single line: 

```
ffmpeg
```

Initiate Git Bash in the folder directory by running the following commands:
- `git init`
- `git add .`
- `git commit -m "my first image segmentation app"`

Go to your GitHub and create a repository where the project files would be hosted, push your local files there.

Once this has been done, simply toggle the bar on the top Right side of your local streamlit app and "Deploy App", it then takes us to the form section of deployment - enter the correct details, give the app a nice name and sweet - Deploy!

![Toggle Bar for Deployment](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/7sgocp5oslgex33gls81.png)

Which brings you to this page:

![Web Deployment Form](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hoyhqyrl1amzfryllhgz.png)

If everything goes well, you should land on something like [this](https://img-segmentation.streamlit.app/), however, if you experience any issues, do not hesitate to share your comments or check out this [repository](https://github.com/AkanimohOD19A/img-segmentation/tree/master) to compare codes.




