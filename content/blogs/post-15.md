---
title: 'Zenml for beautiful beautiful orchestration Pt1'
date: '2025-02-16T07:34:32+01:00'
draft: false
github_link: "https://github.com/AkanimohOD19A/simple_ml_pipeline/tree/main"
description: "deploying models and best practices using zenml x mlflow orchestration"
tags: ["MLOPs", "ZenML", "MlFlow", "data-science", "machine-learning", "streamlit", "best practices"]
author: "AfroLogicInsect"
image: blog-images/blogs-placeholder.png
---

Let's build this!

![Simple ML Pipeline](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kizs8rrbda3or1czu558.png)

To follow, please you must have at least built a Jupyter notebook or familiar with basic machine learning workflows - from data ingesting/wrangling to model deployment.

### ZenML

Now - ZenML is a beautiful tool I just discovered, thanks to @Ayush/@FreeCodeCamp for this [introductory tutorial](https://www.youtube.com/watch?v=dPmH3G9NQtY&t=7807s). 

If you visit its webpage, [ZenML](https://www.zenml.io/) is described as "The bridge between ML and Ops" and it is! I'd show you. However, let's review a few concepts to note on ZenML:

**T𝗵𝗲 𝗰𝗼𝗻𝗰𝗲𝗽𝘁 𝗼𝗳 𝗮 "step"**
[Step operators](https://docs.zenml.io/stack-components/step-operators) in ZenML are specialized components that allow individual pipeline steps to be executed in specific runtime environments optimized for certain workloads. This can be particularly useful when a step requires resources not available in the default environment provided by the orchestrator.

**T𝗵𝗲 𝗰𝗼𝗻𝗰𝗲𝗽𝘁 𝗼𝗳 𝗮 "𝘀𝘁𝗮𝗰𝗸"**
![ZenML Stack](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yp22sljqc2fe2btlwtof.png)

### A Simple Linear Regression
See this [Kaggle Notebook](https://www.kaggle.com/code/ybifoundation/simple-linear-regression).

It's a simple linear regression, that predicts (or best fits) __Salaries__ based off the __Years of Experience__ and stores the output of the trained model - what if we wanted more? We will always get new data sources, use a different prediction model, compare metrics, promote the best performing model to "production"? 

Essentially, how do we continually orchestrate machine learning exercises through its lifecycle? MLOPs - well, here's where ZenML as an orchestrator tool comes in.

We are going to reproduce this example as a set of steps in a single script orchestrator. 

Looking at the notebook above - we had three major steps:
1. Ingest the Data
2. Clean/Wrangle/Tidy the Data
3. Train the Data - Store the output.

We add an additional step
4. Promote the model

### Pre-installation and Virtual Environment
Create your virtual environment - the author recommends python 3.10
```
python3 venv -m <env_name>
# activate environment
<env_name>\Script\activate
```
Install dependencies
```
pip install "zenml[server]"
zenml integration install sklearn -y
zenml integration install mlflow -y
```
Create a stack
```
## Initiate 
zenml init
## Register Local artifact store
zenml artifact-store register <artifact_store_name> --flavor=local
## Register stack
zenml stack register <stack_name> -o default -a <artifact_store_name>
## Set stack
zenml stack set <stack_name>
## Review
zenml stack list
```

## A Simple ML Pipeline
`touch <simple_ml_pipeline>.py` 
<simple_ml_pipeline> is whatever name you want to call your script.

Enter the following into your script
```
## Dependencies
## Import Libraries
import os
import numpy as np
import pandas as pd
from typing import Dict, Tuple
from zenml import Model, pipeline, step
from zenml.client import Client
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error
from sklearn.linear_model import LinearRegression
from zenml.logger import get_logger

logger = get_logger(__name__)

# from sklearn.base import RegressorMixin
df_pth = 'https://github.com/ybifoundation/Dataset/raw/main/Salary%20Data.csv'
local_pth = "./datasets/SalaryData.csv"
local_model_register = "./model_performance.json"
local_model_pth = "./models"
```

### Step 1: Data Ingestion
```
@step(enable_cache=True)
def load_data() -> pd.DataFrame:
    if os.path.exists(local_pth):
        df = pd.read_csv(local_pth)
    else:
        os.makedirs("./datasets", exist_ok=True)
        df = pd.read_csv(df_pth)
        df.to_csv("./datasets/SalaryData.csv", index=False)
    return df
```
This indicates that the function _load_data_ is a pipeline step. Since the dataset is static the _enable_cache=True_ parameter means that the output of this step will be cached, so if the step is run again with the same inputs, the cached output will be used instead of re-executing the step.

_os_ commands check for file in our local paths and saves/loads the _salary_ data where it is found... 

Now that we have data, we train a simple linear regression model.

### Step 2: Train Model
```
@step
def train_model(data: pd.DataFrame) -> Tuple[Model, Dict[str, float]]:
    y = data['Salary']
    X = data[['Experience Years']]

    X_train, X_test, y_train, y_test = train_test_split(X, y, train_size=0.7, random_state=234)

    model = LinearRegression()
    model.fit(X_train, y_train)

    y_pred = model.predict(X_test)

    rmse = np.sqrt(mean_squared_error(y_test, y_pred))

    # Create a ZenML Model object
    zenml_model = Model(name="salary_prediction_model", model=model, metadata={"rmse": str(rmse)})

    return zenml_model, {"rmse": rmse}
```
In this _train_model_ is a pipeline step we wisely do not enable caching to accommodate randomness, like you expect we separate the target from response variables and split the data (from the _load data_ step) and train a Linear Regression, from which we retrieve the RMSE evaluation metric.

Now, Model Object - these are specialized components that ensure that models are versioned, tracked, and integrated into pipelines for reliable model management. See how we assign a model name and also store metadata to the object. Very mindful. 

We return the Model object and a dictionary of the values.

### Step 3: Promote Model
```
@step
def promote_model(
        model: Model,
        metrics: Dict[str, float],
        stage: str = "production"
) -> bool:
    rmse = metrics["rmse"]

    # Get the ZenML client
    client = Client()

    try:
        # Try to get the previous production model
        previous_production_model = client.get_model_version(name=model.name, version="production")
        previous_production_rmse = float(previous_production_model.run_metadata["rmse"].value)
    except:
        # If there's no production model, set a high RMSE
        previous_production_rmse = float('inf')

    if rmse < previous_production_rmse:
        # Promote the model
        model.set_stage(stage, force=True)
        logger.info(f"Model promoted to {stage}!")
        return True
    else:
        logger.info(
            f"Model not promoted. Current RMSE ({rmse}) is not better than production RMSE ({previous_production_rmse})")
        return False
```
This step conditional promotes model, first, it checks for an existing model - if it doesn't exist, sets an infinitely positive value. 

**RMSE is better when smaller**, hence the checks, if the current model’s performance (measured by RMSE) is better/lower than the existing production model. If it is, the model is promoted to the production stage. Otherwise, it remains unchanged. This ensures that only models with improved performance metrics are promoted to production!

That ends it for steps, now let's gather all of these steps into a pipeline, and you guessed it correctly, we get to use a *pipeline* decorator next.

### Pipeline
```
@pipeline
def simple_ml_pipeline():
    dataset = load_data()
    model, metrics = train_model(dataset)
    is_promoted = promote_model(model, metrics)

    return is_promoted
```
This simple_ml_pipeline function defines a ZenML pipeline that comprises of the initial steps that: 

1. Loads the dataset.
2. Trains a model using the dataset.
3. Evaluates and potentially promotes the model to production based on its performance.

This pipeline ensures a streamlined process for loading data, training a model, and promoting it if it meets the performance criteria.

Add
```
if __name__ == "__main__":
    run = simple_ml_pipeline()
```
to call the run method on the pipeline object.

Run `python <simple_ml_pipeline>.py` from your terminal to run your script, this should return the logs like so, through each step.

![logs](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wh64wocaho8yti8ahk1e.png)

Then you may run `zenml up` or `zenml up --blocking` for windows users, to view your pipeline on a UI. This should launch a local web server http://127.0.0.1:8237/ - the default login username is **default**

![Pipelie UI](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9g9vay0xgvftnhlsi55h.png)

That's it, Zen master. Visit [simple_ml_pipeline](https://github.com/AkanimohOD19A/simple_ml_pipeline/tree/main) for the full script.

Next, I'd show you how to induce other ML libraries (DecisionTrees and RandomForests), setup the steps elegantly and integrate with _mlflow_ for experiment tracking. Yes, it   integrates beautifully with _mlflow_.

Please note, the _promote model_ step was deliberately rigged to promote your model regardless, this is to prove efficacy at this level.
