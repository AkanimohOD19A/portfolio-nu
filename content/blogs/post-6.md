---
title: 'git init cohere_streamlit - pt2'
date: '2025-02-16T07:34:32+01:00'
draft: false
github_link: "https://github.com/AkanimohOD19A/initial-cohere"
description: "git init cohere_streamlit"
tags: ["data-science", "machine-learning", "cohere", "streamlit", "YOLOv6", "tutorial", "lifecycle"]
author: "AfroLogicInsect"
image: /images/blog-placeholder.png
---

### Setting Up Dependencies
After you have installed the required libraries, create a folder called .streamlit in your directory, within this folder create a secrets.toml file and put the following in it.
```
COHERE_API_KEY="<YOUR_COHERE_API_KEY>"
```
It is important to note that we would have to do the same when deploying the app on Streamlit within the Advanced Settings option.

#### app.py
Next, create an app.py file and import the required libraries, like so:
```
import cohere
import pandas as pd
import streamlit as st
```
having done this, we will need to retrieve the API Keys we stored earlier by calling the KEY from our Secrets repository, 
```
co = cohere.Client(st.secrets["COHERE_API_KEY"]) or st.secrets["COHERE_API_KEY"]
```

### Cache the data and define functions

The importance of caching is to stay performant even when loading data from the web, manipulating large datasets, or performing expensive computations, in our case this is done with the `@st.cache_data`

Next, we setup our functions:
1. `convert_df`: This function is to generate a dataframe from results/ideas.
2. `generate_idea`: Generates the idea
3. `generate_name`: Generates an idea's name

Let's start:
#### Convert Results to Dataframe
```
idea_bank = []
idea_name_bank = []

def convert_df(df1, df2):
    df = pd.DataFrame(list(zip(df1, df2)), columns=['IdeaName', 'Ideas'])
    return df.to_csv(index=False).encode('utf-8')
```
What this does is create an empty dataframe with two columns: _IdeaName_ and _Ideas_ from the inputs it is given and finally returns the dataframe in a csv format.

How we get these two values is by creating the two following functions with a basic structure that collect three parameters:
1. startup_idea or startup_idea_name: A text input describing an industry
2. creativity: 'the temperature parameter to control the randomness of the model output. The value can range between 0 and 5'
3. model_name: This will be defined to select from the four available cohere models {"command", "command-nightly", "command-light", "command-light-nightly"}

#### Generate the idea
```
def generate_idea(startup_industry, creativity, model_name):
    idea_prompt = f"""Generate a startup idea
Industry:{startup_industry}
Startup Idea: """

    # Call the Cohere Generate endpoint
    response = co.generate(
        model=model_name,
        prompt=idea_prompt,
        max_tokens=50,
        temperature=creativity,
        k=0,
        stop_sequences=["--"],
    )
    startup_idea = response.generations[0].text
    startup_idea = startup_idea.replace("\n\n--", "").replace("\n--", "").strip()
    return startup_idea

```
Here, we have the __idea_prompt__ that defines what the function is about, then a _response_ that generates the idea based on the availed parameters to make the api call. The __startup_idea__ retrieves the response, however untidy, then uses the __startup_idea_replace__ to remove the untidy trailings and that's it! Now what we need is a name for the idea.

#### Generate the idea's name
```
def generate_name(startup_idea, creativity, model_name):
    name_prompt = f"""Generate a startup name and name given the startup idea. 
Startup Idea:{startup_idea}
Startup Name:"""

    # Call the Cohere Generate endpoint
    response = co.generate(
        model=model_name,
        prompt=name_prompt,
        max_tokens=10,
        temperature=creativity,
        k=0,
        stop_sequences=["--"],
    )
    startup_name = response.generations[0].text
    startup_name = startup_name.replace("\n\n--", "").replace("\n--", "").strip()
    return startup_name
```
This function works like the previous function that retrieves the idea, collects three parameters, one of which is the *startup_idea* we had just generated and returns a name after making the api call to cohere.

### Format your web-application

At this stage, we are mostly done with the back-side of our architecture and basically building the front-facing side of this workflow. This is the stage for streamlit.

Give the page a title
```
st.title(' My startUp Application')
```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bnq84eqifmd2fd8wfitu.png)

Next, we create a Form that retrieves the parameters we would need in the functions. What you would see in the form is three columns that:
1. col1: Defines the the Number of ideas you would like to generate
2. col2: Defines the *temperature*, the numerical value for our _creativity_ value
3. col3: Defines the model name from an array of models
```
with form:
    industry_input = st.text_input("Industry", key="industry_input")

    col1, col2, col3 = st.columns(3)
    with col1:
        # User input - The number of ideas to generate
        num_input = st.number_input(
            "Number of ideas",
            value=3,
            step=1,
            key="num_input",
            min_value=1,
            max_value=10,
            help="Choose to generate between 1 to 10 ideas",
        )
    with col2:
        creativity_input = st.slider(
            "Creativity",
            value=0.5,
            key="creativity_input",
            min_value=0.1,
            max_value=0.9,
            help="Lower values generate more “predictable” output, higher values generate more “creative” output",
        )
    with col3:
        model_name = st.selectbox(
            "Please Select from the Available Models",
            ("command",
             "command-nightly",
             "command-light",
             "command-light-nightly")
        )

    # Pt:2 
    # Submit button to start generating ideas
    generate_button = form.form_submit_button("Generate Idea")
    if generate_button:
        if industry_input == "":
            st.error("Industry field cannot be blank")
        else:
            # my_bar = st.progress(0.5)
            my_bar = st.snow()
            st.subheader("Startup Ideas:")

        try:
            for i in range(num_input):
                st.markdown("""---""")
                idea = generate_idea(industry_input, creativity_input, model_name)
                idea_bank.append(idea)
                name = generate_name(idea, creativity_input, model_name)
                idea_name_bank.append(name)
                st.markdown("##### " + str(name))
                st.write(idea)
                # my_bar.progress((i + 1) / num_input)
                my_bar
        except Exception as e:
            print(e)
            # refresher(60)
            st.warning(f"An Error Occured, please retry in a minute and with an Idea value less than {num_input}")
```

In the *Pt:2*, we create a _generate_button_ that runs the function, appends the results to the set dataframes, then post the idea with the `st.write(idea)`. Note the Error Handler, the author added this to identify the limits within the free framework on cohere's api calls.

#### Finally, Download a Copy

This code will retrieve all of our ideas into a dataframe and create a downloaded copy.

```
if idea_bank:
    csv = convert_df(idea_name_bank, idea_bank)

    st.download_button(
        label="Download Idea as CSV",
        data=csv,
        file_name='Idea_from_Cohere.csv',
        mime='text/csv',
    )
```

To run your application, simply go to your terminal and 
```streamlit run app.py```

phew, we finally completed a streamlit application with LLMs, remember to create your `requirements.txt` before you deploy, a guide can also be found [here](https://txt.cohere.com/deploy-cohere-streamlit/), the author added functionality for caching and model selection, [github](https://github.com/AkanimohOD19A/initial-cohere/blob/master/app.py). Remember to deploy and share and to create something cool today.