---
title: "Let's build your first ML app in Google Cloud Run"
date: '2025-02-16T07:34:32+01:00'
draft: false
github_link: "https://github.com/AkanimohOD19A/automl_init"
description: "git init cohere_streamlit"
tags: ["data-science", "machine-learning", "GCP", "Vertex Ai", "streamlit", "tutorial", "lifecycle"]
author: "AfroLogicInsect"
image: /images/blog-placeholder.png
---


Unleash your inner data wizard! This blog is your roadmap to building a real-world machine learning app on Google Cloud Run, even if you're a coding newbie. Let's turn your ideas into intelligent reality. 

## Getting Started
Google Cloud Platform (GCP) provides a very befitting Machine Learning solution called [Vertex Ai](https://cloud.google.com/vertex-ai) that handles Google Cloud's unified platform for building, deploying, and managing machine learning (ML) models. Our goal is to build a simple Machine Learning application that optimizes all that GCP provides plus an implementation of continuous integration and continuous development (CI/CD). 

There are few pre-requisites on our short course, a [GCP account](https://console.cloud.google.com/) to get started her for free, Streamlit for python web-development, a GitHub repository - for Continuous Deployment and to ensure we stick the point, please follow through this [tutorial: Predicting Loan Risk with AutoML](https://www.cloudskillsboost.google/focuses/46225?locale=ar&parent=catalog) to build and deploying your first ML model. Ensure you use your account instead of the student account while you follow the tutorial.

## Model Development: Loan Risk Prediction
Vertex AI empowers you to build a data-driven loan risk prediction workflow, optimize lending decisions, and minimize financial risk. the workflows from:
- Data Mining: Retrieving the dataset, training an [AutoML](https://cloud.google.com/vertex-ai/docs/training/tabular-opt-obj) model, and getting predictions on whether a customer will repay a loan or not.

- Model evaluation and explanation: Vertex AI provides metrics such as confidence threshold, confusion matrix, and feature importance to assess the model performance and interpret the predictions.

- Endpoint deployment and testing: Vertex AI enables users to create an endpoint for the trained model.

![This flowchart summary condenses the key steps and emphasizes the iterative nature of the process](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/oifp49j6qjfvfwthncjo.png)

Once your model is fully trained and registered, you would get a mail. Navigate to your Endpoints;


![Online Prediction](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/syvg86bdh2l42sr8tp04.png)

Note the Endpoint ID and click on the Name - then click on 'Sample Request' at the top to see an example on how to make your API calls.
![Fetch Endpoint Details](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vadioj5yqvk3szo469t1.png)

This is a sample request in Python, Note the Project and Endpoint ID.
![Sample Request](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/q12pv1xcu226riygp3cl.png)




## Web-Page Development
Having successfully deployed your model endpoint, it is now time to show the world how it works, to achieve this we build a web-interface for anyone to use. Enter [Streamlit](https://streamlit.io/), think of it as a bridge between your data and the world, letting you present insights, perform analyses, and even collect user input with ease. It's perfect for data scientists, analysts, and anyone who wants to leverage the power of their data in a visually appealing and interactive way.

Create a directory, say **gcp-automl** and create an *app(dot)py* file, this will contain the web-page's python code, please simply paste the following code:

```
## Neccessary Imports
import os
import json
import random
import streamlit as st
import google.oauth2.credentials
from google.cloud import aiplatform
from google.oauth2 import service_account

## set environment variable to your credentials
os.environ("GOOGLE_APPLICATION_CREDENTIALS") = './credentials.json'

## set ids
project_id = 654321
endpoint_id = 123456

## Vertex AI Initialization

endpoint = aiplatform.Endpoint(
    endpoint_name=f"projects/{project_id}/locations/us-central1/endpoints/{endpoint_id}"
)
# ... proceed with endpoint operations ...

## Random Number Generator; to handle 'ClientID'
def generate_random_numbers(length):
    # Generate the random numbers
    random_numbers = ''.join(random.choice('0123456789') for _ in range(length))

    return random_numbers

## Build Streamlit App
def app():
    # Streamlit app title and expander for defining or updating the knowledge base
    st.title("TEST AutoML deployement")

    ## Inputs
    loan = st.slider("LOAN", 0, 100, key='1001')
    age = st.slider("AGE", 0, 100, key='1002')
    Income = st.slider("INCOME", 0, 100, key='1003')

    # Check if all fields have been filled
    if loan and age and Income:

        if st.button("Get Answer"):
            # Generate Client ID
            ClientID = generate_random_numbers(11)
            st.error(f'Your ID: {ClientID}')
            # Create a dictionary
            data = {
                "loan": str(loan),
                "age": str(age),
                "income": str(Income),
                "ClientID": str(ClientID)
            }
            # Now 'data' is a dictionary that holds your data

            st.json(data)
            ### Uncomment this and replace with your endpoint id from the Endpoints page
            #ENDPOINT_ID = endpoint_id
            PROJECT_ID = os.getenv("BUILD_SPECIFIC_GCLOUD_PROJECT")

            instance_dict = data
            response = endpoint.predict([instance_dict])

            # print('API response: ', response)
            st.success(f'API response: {response}')
    else:
        st.write("Please fill all the fields.")


# Run the Streamlit app
if __name__ == "__main__":
    app()
```

Here's a summary of what the code does:

1. Imports Necessary Libraries:

Imports modules for authentication, Vertex AI, random number generation, and Streamlit.
2. Sets Credentials and IDs:

Points the GOOGLE_APPLICATION_CREDENTIALS environment variable to the service account credentials file.
Specifies the project ID and endpoint ID for Vertex AI interaction.
3. Initializes Vertex AI Endpoint:

Creates an Endpoint object, representing a deployed model on Vertex AI.
4. Defines a Random Number Generator Function:

Generates random numbers of a specified length, for generating unique client IDs - a feature used in training.
5. Builds a Streamlit App:

Creates a web app using Streamlit with the following features:
Title: "TEST AutoML deployment"
User inputs: Sliders for "LOAN", "AGE", and "INCOME" values.
"Get Answer" button.
Error message if fields are not filled.
6. Handles User Input and Prediction:

When the "Get Answer" button is clicked:
Generates a random ClientID.
Creates a dictionary with the user input data and ClientID.
Displays the data in JSON format.
Uses the Vertex AI endpoint to make a prediction using the input data.
Displays the API response from the prediction in a success message.

### <caveat> This is not the best web-page ever built, the author admits.

Next, we create a Dockerfile that shows the application build, you can see it as an instruction to the host on how the app is to be built, in your directory create a 'Dockerfile' and paste the following;
```
FROM python:3.11
WORKDIR /app
COPY requirements.txt ./requirements.txt
RUN pip install -r requirements.txt
EXPOSE 8080
COPY . /app
ENTRYPOINT ["streamlit", "run", "app.py", "--server.port=8080", "--server.address=0.0.0.0"]
```

Remember to modify the code where necessary.

Now create a *requirements.txt* and paste the following packages in the text file:

```
streamlit>=1.18.0
asyncio==3.4.3
asyncpg==0.27.0
numpy
pandas
altair<5
protobuf<=3.20.1
cloud-sql-python-connector
google-cloud
google-cloud-secret-manager
google-cloud-aiplatform
shapely<2
kfp
google-cloud-pipeline-components
```

Navigate to the terminal and run the following:
Installed dependencies: `pip install -r requirements.txt`
Launch Streamlit: `streamlit run app.py`

Now you should get an FileNotFound error - don't worry - but you should see the application launched on your browser.

## Authenticating (Locally)
Remember this line of code:
```
## set environment variable to your credentials
oe.environ("GOOGLE_APPLICATION_CREDENTIALS") = './credentials.json'
```
We actually need the json file to create a connection to our GCP project, to achieve this: 
 - Visit the Google Cloud Console (https://console.cloud.google.com/).
 - Select the project you need credentials for.
 - Navigate to the "IAM & Admin" section.
 - Choose "Service Accounts."


![Service Accounts](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5i5je7sv5jp35k5pk7gj.png)


Create a Service Account (if needed):
 - If you don't have an existing service account, click "Create Service Account."
 - Provide a name and description for the account.
 - Grant it suitable roles and permissions based on your intended use.
 - Click "Done" to create the account.

Access the Credentials Tab:
 - Click on the newly created service account (or an existing one).
 - Navigate to the "Keys" tab.

Create a New Key:
 - Click "Add Key" and select "Create new key."
 - Choose "JSON" as the key type.
 - Click "Create" to generate the JSON file.

Download the JSON File:
The JSON file containing the service account credentials will automatically download to your local machine.

**Store it securely**, as it contains sensitive information.

Now before we continue, it is imperative that you are not careless with this file, it is quite sensitive. Create a '.gitignore' file in your directory and place the path there - e.g `credentials.json`, this will ensure that you don't push it to your GitHub repository.

Place the json file in your directory and refresh your Streamlit page, now you should see this:

![Streamlit page](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xg9k0qmsh4c1ufzxl85l.png)

## Authenticating (Cloud)
First, we need some changes - go back to your Streamlit *app.py* file and make the following changes:

### Old
```
## set environment variable to your credentials
os.environ("GOOGLE_APPLICATION_CREDENTIALS") = './credentials.json'

## set ids
project_id = 654321
endpoint_id = 123456

## Vertex AI Initialization

endpoint = aiplatform.Endpoint(
    endpoint_name=f"projects/{project_id}/locations/us-central1/endpoints/{endpoint_id}"
)
```

### New ✨
```
## Authenticates with Google Cloud Platform
credentials_json = os.environ.get("GOOGLE_APPLICATION_CREDENTIALS")
credentials_data = json.loads(credentials_json)
# private_key = credentials_data.pop("private_key")  # Extract private key
project_id = credentials_data.pop("project_id", None)  # Store project ID separately (optional)
credentials = google.oauth2.service_account.Credentials.from_service_account_info(
    credentials_data,
    scopes=None  # Optional: Specify OAuth 2.0 scopes if needed
)

## Sets project and endpoint IDs
project_id = 654321
endpoint_id = 123456

## Creates an endpoint object
aiplatform.init(project=project_id, credentials=credentials)  # Use optional project_id if stored
endpoint = aiplatform.Endpoint(endpoint_name=f"projects/{project_id}/locations/us-central1/endpoints/{endpoint_id}")
```
Breaking the **New** ✨ code chunk:
 - Prepares for interaction with Google Cloud Platform services.
 - Sets up access to a specific Vertex AI endpoint within a designated project.
 - Creates a reference to that endpoint for further operations using the Vertex AI library.

Essentially, the change we made, sets the stage for subsequent actions involving the Vertex AI endpoint, such as deploying a model, running predictions, or managing the endpoint itself.

Next, we have to do the same thing on the Host side, search for 'Secrets' and enable the API. Next we create a Secret that stores our credentials like so: 

 - Create a secret: On the Cloud, search 'Secret Manager' page, click "Create Secret".
 - Name and Access: Name the secret descriptively and browse through your drive to select the <credentials>.json file
 - Click on 'Create Secret': This creates an encryption of the document

![Creating Secret](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1oohvlugk7qmgnr9ktk3.png)

#### Don't forget the name of the Secret.


Then we enable our Service Account to the Secret Manager Secret Accessor" role, like so:
 - Stay on the Secret Manager page
 - Click the Secret: This opens the secret details page.
 - Go to the "Permissions" tab: This shows who has access to the secret and what permissions they have.
 - Click "Add principal": This opens a dialog box to add new users or groups.
 - Choose the service account: Select the service account you want to grant access to.
 - Select the "Secret Manager Secret Accessor" role: This grants the service account read and access permissions to the secret.
 - Click "Save": This saves the new permission granted to the service account.

![Secret Manager Secret Accessor](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hwnj44s9tcb90emvi3e7.png)

Phew, security..


## Continuous Integration (CI)/Continuous Deployment (CD) Pipeline

So, there's the tidy means of containerizing, that is, just deploying the Docker Image to Cloud Run to build and deploy, but it can be imagined that the elements of the ML prototype develops and changes, hence Continuous Integration (CI)/Continuous Deployment (CD).

Here, we graduate into fully using cloud technologies Git, Github, GCP Cloud Build and Cloud Run. 

Cloud Build and Cloud Run are a dynamic duo in Google Cloud:
Cloud Build automates building and testing your code. Cloud Run seamlessly deploys your code as containers, instantly serving it up to the world. Together, they streamline development, scale effortlessly, and save you time and money. 

Let's see how it works:
First, Git Bash into your directory - init, add, commit and push your files into a GitHub Repository.

Secondly, on 'Cloud Run' enable required Services - Activate both Cloud Build and Cloud Run APIs for your project:

 - Create a service - select **Continuous Development**

![Service Creation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/7ouk0thw33o4h452ia57.png)

 - Connect Source Code:
Link your source code repository (e.g., GitHub, GitLab, Cloud Source Repositories) to Cloud Build.
 - Build configuration:
Define branch on the Repository and select the Build Type, Save your selection.

![Build configuration](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/m0jlorv2nvmewg6lsb8h.png)

 - Security: select 'VARIABLES & SECRETS' and use environment variables and secrets for sensitive information referenced from *Secrets Manager*.
 - *Create* your service

![Deploy to cloud Build](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/r6k2splq1tyyxo4nxhql.png)

### Deploy!
Finally, your build start weaving into existence and any changes you make automatically updates ✨✨.

That's it! when the application is ready, you would see it listed in your Cloud Run profile, click on the 'Name' and retrieve your url from the top.

![Streamlit URL](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/j02i1wvdy1wqam27yxgb.png)

Cheers on completing this tutorial, if there are any issues, here are some troubleshooting tips: __Google documentation__, Check your logs, use a reference [repository](https://github.com/AkanimohOD19A/automl_init), Bard/ChatGPT, or just share using the comment section.

Finally, if you liked Two Thousand and Twenty-Three, you would love Two Thousand and Twenty-Four. Happy New Year ✨ in advance.
