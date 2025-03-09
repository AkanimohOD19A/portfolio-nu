---
title: 'Wk 1: MLOPs with DataTalks'
date: '2025-02-16T07:34:32+01:00'
draft: false
github_link: "https://github.com/AkanimohOD19A/MLOps_24"
description: "git init cohere_streamlit"
tags: ["MLOPs", "data-science", "machine-learning", "streamlit"]
author: "AfroLogicInsect"
image: blog-images/blogs-placeholder.png
---

Recently joined the [DataTalks 2024 cohort](https://github.com/DataTalksClub/mlops-zoomcamp) to earn a **MLOps** Certificate and essentially build on Machine Pipeline competencies. To complete the course are assignments that have to be completed every other week.

This will be a series on how the Author approaches these assignments and serve as solutions to those struggling.

**Week 1**
The assignment here is basic, you would have some requisite skill in python, ml libraries and bash scripting to see this through. See the Homework below:

![Homework_Description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/d7ipr4wbcxtyp57vhx67.png)

We will build a __jupyter notebook__ that addresses each question. 

Before anything, create a directory to house your work, now and later, like so:

```
MLOPS 
 |
 - wk1
```

Then create a virtual environment in the parent directory, this is where you install all the require packages for the whole journey:
1. Launch your bash and run the following commands:
```
cd MLOPS
python3.10 -m venv MLOPS_venv
source MLOPS_venv/Scripts/activate
```
This would create the virtual environment __MLOPS_venv__ with Python 3.10 as well as a directory in your parent folder with the same name, Now you can install packages in this environment. The last line is to activate this environment.

And to deactivate:
`deactivate`

#### Wk1:
###### Setup
```
mkdir wk1
cd wk1
mkdir datasets
code .
```
This makes the **wk1** directory to this week's work, if you haven't created it before, then navigates into it to create a _datasets_ subdirectory after which it launches [VS code](https://www.bing.com/ck/a?!&&p=ff50085fbff1f9b7JmltdHM9MTcxODU4MjQwMCZpZ3VpZD0zODA4ZWQwNS00YmRlLTYwYjQtMjNhNi1mZWZiNGEwMzYxOWMmaW5zaWQ9NTUwMg&ptn=3&ver=2&hsh=3&fclid=3808ed05-4bde-60b4-23a6-fefb4a03619c&psq=vs+code+community+download&u=a1aHR0cHM6Ly92aXN1YWxzdHVkaW8ubWljcm9zb2Z0LmNvbS9kb3dubG9hZHMv&ntb=1), **Ctrl+Shift+P** is the command to create a notebook, name it _homework_.

When this has been created, make sure you set the kernel to the _MLOPS_venv_ environment. 

###### Jupyter Notebook
In your _homework.ipynb_ notebook file, run `!ls` to see that you have the needed directories, it should look like this:

- datasets
- homework.ipynb

Then install a few libraries like so:
`## Install Packages
!pip install numpy pandas seaborn scikit-learn`

**!** - This is used in Jupyter notebooks to run shell commands.

**Q1: Green Taxis - Download the data for January and February 2023.**
1.1 Download datasets
```
## Download Yellow Taxi Trips Files
! curl -o ./datasets/jan_yellow.parquet https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2023-01.parquet
! curl -o ./datasets/feb_yellow.parquet https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2023-02.parquet
```
**curl** is a tool for transferring data from or to a server. Here’s what each part does:

`curl` - The command-line tool for making requests to URLs.
`-o `- This option tells curl to save the output to a file instead of displaying it.
`./datasets/jan_yellow.parquet` - The path where the first file will be saved.

So, the first command downloads a file named _yellow_tripdata_2023-01.parquet_ from the given URL and saves it as _jan_yellow.parquet_ in the `datasets` directory.

The second command does the same for a different file, saving it as _feb_yellow.parquet_.

1.2 Import Libraries
```
## Load Libraries
import numpy as np
import pandas as pd
from sklearn.feature_extraction import DictVectorizer

## Load Dataset
jan_df = pd.read_parquet("./datasets/jan_yellow.parquet")
print(f"1, Data Dimension: {jan_df.shape[0]} rows | {jan_df.shape[1]} columns \n")
```
=> Data Dimension: 3066766 rows | 20 columns 
The output of this returns the answer to the question.

**Q2: Compute the duration variable (in minutes) and fetch the standard deviation of the trips duration in January?**

2.1 Compute Trip duration & Std Deviation

```
jan_df[["tpep_pickup_datetime", "tpep_dropoff_datetime"]] = jan_df[["tpep_pickup_datetime", "tpep_dropoff_datetime"]].apply(pd.to_datetime)
jan_df["duration"] = (jan_df["tpep_dropoff_datetime"] - jan_df["tpep_pickup_datetime"]).dt.total_seconds()/60

print(f"2, Duration Standard Deviation: {jan_df['duration'].std()} \n")
```

=> Duration Standard Deviation: 42.59435124195458.

This code converts the “tpep_pickup_datetime” and “tpep_dropoff_datetime” columns in the jan_df DataFrame to datetime objects using pandas’ to_datetime function. Then, it calculates the duration of each trip by subtracting the pickup time from the dropoff time, converting the result to total seconds, and then dividing by 60 to get the duration in minutes.

**Q3: Drop Outliers**
```
filtered_duration = jan_df[jan_df['duration'].between(1,60)]
clean_prop = len(filtered_duration['duration'])/len(jan_df['duration'])

print(f"3, Outlier Proportion: {clean_prop} \n")
```
~> 98%
We filter `jan_df` to include only the rows where the "duration" column values are between 1 and 60 minutes. This forms about 98% of the initial dataframe.

**Q4: Dimentionality of Feature Matrix**
```
## Filtered columns
ml_df = filtered_duration[['PULocationID', 'DOLocationID']].astype(str)
ml_df['duration'] = filtered_duration['duration']

## Dictionaries
dicts_train = ml_df[['PULocationID', 'DOLocationID']].to_dict(orient='records')
dicts_train[1:5]

## Vectorizers
vec = DictVectorizer(sparse = True)
feature_matrix = vec.fit_transform(dicts_train)

print(f"4, Dimension of feature_matrix: {feature_matrix.shape} \n")
```
=> 4, Dimension of feature_matrix: (3009173, 515)

This code does the following:

- Creates a new DataFrame ml_df with only the ‘PULocationID’ and ‘DOLocationID’ columns from filtered_duration, converting them to strings.
- Adds the ‘duration’ column from filtered_duration to ml_df.
- Converts ml_df into a list of dictionaries with ‘PULocationID’ and ‘DOLocationID’ as keys, using the to_dict method with orient='records'.
- Initializes a DictVectorizer which is used to convert the list of dictionaries into a matrix of features for machine learning models.
- Transforms the list of dictionaries into a sparse matrix feature_matrix.
- Prints the dimensions of feature_matrix.
- The output will show the number of rows and columns in the feature matrix.

**Q5: Training a Linear Regression Model**
```
## Linear Regression Model
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error

y = ml_df['duration']

model = LinearRegression()
model.fit(feature_matrix, y)
y_pred = model.predict(feature_matrix)
rmse = np.sqrt(mean_squared_error(y, y_pred))

print(f"5, RMSE: {rmse}")
```
=> RMSE: 7.649262236295703

Here, we call the **LinearRegression** models from the scikit-learn/sklearn library and train on the target variable **duration**, fit the model to a feature matrix and then predict. 

The Root Mean Squared Error (RMSE) is calculated based on the differences between the actual and predicted values of the Target Variable, the lower the value, the better.

Q6: Evaluating the Model

Here, we apply all we've done to the validation _Feb_ dataset, by simply creating a function:
```
## Compile chunks into a function
def rmse_validation(df_pth: str):
    val_df = pd.read_parquet(df_pth)
    val_df[["tpep_pickup_datetime", "tpep_dropoff_datetime"]] = val_df[["tpep_pickup_datetime", "tpep_dropoff_datetime"]].apply(pd.to_datetime)
    val_df["duration"] = (val_df["tpep_dropoff_datetime"] - val_df["tpep_pickup_datetime"]).dt.total_seconds()/60
    val_df = val_df[val_df['duration'].between(1,60)]

    val_df[['PULocationID', 'DOLocationID']] = val_df[['PULocationID', 'DOLocationID']].astype(str)
    dicts_val = val_df[['PULocationID', 'DOLocationID']].to_dict(orient='records')
    
    feature_matrix_val = vec.transform(dicts_val)
    #print(f"Dimension of feature_matrix: {feature_matrix_val.shape} \n")

    y_val = val_df['duration']
    y_pred = model.predict(feature_matrix_val)
    rmse = np.sqrt(mean_squared_error(y_val, y_pred))

    return rmse

result_feb_df = rmse_validation("./datasets/feb_yellow.parquet")
print(f"6, Validation_RMSE: {result_feb_df}")
```
=> 6, Validation_RMSE: 7.811812822882009

The only difference here is the distinction between __fit_transform__ and __transform__ as it applies to the vectorizer, we use transform in the validation set to simply inherit the fitting transformation earlier done on the training set.

That's it! 
Visit [wk1_submission](https://github.com/AkanimohOD19A/MLOps_24/tree/main/wk1) to review the codes and Cheers!
Comment below if there are any issues.