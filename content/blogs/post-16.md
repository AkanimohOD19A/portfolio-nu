---
title: 'Zenml for beautiful beautiful orchestration Pt2'
date: '2025-02-16T07:34:32+01:00'
draft: false
github_link: "https://github.com/AkanimohOD19A/simple_ml_pipeline/tree/main"
description: "deploying models and best practices using zenml x mlflow orchestration"
tags: ["MLOPs", "ZenML", "MlFlow", "data-science", "machine-learning", "streamlit", "best practices"]
author: "AfroLogicInsect"
image: blog-images/blogs-placeholder.png
---

Here's where we take the next step of organizing our single script by transforming it into specific scripts per _step_ - this makes things tidier. More importantly, we'd get to learn more about decorators and _Model_ objects, for example we get to use **Model** Objects and how to store model artifacts.

Since this is sequel to this [post](https://dev.to/afrologicinsect/zenml-for-beautiful-beautiful-orchestration-46db), we can just continue with the created stack - in the author's example it is called *new_stack*. However, before we go ahead, let's update this stack by registering an _experiment tracker_, which would be used later on to configure [_MLflow_](https://www.bing.com/ck/a?!&&p=8865c0a4693eaf53JmltdHM9MTcyNjg3NjgwMCZpZ3VpZD0zODA4ZWQwNS00YmRlLTYwYjQtMjNhNi1mZWZiNGEwMzYxOWMmaW5zaWQ9NTQ4NQ&ptn=3&ver=2&hsh=3&fclid=3808ed05-4bde-60b4-23a6-fefb4a03619c&psq=mlflow&u=a1aHR0cHM6Ly9tbGZsb3cub3JnL2RvY3MvbGF0ZXN0L3RyYWNraW5nLmh0bWw&ntb=1).

##### Experiment Tracking: The concept of *Client* and the *MLflow* setup

The primary **client** is the ZenML Python Client, it allows you to programmatically manage various resources within, which is quintessential for tracking your ML experiments, in this case, comparing the _rmse_ across varied model outputs.

So let's intitiate one for __MLflow__.

On your terminal update your stack by running the following:
```
# Register the MLflow experiment tracker
zenml experiment-tracker register <mlflow_experiment_tracker_name> --flavor=mlflow

# Register and update a stack with the new experiment tracker
zenml stack update new_stack -a <artifact_store_name> -o default -d default -e <mlflow_experiment_tracker_name>
```
If you run `zenml stack list` it should look something like so:

![stack list](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jpeul6fkhd6escyybkr2.png)

Now that you have an experiment tracker set your stack, you can call it in your python environment like so - to store and retrieve records.

```
from zenml.client import Client

## Initiate Experiment Tracker
experiment_tracker = Client().active_stack.experiment_tracker
```

This loads your active tracker, which we will store values and subsequently pass to the following _step_ decorator. Let's, however, start by setting up the directories.

### Initiate Directories
The first thing we do is to place our scripts in directories. From the terminal we need to create two new directories - `mkdir pipelines steps` - to accommodate their respective orchestration scripts.

#### *steps* directory
Let's deal for the _steps_:
```
## Navigate into the steps folder
cd steps
### create the dedicated step scripts
touch __init__.py ingest_data.py model_deployer.py promote_model.py train_model.py
```
The next step is intuitive, for each _.py_ script simply place the necessary **step**, so the __ingest_data__ script will look like so:

```
# ingest_data.py
## Import Libraries
import os
import pandas as pd
from zenml import step
from typing import Annotated

## Data Dependencies
df_pth = 'https://github.com/ybifoundation/Dataset/raw/main/Salary%20Data.csv'
local_pth = "./datasets/SalaryData.csv"


## Ingestion Step
@step(enable_cache=True)
def load_data() -> Annotated[pd.DataFrame, "Salary_data"]:
    if os.path.exists(local_pth):
        df = pd.read_csv(local_pth)
    else:
        os.makedirs("./datasets", exist_ok=True)
        df = pd.read_csv(df_pth)
        df.to_csv("./datasets/SalaryData.csv", index=False)

    return df
```

### Model Training
This step induces the training of a regression model, initiallywe used just the Linear Regression module - now, we would expand to three Regressors (add; Decision Tree and Random Forest)**set link, additionally we would further explore the ZenML Model Object to see how we log metadata.

```
# train_model.py
## Load dependencies
# import ..
from zenml.client import Client

## Initiate Experiment Tracker
experiment_tracker = Client().active_stack.experiment_tracker

# Get today's date
today = date.today()
## Model Directory
model_dir = 'model_dir'


@step(enable_cache=False, experiment_tracker=experiment_tracker.name)
def train_model(
    data: pd.DataFrame,
    model_name: Union[str, None]
) -> Annotated[Model, "trained_model"]:
    y = data['Salary']
    X = data[['Experience Years']]

    X_train, X_test, y_train, y_test = train_test_split(X, y,
                                                        train_size=0.7,
                                                        random_state=234)

    model = None
    if model_name == "linearRegression" or model_name is None:
        model = LinearRegression()
    elif model_name == "decisionTree":
        model = DecisionTreeRegressor()
    elif model_name == "randomForest":
        model = RandomForestRegressor()

    # Log the model type
    mlflow.log_param("model_type", model_name)

    model.fit(X_train, y_train)
    y_pred = model.predict(X_test)
    rmse = np.sqrt(mean_squared_error(y_test, y_pred))

    # Log Metrics
    mlflow.log_metric("rmse", rmse)

    # Log the sklearn model
    mlflow.sklearn.log_model(model, "salary_prediction_model")

    # Create a ZenML Model object
    zenml_model = Model(
        name="salary_prediction_model",
        model=model,
        metadata={"rmse": str(rmse)}
    )

    # Log metadata directly to the zenml_model object
    zenml_model.log_metadata({"rmse": str(rmse)})
    zenml_model.set_stage("staging", force=True)
    # Export Model to Local Directory
    model_dir_name = f"{rmse}_{model_name}_{today}.pkl"

    if not os.path.exists(model_dir):
        os.makedirs(model_dir, exist_ok=True)
    model_path = os.path.join(model_dir, model_dir_name)

    # Dump model
    joblib.dump(model, model_path)

    # Log the model path as an artifact
    mlflow.log_artifact(model_path)

    logging.info(f"Successfully Trained {model_name} \n"
                 f"and stored trained model to Model Register: {model_path}")

    return zenml_model 
```

The _step_ decorator now disables caching AND sets up experiment tracking. Other amendments include the model selection that chooses between **LinearRegression**, **DecisionTreeRegressor**, or **RandomForestRegressor** based on the `model_name` parameter. Furthermore we save a pickled version of the models in our local drive.

The experiment_tracker client becomes useful where we start to integrate Mlflow, which logs the model type, RMSE metric, and the trained sklearn model artifacts.

We also create a Model Object which we will set to "staging", it is one the object that we will return in our next step.

### Model Promotion
This step, _promote_model_, is a crucial component in our pipeline, designed to manage the promotion of models to production. It uses of ZenML's Model class and client.

The function's core purpose is to compare the performance of a staging model against the current production model, using RMSE (Root Mean Square Error) as the key metric. It begins by retrieving the RMSE of the staging model and then attempts to fetch the same for the existing production model, gracefully handling scenarios where no production model yet exists.

```
# promote_model.py
## Load dependencies
from zenml import step, Model, log_model_metadata
from typing import Dict, Annotated
from zenml.logger import get_logger
from zenml.client import Client

logger = get_logger(__name__) 

@step
def promote_model(
        model: Model,
        # metrics: Dict[str, float],
        stage: str = "production"
) -> Annotated[bool, "best_model"]:
    # Get the ZenML client
    client = Client()

    # Staging RMSE
    current_staging = client.get_model_version(
        model_name_or_id=model.name,
        model_version_name_or_number_or_id="staging"
    )
    staging_rmse = float(current_staging.run_metadata["rmse"].value)
    logger.critical(f"STAGING RMSE: {staging_rmse}")

    # Production RMSE
    #> Initiate with None & Inf
    current_production = None
    production_rmse = float('inf')

    try:
        # Try to get the production model
        current_production = client.get_model_version(
            model_name_or_id=model.name,
            model_version_name_or_number_or_id="production"
        )
        if current_production:
            production_rmse = float(current_production.run_metadata["rmse"].value)
            logger.info(f"Current production model version: {current_production.id}, RMSE: {production_rmse}")
    except Exception as e:
        logger.warning(f"Error fetching previous production model: {str(e)}")

    # Condition for promotion
    if staging_rmse < production_rmse:
        try:
            # Archive the current production model if it exists
            if current_production:
                try:
                    production_model = Model(
                        name=model.name,
                        version="production"
                    )
                    # This will set this version to production
                    production_model.set_stage(stage="archived", force=True)
                    logger.info(f"Previous production model (id {current_production.id}) archived. \n"
                                f"Previous RMSE: {production_rmse}, New RMSE: {staging_rmse}")
                except Exception as archive_error:
                    logger.warning(f"Failed to archive previous model: {str(archive_error)}")

            log_model_metadata(
                model_name=model.name,
                model_version=model.version,
                metadata={"rmse": staging_rmse},
            )
            # Promote the new model
            model.set_stage(stage, force=True)
            logger.info(f"New model (version {model.version}) promoted to {stage}!")
            return True
        except Exception as e:
            logger.error(f"Error during model promotion: {str(e)}")
            return False
    else:
        logger.info(
            f"Model not promoted. STAGING RMSE ({staging_rmse}) is not better than "
            f"PRODUCTION RMSE ({production_rmse})")
        return False
```

The scripts initialize a ZenML client to interact with the model registry and sets a positive and infinite value where there is no previously registered model.

The heart of the function lies in its promotion logic. 

![promotion logic](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/srxvwbkfyniucwciabdq.png)

If the staging model demonstrates superior performance (lower RMSE), a series of actions are triggered. This `production_model.set_stage(stage="archived", force=True)` archives the current production model (if it exists), logging new metadata for the staging model, and officially promoting it to production status. 

Throughout this process, comprehensive logging provides visibility into each step, capturing both successful operations and any encountered errors.

Remember maintaining system integrity is paramount, hence, error handling is woven throughout the function, ensuring robustness in various scenarios, such as failures in fetching model information or during the promotion process itself. 

This has to take a careful approach that ensures that only models with proven improvements are promoted, maintains a clear record of model lineage, and provides detailed logging for transparency and debugging.

We are done with the *steps* - time to configure our pipeline.

### Pipelines
Here, we don't do much differently. Simply, `cd ./pipelines`, to navigate to the pipeline initially created, then `touch __init__.py <pipeline_name.py>`.

Paste the following in <pipeline_name.py>
```
from zenml import pipeline
from steps.ingest_data import load_data
from steps.train_model import train_model
from steps.promote_model import promote_model

@pipeline
def simple_ml_pipeline(modelname: str):
    dataset = load_data()
    model = train_model(dataset, modelname)
    is_promoted = promote_model(model)

    return is_promoted
```
The `@pipeline` decorator indicates that it represents a complete ML workflow. This pipeline encapsulates the three main stages of a our ML process: data ingestion, model training, and model promotion.

The pipeline takes a single parameter, `modelname`, which determines the type of model to be trained. It then executes three key steps in sequence.

See how the modular structure defines each step as a separate function for code organization and reusability. Very cutesy.

### Runing the pipeline
All that's left now is a script that runs this pipeline. To achieve this `cd ..` into your parent directory and `touch run_pipeline.py` (You can call it anything you want) - in this script paste the following:
```
from pipelines.training_pipeline import simple_ml_pipeline
from zenml.client import Client
import logging
import click
from zenml.integrations.mlflow.mlflow_utils import get_tracking_uri


## tracking uri
track_uri = get_tracking_uri()

## Click Options
LR = "linearRegression"
DT = "decisionTree"
RF = "randomForest"


@click.command()
@click.option(
    "--modelname",
    "-m",
    type=click.Choice([LR, DT, RF]),
    default=LR,
    help="Optionally you can choose any of the models to train "
         "By default the Linear Regression Model will be used ",
)
def execute_pipe(modelname: str):
    simple_ml_pipeline(modelname)

    print(
        "Now run \n "
        f"    mlflow ui --backend-store-uri '{track_uri}'\n"
        "To inspect your experiment runs within the mlflow UI.\n"
        "You can find your runs tracked within the `mlflow_example_pipeline`"
        "experiment. Here you'll also be able to compare the two runs.)"
    )


if __name__ == "__main__":
    execute_pipe()
```
This code snippet defines a command-line interface (CLI) for executing the `simple_ml_pipeline` using the Click library, a user-friendly command-line tool, providing options for model selection.
   
Three constant variables (LR, DT, RF) are defined, representing different model types: Linear Regression, Decision Tree, and Random Forest. These serve as shorthand identifiers for the available model choices.

The `execute_pipe` function is decorated with Click commands and options. It takes a `modelname` parameter, which the user can specify when running the script. The `--modelname` or `-m` option allows users to choose between the three predefined model types, with Linear Regression set as the default.

e.g `python run_pipeline.py -m decisionTree`

![run_pipeline logs](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rnv0zelscdnq5h3sv4nc.png)

When executed, the function calls the `simple_ml_pipeline` with the chosen model name, Decision Tree Regressor, (which performs worse than the previous model - and is not promoted to "production"), initiating the ML workflow defined earlier.

The `if __name__ == "__main__":` block ensures that the `execute_pipe` function is called only when the script is run directly, not when imported as a module. After pipeline execution, it logs the tracking URI of the active experiment tracker, which is useful for monitoring and accessing experiment results.

Phew! It gets easier.

#### User Interface
##### ZenML
To review your work with the dedicated server, like before, run `zenml up` or `zenml up --blocking` (windows)

##### Mlflow
To review your experiments and artifacts follow the output of the print statement, in this case `mlflow ui --backend-store-uri 'file:C:\Users\buasc\zenml_store\mlruns'`.

You should launch your mlflow web server locally and see something like so.

![Web Server](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vsmcvi5jjzm0kwvy0549.jpg)

That's it, for now. The next tutorial we would see how to add another _step_ that deploys the promoted model, such that we can it as a predictive service from streamlit*.

If you found any of these difficult, please reach out via the comment section. The full scripts can be found [here](https://github.com/AkanimohOD19A/simple_ml_pipeline-Pt-2/tree/main/wk_exp_02).