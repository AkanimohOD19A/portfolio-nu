---
title: 'ChatGPT on Streamlit and monitoring query response, a simple implementation'
date: '2025-02-16T07:34:32+01:00'
draft: false
github_link: "https://github.com/AkanimohOD19A"
description: "real-time: Streamlit and monitoring query response"
tags: ["data-science", "machine-learning", "streamlit", "ChatGPT", "tutorial", "python", "EDA"]
author: "AfroLogicInsect"
image: /images/blog-placeholder.png
---

In this blog post, we will explore my approach to creating a prototype or MVP of sorts that demonstrates the use of the ChatGPT API to retrieve responses on queries fed, optimized for certain keywords and also present an exploration of the responses.

ChatGPT is the immensely popular model built by OpenAI, trained on massive amounts of data to provide human-like responses to prompts, recently the OpenAI team released APIs enabling us to be able to call this immense function from our application. 

To build such a website application we would use the Streamlit python library which is an open-source app framework and perhaps the best for anybody hoping to create fast prototypes, in this post we would see it at work.

This project is broken up into major parts:
**Create API Key from OpenAI**
The first part of this application is getting acquainted with the OpenAi ChatGPT API, first, create an account here then navigate to creating your API Keys that allow us to run queries using this powerful model.

Your API Key should look something like `Openai_SECRET_KEY = "sk-123oNeTwoTHRee"` and you should keep it private. Next, we start building our web application.

**Set up Streamlit environment and package installation**
Before we start, navigate to your work directory, and update your packages then we can begin with importing/installing the required libraries, let's see how we can achieve this with pipenv and conda environments.
Pipenv
Streamlit's officially-supported environment manager for macOS and Linux is Pipenv.
Navigate to the project folder containing your Pipenv environment:

```
cd myproject
```


Activate that environment, upgrade Streamlit, and verify you have the lastest version (at least 1.10.1):

```
pipenv shell
pip install --upgrade streamlit
pip install --upgrade openai
streamlit version
```


Or if you want to use an easily-reproducible environment, replace pip with pipenv every time you install or update a package:

```
pipenv update streamlit
pipenv update openai
pipenv run streamlit version
```


Conda
Activate the conda environment where Streamlit is installed:

```
conda activate $ENVIRONMENT_NAME
```


Be sure to replace _$ENVIRONMENT_NAME_ ☝️ with the name of your conda environment!
Update Streamlit within the active conda environment and verify you have the latest version:

```
conda update -c conda-forge streamlit -y
conda install -c anaconda openai
streamlit version
```


### Build Web-Application
Within your work directory, create a file called **app.py** and start importing the necessary packages, like so.

```
import streamlit
import openai
```


then for our secret key, create a dir called **.streamlit**, within this folder create a **secret.toml** file and place your key in this file, finally don't forget to create a git.ignore file that contains ***.toml** to ignore this file when pushing your code to GitHub.

Next, we start calling the API. From our app.py file enter the following code

```
def ChatGPT(user_query):
    '''
    This function uses the OpenAI API to generate a response to the given
    user_query using the ChatGPT model
    :param user_query:
    :return:
    '''
    # Use the OpenAI API to generate a response
    completion = openai.Completion.create(
        engine=model_engine,
        prompt=user_query,
        max_tokens=1024,
        n=1,
        temperature=0.5,
    )
    response = completion.choices[0].text
    return response
```

```
@st.cache_data
def api_call_on(query):
    '''
    This function gets the user input, pass it to ChatGPT function and
    displays the response
    '''

    response = ChatGPT(query)
    return response
```




```
# Set the model engine and your OpenAI API key
model_engine = "text-davinci-003"
openai.api_key = st.secrets["Openai_SECRET_KEY"]
```


Simply, the first function creates the interaction node to the API and api_call_on retrieves the query response st.cache_data, another Streamlit method for managing recurrent API calls.
At this stage, you may simply create a text input element and run your queries on them like so:

```
# Get user input
user_query = st.text_input("Enter query here, to exit enter :q",
                           "what is ChatGPT?")
# Retrieve response on input
response = api_call_on(prefix_query)
# Present response
st.success(f"{response}")
```


Simple and this exercise is complete as well as in debt to [Yasser Mustafa's](https://blog.devgenius.io/building-a-chatgpt-web-app-with-streamlit-and-openai-a-step-by-step-tutorial-1cd57a57290b) tutorial on this. However, this can be improved and for this particular project, we are adapting for specific keywords to support our user inputs as well as the mechanisms to manage and monitor these entries and responses. 

You may stop here or follow along to see how this is achieved.

### Build Web-Application II
**Creating and modifying Keywords**

Create a new directory within your work directory, **<keyword_dir>**, and add a **<action_words.>csv** file of a list of words, say action words.
Navigate to your app.py file and add the following to present a sidebar where we can interact with these words and well as merge them with our user input:

```
import pandas as pd
## Function to create list of words selected
def create_action(response_type):
    strList = [str(i) for i in response_type]
    myString = ", ".join(strList)
    prefix_keyword = myString + " and " + str(response_type)
    return prefix_keyword

## Read a list of adjectives and actions
actions = pd.read_csv('./<keyword_dir>/<action_words.>csv')

response_type = st.sidebar.selectbox("Please specify action", actions)
## Main Statement
if response_type != "":
    prefix_keyword = create_action(response_type)
    prefix_query = f'A {prefix_keyword} response to {user_query}'
    st.sidebar.markdown('##')  ##-> Empty Space Divider
    if st.sidebar.button("Run"):
        with st.spinner('Your query is running...'):
            response = api_call_on(prefix_query)
            st.success(f"{response}")
```

Note this "_Main Statement_", we will come back to this shortly.

### Modifying Keywords
To modify the keywords file we had created, we would use the **st_experimental_data_editor** feature in Streamlit, which allows us to open a data frame of our **<action_words.>csv** values, change, delete or even add new items and have it auto saved to our file, the change is real-time, and the following help us achieve this:

```
if st.sidebar.button("Edit Keywords"):
    ### Lifecycle of Editable Dataframe
    ## Read csv
    df = pd.read_csv('./<keyword_dir>/<action_words.>csv', names=['ACTIONS'], header=1)

    ## Editable Dataframe
    editable_df = st.sidebar.experimental_data_editor(df, axis=1), num_rows="dynamic")

    ## Update csv
    editable_df['ACTIONS'].to_csv('./<keyword_dir>/<action_words.>csv', index=False)
```


Notice, how we are keeping our handles in a sidebar, this is to keep this tidy and easily reconcilable.

**Monitoring Responses**
In this section, we are adding final touches to the app, the features are to help us review query time and the quick breakdown of the result retrieved, one thing you would appreciate is just how fast (i.e lightweight) and immensely powerful the model is, that's a talk for another time.
To achieve query time and word distribution we implement the following:

**Query Time**
To achieve this, wrap the "Main Statement" with a timer like so

```
import time
## start timer
start = time.time()

##=> "Main Statement" comes here
...
response = api_call_on(user_query)
st.success(f"{response}")

## End Timer
end = time.time()
## Query Time
query_time = end - start
```

**Word Count**
We create a function that collects the words retrieved from our query and strips them, then accounts for frequency. We may even present a data frame showing each word and how many times it occurs.

```
## Word count
def word_count(string):
    strip_words = (len(string.strip().split(" ")))
    return strip_words
```

```
col1, col2 = st.columns(2)

with col1:
    st.warning("Query Data")
    st.write('Keywords: ', prefix_keyword)

with col2:
    st.warning("Basic Text Analysis")
    st.write("Query Time :   s", round(query_time / 1000, 2), "-", round(query_time, 2), "ms")
    st.write("Word Count: ", word_count(response) - 1)
```


That brings us to the end of this Implementation you may share what you created here in the comments.

Please check out the web application and GitHub repository.


