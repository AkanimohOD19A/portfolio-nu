---
title: 'Wk 4 Experiment Tracking: MLOPs with DataTalks'
date: '2025-02-16T07:34:32+01:00'
draft: false
github_link: "https://github.com/AkanimohOD19A/MLOps_24"
description: "git init cohere_streamlit"
tags: ["MLOPs", "data-science", "machine-learning", "streamlit"]
author: "AfroLogicInsect"
image: blog-images/blogs-placeholder.png
---


Oh, you made it to the 4th week! Nice!

Here, we see  how to handle Deployment of our derived models, there are two major options for deployment - Stream or Batch Prediction services. <More?>

However, let's focus on how to get the [assignment](https://github.com/DataTalksClub/mlops-zoomcamp/blob/main/cohorts/2024/04-deployment/homework.md) done with.

**Q1: Standard deviation - Predicted duration for March 2023**
We already have the notebook we used in [Wk 1](https://github.com/DataTalksClub/mlops-zoomcamp/tree/main/cohorts/2024/04-deployment/homework) to achieve something like this we are just making adjustments - mainly to acquire the 2023 dataset.

###### 1.1 Create your Wk4 Directory and setup your environment
###### 1.2 Fetch the items in the link above, leave the items in the same directory

You would make a few changes in the _starter.ipynb_ file

![starter notebook](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/e3839p1degkrhsur63kl.png)

make it:
```
## Load Mar Data'23
df = read_data("https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2023-03.parquet")
```
to fetch 2023 records, in the next chunk, run:
```
## Isolating stated catgeories, call dict vectorizer and predict
dicts = df[categorical].to_dict(orient='records')
X_val = dv.transform(dicts)
y_pred = model.predict(X_val)
print(f"Q1. Standard Deviation of predictions: {round(y_pred.std(), 3)}")
```

**Q2: Preparing the output**
###### 2.1: Create an artificial **ride_id** column:
Simply create the _year_ and _month_ variable to set the *ride_id*
```
## Ride ID Column
year = 2023
month = 3

df["ride_id"] = f"{year:04d}/{month:02d}_" + df.index.astype("str")
```
Now, we can add the *ride_id* to the predictions with the following chunk:
```
## Save predictions to dataframe
df_results = pd.DataFrame(columns = ["predictions", "ride_id"])
df_results["predictions"] = y_pred
df_results["ride_id"] = df["ride_id"]
```
View the head of the result with `df_results.head()` it should look like:

![Prediction Head](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/owe5980uz8wfzkbvl7cj.png)

###### 2.2 Save the results with:
```
## Path to output file
import os
from datetime import date

output_pth = "../output_files"
if not os.path.exists(output_pth):
    os.makedirs(output_pth)

output_file = output_pth + "/results_" + str(date.today()) + ".parquet"

df_results.to_parquet(
    output_file,
    engine = 'pyarrow',
    compression = None,
    index = False
)
# Size of the output file
```
We could have easily visit the explorer folder to check the file size, but let's learn a little bash - run the following to retrieve the file sizes in the folder:

`! ls -lh ../output_files/* | awk '{print $5, $9}'`

**Remember** the exclamation(!) prefix is because you are in the jupyter notebook, you wouldn't need it if you were running this from the terminal.

-lh: The -l flag displays detailed information about each file (including permissions, owner, size, modification date, etc.), and the -h flag makes the file sizes human-readable (e.g., KB, MB, GB)

The result should look like:
--> **65M ../output_files/results_2024-06-08.parquet**

i.e 65

We are on track to automation.

**Q3. Creating the scoring script**
Just run:
```
!jupyter nbconvert --to script starter.ipynb
```
- starter.ipynb: is the name of your notebook.

**Q4 Pipenv**
Here, you have to `pip install pipenv pipenv --python 3.x` (put the appropriate python version in) `pipenv install <package>==<package.version>` to then install the necessary packages like _os_, _sklearn_, etc

Once this is done, indeed your **Pipfile** and **Pipfile.lock** files appear.

![Pipfile.lock](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zkbz0gmgkm7f5n694r8d.png)

**Q5: Parametrize the script**
Parametrize here means that we use/set system variable that can accept values, like the _year_, _month_, we initially set, such that our script can now run on its own from the terminal. **Automation**.

###### 5.1 Retrieve derived script
See how this is done in the derived _[script.py](https://github.com/AkanimohOD19A/MLOps_24/blob/main/wk4/notebooks/script.py)_ script, particularly this is set for taxi-type, year and month like so:

###### 5.2 Set Parameters
```
taxi_type = sys.argv[1] #yellow
year = int(sys.argv[2]) #2024
month = int(sys.argv[3]) #3
```
after you have the script, run the following in the terminal: `python starter.py yellow 2023 4` - a print statement would handle the mean predicted duration for April 2023.

You should have:
```
(MLOps_24) C:\Users\..\MLOps_24\wk4\notebooks>python starter.py yellow 2023 4
reading the data from https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2023-04.parquet...
loading the model with RUN_ID=../models/model.bin...
applying the model...
Q5. Mean of predictions: 14.292
saving the result to ../output_files/yellow/2023-04.parquet...
```

**Q6. Docker container** 
###### 6.1 What are Docker Containers?
[ChatGPT] A Docker container is a lightweight, **standalone**, and executable package of software that includes everything needed to run an application. It packages up code and all its dependencies, so the application runs quickly and reliably from one computing environment to another.

_Speed and Independence_ - it means we can container the requirements for the desired output and run our applications anywhere... You don't have to understand it all, right now - let's go.

###### 6.2 Create Docker and Launch Docker Engine:
In the same directory as we have been working all along; We simply create a _Dockerfile_, launch our Docker Engine locally and create a _requirements.txt_ that has a list and versions of our packages.

Update the _Dockerfile_ like you so:

```
# Use the base image with the model and vectorizer
FROM agrigorev/zoomcamp-model:mlops-2024-3.10.13-slim

# Set work directory
WORKDIR /app

# Copy your script into the container
COPY script.py .

# If you have any dependencies in a requirements.txt file, copy that in as well
COPY requirements.txt .

# Install any required packages
RUN pip install --no-cache-dir -r requirements.txt

# Run script
CMD [ "python", "script.py", "yellow", "2023", "5" ]
```

###### 6.2.1 Base Image Selection:

> FROM agrigorev/zoomcamp-model:mlops-2024-3.10.13-slim
: This line specifies the base image for our Docker container. It uses the image tagged as `mlops-2024-3.10.13-slim` from the `agrigorev/zoomcamp-model` repository.
6.2.2 Setting the Working Directory:

`WORKDIR /app`: This line sets the working directory inside the container to `/app`. Any subsequent commands will be executed relative to this directory.

###### 6.2.3 Copying Files into the Container:

`COPY script.py .`: This command copies the _script.py_ file from your local machine (the host) into the `/app` directory within the container.
`COPY requirements.txt .`: Similarly, it copies the requirements.txt file (if it exists) into the same directory.

###### 6.2.4 Installing Dependencies:

`RUN pip install --no-cache-dir -r requirements.txt`: This line installs Python packages listed in requirements.txt using pip. The `--no-cache-dir` flag prevents caching of downloaded packages.

###### 6.2.5 Running the Script:

`CMD [ "python", "script.py", "yellow", "2023", "5" ]`: The _CMD_ instruction specifies the default command to run when the container starts. In this case, it runs the _script.py_ Python script with the provided arguments: **"yellow"**, **"2023"**, and **"5"**

###### 6.3 Run Docker
Run from terminal: To build your docker image, replace **"<name>"** to any name of your choice, here "my-model" is used.  
So 
`docker build -t <name> .`
`docker run my-model`

```
$ docker run my-model
reading the data from https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2023-05.parquet...
loading the model with RUN_ID=model.bin...
applying the model...
Q6. Mean of predictions: 0.192
saving the result to ../output_files/yellow/2023-05.parquet...
```
That's it!

Let me know if you have any challenges.

**NB** If you are visiting the Author's repository [here](https://github.com/AkanimohOD19A/MLOps_24/tree/main/wk4), the difference between **starter.py** and **script.py** is the latter is built to have the pre-built model from the FROM agrigorev/zoomcamp-model:mlops-2024-3.10.13-slim docker image.
