---
title: 'Zenml for beautiful beautiful orchestration Pt3'
date: '2025-02-16T07:34:32+01:00'
draft: false
github_link: "https://github.com/AkanimohOD19A/simple_ml_pipeline/tree/main"
description: "Mlflow Aliases/Tags: deploying models and best practices using zenml x mlflow orchestration"
tags: ["MLOPs", "ZenML", "MlFlow", "data-science", "machine-learning", "streamlit", "best practices"]
author: "AfroLogicInsect"
image: blog-images/blogs-placeholder.png
---


# Deploying a Local MLOps Solution: The Final Chapter

*Part 3 of our MLOps Series*
![Orchestration Lineage] (https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gfnjb3ylhxfc3c91e0pl.png)

In the previous parts of this series, we explored the fundamentals of MLOps and set up our local environment. Now, we'll conclude by deploying our solution locally.

## Adapting Our Approach: From ZenML to MLflow

Model deployment with the MLflow flavor in ZenML proved challenging on Windows. Since our overarching goal is to create a simple solution that can run exclusively on your local machine, we opted for the best alternative: Model Registry using Tags/Aliases on MLflow.

Instead of deploying the MODEL object through ZenML, we now deploy entirely through MLflow. This change primarily impacts our promotion logic.

### MLflow: Model Registry with Tags/Aliases

![Promotion Logic](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qhgj9u9ftbvl2htj0v22.png)

The [MLflow Model Registry documentation](https://mlflow.org/docs/2.10.0/model-registry.html#register-a-model) is invaluable and will save you a ton of despair. 
We've adapted the approach using the *tags* [**staging**, **production**, **archived**] route, 
we use the MLflow's client API to manage these tag changes, deleting old tags and setting new ones to reflect the 
updated status of each model version.

This logic ensures that only better-performing models (based on RMSE) are promoted to production, while maintaining a history of all models in the "archived" state for future reference or rollback if needed.

1. We tag a model as "staging" immediately after training.
2. We pass it to the promotion stage, where we compare the Root Mean Squared Error (RMSE).
3. Based on performance, we update the tag to either "archived" or "production".

Here's how the crux of the `promotion_logic` as modified from our previous versions:

```python
# Determine the new tag based on RMSE comparison
if production_model:
    # Compare RMSEs
    if staging_rmse < production_rmse:  # Staging is better, promote it to production
        # Change tag "production" - "archived"
        mlflowClient.delete_model_version_tag(
            name=model_name, version=str(production_version), key="production"
        )
        mlflowClient.set_model_version_tag(
            name=model_name, version=str(production_version), key="archived", value=str(production_rmse)
        )
        # Change tag "staging" - "production"
        mlflowClient.delete_model_version_tag(
            name=model_name, version=str(staging_version), key=tag_key
        )
        mlflowClient.set_model_version_tag(
            name=model_name, version=str(staging_version), key="production", value=str(staging_rmse)
        )
        print(f"Promoted staging model to production. Archived previous production model.")
    
        return True
    else:  # Production is better, archive staging
        mlflowClient.delete_model_version_tag(
            name=model_name, version=str(staging_version), key=tag_key
        )
        mlflowClient.set_model_version_tag(
            name=model_name, version=str(staging_version), key="archived", value=str(staging_rmse)
        )
    
        return False
```
Here is how it works:

1. It first checks if there's an existing production model.
2. If a production model exists, it compares the RMSE of the staging model with the production model:
   - If the staging model performs better (lower RMSE):
        - The current production model is demoted to "archived" status.
        - The staging model is promoted to "production" status.
        - The function returns True to indicate a promotion occurred.

   - If the production model performs better:
        - The staging model is moved to "archived" status.
        - The production model remains unchanged.
        - The function returns False to indicate no promotion occurred.

![Model Register](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kc02y23zo7r77qe4byft.png)

## Using the Predictive Service

The culmination of this exercise is hosting a User Interface where 
the end-user can use the predictive service.
On the backside, we fetch the best-performing (**Production**) model from the model
register as a service on a streamlit. 
Streamlit [link] is an open-source Python framework designed for quickly developing interactive app prototypes like this.
Here's a simple script that demonstrates this:

```python
@st.cache_resource
def load_production_model():
    mlflow.set_tracking_uri("file:C:/Users/buasc/zenml_store/mlruns")
    mlflowClient = MlflowClient()
    model_name = "salary_prediction_regression-model"

    try:
        production_model, production_rmse, production_version = get_model_by_tag(
            tag="production",
            model_name=model_name
        )
        st.success(f"Loaded production model version {production_version} with RMSE: {production_rmse}")
        return production_model
    except Exception as e:
        st.warning("No production model found!")
        try:
            local_production_model_pth = "model_dir/5152.801173710256_randomForest_2024-09-18.pkl"
            with open(local_production_model_pth, 'rb') as file:
                production_model = pickle.load(file)
            st.success("Loaded from local dir!")
            return production_model
        except Exception as e:
            st.warning("No Local Production Model Found!")
            return None
```
This helper function attempts to load the model from MLflow. 

It first sets the tracking URI and tries to retrieve the model tagged 
as “production” from the specified MLflow tracking server. If successful, 
it returns the model along with its version and RMSE. 

If the production model is not found, it falls back to loading a local model 
from a specified file path. If both attempts fail, it returns None and displays 
appropriate warnings.

```python
# Load the model only once
production_model = load_production_model()

st.title("Model Prediction App")
# Create input fields for your features
work_experience = st.number_input(
    "Experience Years",
    min_value=1, value=5, step=1
)

if st.button("Predict"):
    if production_model is not None:
        # Create a DataFrame with the input data
        input_data = pd.DataFrame(
            [[float(work_experience)]], 
            columns=["Experience Years"]
        )

        # Make prediction
        prediction = production_model.predict(input_data)
        st.write(f"Prediction: {prediction[0]}")
    else:
        st.error("No model available for prediction.")
```

Now, we pass the function, users can enter their _years of experience_,
after the "Predict" button is clicked,
the app checks if the model is available, it makes a prediction and 
displays the predicted _salary_.

![User Interface | Streamlit](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8oqyn2xm4qv04fjos72x.png)

That's it! With this script, you can now interact with your deployed model through a user-friendly interface.

## Conclusion: Your Local MLOps Pipeline

This concludes our series on building a local MLOps solution. We've created a simple yet effective MLOps pipeline that:

1. Fetches and wrangles data
2. Trains a defined model
3. Stores the model's artifacts
4. Registers the model to a model registry
5. Iteratively promotes or discards models based on performance
6. Retrieves the best-performing model for end-use via a User Interface

This pipeline embodies the essence of ML Engineering, bringing together various components to create a robust, iterative process for managing machine learning models.

The full repository for this project is available [here](https://github.com/AkanimohOD19A/local_mlops_orchestration). Please don't hesitate to reach out if you have any questions or need further clarification. And if you are a video person, see the screencast [here](https://www.youtube.com/playlist?list=PLLE0uKjUE6VfBlpZj2dPEc_SZW4ZRdt5U).

Happy ML Engineering!
