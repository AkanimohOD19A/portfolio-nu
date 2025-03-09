---
title: 'Wk 3 Experiment Tracking: MLOPs with DataTalks'
date: '2025-02-16T07:34:32+01:00'
draft: false
github_link: "https://github.com/AkanimohOD19A/MLOps_24"
description: "git init cohere_streamlit"
tags: ["MLOPs", "data-science", "machine-learning", "streamlit"]
author: "AfroLogicInsect"
image: /images/blog-placeholder.png
---


In this week, the course discusses how to carry out ML experiments, using the __MLflow__ experiment tracking tool. This setup allows you to track and log your machine learning experiments and results.

For this week's assignment, we have been provided with 4 scripts for us to modify to complete the assignment. 

Here's what the questions are:

The only setup required now is to install the __mlflow__ library into the our __MLOPS_env__ virtual environment as well as download the 4 scripts.

**Q1: Install MLflow**
Like before from your parent folder (MLOPS), create a __wk2__ sub-directory and navigate into it, download the scripts from [here](https://github.com/DataTalksClub/mlops-zoomcamp/blob/main/cohorts/2024/02-experiment-tracking/homework) and then run the following:
```
mkdir wk2
cd wk2
```
Then activate your virtual environment and install like so:
```
pip install -mlflow
mlflow --version
```
=> mlflow, version 2.13.0

Or just create a __jupyter notebook__ to have a compact file for your answers, simply prefix these bash/terminal commands with a `!`.

**Q2. Download and preprocess the data**
To store the datasets, we will create a new sub-directory - 

```
## Fetch Data
! mkdir datasets 
! curl -o ./datasets/green_tripdata_2023-01.parquet https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2023-01.parquet
! curl -o ./datasets/green_tripdata_2023-02.parquet https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2023-02.parquet
! curl -o ./datasets/green_tripdata_2023-03.parquet https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2023-03.parquet
```
To **preprocess** the datasets, we will use one of the pre-defined scripts - preprocess.py
`! python preprocess_data.py --raw_data_path datasets/ --dest_path ./output` - what does this do?

This command runs the `preprocess_data.py` Python script with two command-line arguments:

- `--raw_data_path`: Specifies the path to the raw data, which is set to `datasets/`.
- `--dest_path`: Specifies the destination path for the output of the script, which is set to `./output` which it creates to save the processed data and automatically upon run.

Now if you run if you run `! ls`, we should have:
```
wk2
 |
 - datasets/
 - output/
 - homework.ipynb
 - hpo.py preprocess_data.py register_model.py train.py
```

Now, `!ls output` and you have 4 pickle files - 
```
dv.pkl
test.pkl
train.pkl
val.pkl
```

NB: Notice that the download files follow a `{dataset}_tripdata_YYYY-MM.parquet` naming convention.

**Q3. Train a model with autolog**
The task is to modify the script to enable autologging with MLflow, execute the script and then launch the MLflow UI to check that the experiment run was properly tracked.

For brevity, the author would just show where modifications were made on the original train.py file.

3.1 Set Experiment Tracking
First, we setup our mlflow server to allows us track experiments, store results, and manage the machine learning models.
`
mlflow server --backend-store-uri sqlite:///mlflow.db  --default-artifact-root ./artifacts  --host 0.0.0.0
`

This command starts an MLflow tracking server with the following configurations:

- `--backend-store-uri sqlite:///mlflow.db`: Sets the backend store to a SQLite database located at `mlflow.db`.
- `--default-artifact-root ./artifacts`: Sets the default location for storing artifacts (like models and plots) to the `./artifacts` directory.
- `--host 0.0.0.0`: Binds the server to all public IPs (`0.0.0.0`), making it accessible from other machines.

Now, we make modifications to the training script.
```
# Set Tracking URI
mlflow.set_tracking_uri("http://127.0.0.1:5000")
# Set the experiment name
mlflow.set_experiment("sklearn-init")
```
This code is used to set up MLflow for tracking experiments:

1. `mlflow.set_tracking_uri("http://127.0.0.1:5000")`: This sets the tracking URI to the local server running at `http://127.0.0.1:5000`, which is where MLflow is listening for incoming tracking data.
2. `mlflow.set_experiment("sklearn-init")`: This sets the name of the experiment to "sklearn-init" in MLflow, under which all runs will be logged.


```
def run_train(data_path: str):
    # Enable autolog
    mlflow.sklearn.autolog()
    with mlflow.start_run():
        <Original training Commands>
```
The `run_train` function is designed to train a machine learning model and log the training process with MLflow:

1. `mlflow.sklearn.autolog()`: Automatically logs MLflow metrics, parameters, and models when training with **scikit-learn**.
2. `with mlflow.start_run()`: Starts a new MLflow run to track the training process within the block.

The placeholder `<Original training Commands>` is where the actual machine learning training commands would be placed. When this function is called with a data path, it will train the model and log all relevant information to MLflow.

Run `mlflow server` from the terminal to launch MLflow you should see this:
`INFO:waitress:Serving on http://127.0.0.1:5000`

Now run `python train.py` you should see something like:
```
2024/06/17 18:38:32 INFO mlflow.tracking.fluent: Experiment with name 'sklearn-init' does not exist. Creating a new experiment.
2024/06/17 18:38:32 WARNING mlflow.utils.autologging_utils: You are using an unsupported version of sklearn. If you encounter errors during autologging, try upgrading / downgrading sklearn to a supported version, or try upgrading MLflow.
2024/06/17 18:38:34 WARNING mlflow.sklearn: Failed to log training dataset information to MLflow Tracking. Reason: 'numpy.ndarray' object has no attribute 'toarray'
2024/06/17 18:39:03 WARNING mlflow.utils.autologging_utils: MLflow autologging encountered a warning: "c:\Users\buasc\OneDrive\Desktop\MLOps_24\MLOps_24\Lib\site-packages\_distutils_hack\__init__.py:26: UserWarning: Setuptools is replacing distutils."
c:\Users\buasc\OneDrive\Desktop\MLOps_24\MLOps_24\Lib\site-packages\sklearn\metrics\_regression.py:492: FutureWarning: 'squared' is deprecated in version 1.4 and will be removed in 1.6. To calculate the root mean squared error, use the function'root_mean_squared_error'.
  warnings.warn(
```

![MLflow server](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jibml1dflqlqnrfltr4a.png)

![min_samples_split](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/o6tm405ey98s8w85hfjr.png)

**Q5. Tune model hyperparameters**
Our task is to modify the script _hpo.py_ and make sure that the **validation RMSE** is logged to the tracking server for each run of the hyperparameter optimization (we will need to add a few lines of code to the _objective_ function) and run the script without passing any parameters. "Note: Don't use autologging for this exercise."

Modifications:
```
def run_optimization(data_path: str, num_trials: int):
    # Enable autolog
    # mlflow.sklearn.autolog()
    with mlflow.start_run():
        <nested commands to retrieve pickle files>

        def objective(params):
            with mlflow.start_run(nested=True):
                <nested commands to generate rmse>

                mlflow.log_metric("rmse", rmse)
                ....
```
Run `python hpo.py` - this one will take a while to run, because we introduce hyper-parameter tuning. You should have something like this.

![hyper-params tuning](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jhthdx517yj8wxunvezc.png)

Now refresh your MLflow ui, you should have:

![random-forest-hyperopt](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ifnra6g17yontan1n2sf.png)

Collapse the Run Name, to explore the various results.
![Collapsed Run](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/40czv0401tpayqhspxx7.png)

![rambunctious-hog-867 rmse: 5.335419588556921](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hkty5xnfpnmv8iww0ptq.png)

=> rambunctious-hog-867 (rmse)
=> rmse: 5.335419588556921

*Q6. Promote the best model to the model registry*
Our task is to update the script _register_model.py_ so that it selects the model with the lowest RMSE on the test set and registers it to the model registry.

Modifications:
```
with mlflow.start_run():
        new_params = {}
        for param in RF_PARAMS:
            new_params[param] = int(params[param])
            ## Coarse params to integers
            params[param] = int(params[param])
```
```
# Select the model with the lowest test RMSE
    experiment = client.get_experiment_by_name(EXPERIMENT_NAME)
    best_run = client.search_runs(
        experiment_ids=experiment.experiment_id,
        run_view_type=ViewType.ACTIVE_ONLY,
        max_results=1,
        order_by=["metrics.test_rmse ASC"]
    )[0]
```

1. `client.get_experiment_by_name(EXPERIMENT_NAME)`: Retrieves an experiment object by its name.
2. `client.search_runs(...)`: Searches for runs from the retrieved experiment, filtering for active runs only, limiting the results to one, and ordering them by the "test_rmse" metric in ascending order.

The variable `best_run` will hold the run with the lowest RMSE on the test set, indicating it's potentially the best-performing model.

```
# Register the best model
    model_uri = f"runs:/{best_run.info.run_id}/model"
    mlflow.register_model(model_uri, "best_random_forest_model")
```

1. `model_uri = f"runs:/{best_run.info.run_id}/model"`: Constructs the URI for the model from the best run's ID.
2. `mlflow.register_model(model_uri, "best_random_forest_model")`: Registers the model with the given URI under the name "best_random_forest_model" in MLflow's model registry.
Allowing us to version and manage our models systematically.

Run this script as we have done previously and you should have around **5.567** as your best performing  RMSE.

That's it!
Visit [wk2_submission](https://github.com/AkanimohOD19A/MLOps_24/tree/main/wk2) to review the codes and Cheers!
Comment below if there are any issues.

I'd be skipping the solutions on Wk3 as it is extensively covered in this [YouTube Tutorial](https://www.loom.com/share/802c8c0b843a4d3bbd9dbea240c3593a)



