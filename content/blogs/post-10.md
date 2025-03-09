---
title: 'From Data to Insights: Building a No-Code Web App for Exploratory Data Analysis'
date: '2025-02-16T07:34:32+01:00'
draft: false
github_link: "https://auto-eda.streamlit.app/"
description: "git init cohere_streamlit"
tags: ["data-science", "machine-learning", "EDA", "Retrieval Augmented Generation", "streamlit", "cohere", "tutorial", "lifecycle"]
author: "AfroLogicInsect"
image: /images/blog-placeholder.png
---

Hi everyone, welcome! Today we are going to create this awesome [web-app](https://auto-eda.streamlit.app/): that simply takes your data file and provides first-level analysis or what is commonly known as exploratory data analysis (EDA).

![EDA Web App](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qlxnqbe9fdb41yl5r7qw.png)

This web-app is a tool for EDA, which is a process of understanding the data, finding patterns, outliers, correlations, and visualizing the results. EDA is an essential step before building any machine learning model or applying any statistical technique.

The web-app is built using Streamlit, a Python framework that lets you create interactive web-apps with minimal code. Streamlit is very easy to use and has a lot of built-in widgets and components that make your app look professional and engaging.


## 1. Create/Version a Directory
To start developing your app, you will need to create a new directory to store your code and data. Additionally, you will need to set up a clean Python virtual environment. Here are the steps to create a virtual environment:

First, make sure you have Python installed on your machine. You can check if Python is installed by opening a command prompt or terminal window and typing python --version. If you see a version number displayed, you’re good to go. If not, head over to the official Python [website](https://www.python.org/downloads/) and download the latest version.

Navigate to the directory where you want to create your virtual environment using the cd command in your command prompt/terminal window — e.g., 

`cd Documents/Projects/my_dir`

Execute the following command in your command prompt/terminal window, replacing _my_env_ with whatever name you want for your virtual environment (no spaces):

`python -m venv my_env`

This will create a new folder called _my_env_ in the current directory that contains all necessary files for our venv.

To activate our newly created virtual environment, we need to run a script depending on which operating system we’re using:

_On Windows_: In your command prompt window, type `.my_env\Scripts\activate.bat` followed by Enter.
_On macOS/Linux_: In Terminal app or similar program, type source `my_env/bin/activate`, then press Enter.

And voila! Your virtual environment is now activated! You should see _my_env_ added at the beginning of every line in your command prompt/terminal window from now on indicating that you’re working within this specific venv.

#### 1.1 Install Dependencies

Next, we install the necessary libraries, such as pandas, numpy, seaborn, matplotlib, and streamlit. To install multiple packages in your virtual environment, you can create a _requirements.txt_ file that lists all the packages you want to install, one per line. You can also specify a version number for each package by adding ==<version> after the package name.

```
##requirements.txt
streamlit>=1.27.0 2
pandas>=1.4.0 1
numpy>=1.22.0 1
seaborn>=0.11.2 1
matplotlib>=3.5.0 2
pyarrow>=4.0.0
plotly>=5.13.1
```

You can then use the following command to install all the packages at once:

`pip install -r requirements.txt`

This command will read the requirements.txt file and install all the packages listed in it. 

#### 1.2 Other dependencies
Create a file called _app.py_ to hold our python codes and create a new directory that will hold our [sample data](https://github.com/AkanimohOD19A/AutoEDA/blob/basic_dist/dta/sample_data.csv). At this stage your directory should look like this.

```
my_dir/
|
├── venv/
|
├── dta/
       ├── sample_data.csv
├── requirements.txt
|
└── app.py
```

## 2. Building the Web-Page
The Streamlit page is basically divided into two parts: the sidebar and main page. The side-bar handles data loading, data descriptions and other page handlers, while the main page holds the visualizations and data page, don't worry we will build this together.

#### 2.1 Side Bar
Create a sidebar with file upload and page handles.
Now in your app.py. Import the dependencies, define the handles and create your functions, like so:

```
### Libraries come here
import streamlit as st
import numpy as np
import pandas as pd
import pyarrow
import plotly.express as px

### Placeholder Data
df = pd.read_csv("./dta/sample_data.csv")
```
This will load our packages and also read in the sample file we had created.

```
### Functions come here
numerics = ['int16', 'int32', 'int64', 'float16', 'float32', 'float64']
numeric_df = df.select_dtypes(include=numerics)
numeric_columns = numeric_df.columns.tolist()
category_df = df.drop(columns=numeric_columns)
category_columns = category_df.columns.tolist()

### Basic Analytics
shape = df.shape
null_values = sum(df.isnull().sum())
```
This code block selects all columns with numeric data types and stores them in a new dataframe called _numeric_df_. It also creates a list of the column names in numeric_df called _numeric_columns_.

The code then creates a new dataframe called _category_df_ that contains all columns that are not numeric. The column names in category_df are stored in a list called _category_columns_.

Finally, the code calculates the shape of the original dataframe using the shape attribute and calculates the total number of null values in the dataframe using the _isnull()_ and _sum()_ methods.

That settles the basic needs, next let's build the sidebar. Simply use this code:

```
#### > Handles
st.sidebar.title("Page Handles")

uploaded_file = st.sidebar.file_uploader("Choose a File {.csv, .parquet}", type=["csv", "parquet"])

### Read Uploaded data
if uploaded_file is not None:
    if uploaded_file.name.endswith(".csv"):
        df = pd.read_csv(uploaded_file)
    elif uploaded_file.name.endswith(".parquet"):
        df = pd.read_parquet(uploaded_file)
    else:
        st.error("Provide an acceptable file extension")
```

This code block creates a file uploader widget in the sidebar of your Streamlit app using _st.sidebar.file_uploader_. The widget allows users to upload files with the extensions **.csv** or **.parquet**.

The code then reads the uploaded file into a Pandas DataFrame using _pd.read_csv_ if the file is a CSV or _pd.read_parquet_ if the file is a Parquet file. If the uploaded file has an extension other than .csv or .parquet, an error message is displayed using _st.error_.

```
### Option to edit data
Editable_Status = {0: "No", 1: "Yes"}
if st.sidebar.checkbox("Would you like to edit your data"):
    edit_df = st.experimental_data_editor(df, use_container_width=True)
    # edit_df.to_csv('./dta/edit_data.csv')
    # df = pd.read_csv("./dta/edit_data.csv")
    Editable_Status = Editable_Status[1]
else:
    st.dataframe(df, use_container_width=True)
    Editable_Status = Editable_Status[0]
```

```
st.sidebar.divider()

s_Col1, s_Col2, s_Col3 = st.sidebar.columns(3)
with s_Col1:
    st.markdown("### DATA SHAPE")
    st.write(shape)

with s_Col2:
    st.markdown("### EDITABLE STATUS")
    st.write(Editable_Status)

with s_Col3:
    st.markdown("### NULL VALUES")
    st.write(null_values)
    
st.sidebar.divider()  
```

This code block creates a sidebar in your Streamlit app using _st.sidebar_. The sidebar contains three columns, each with a title and a value. The first column displays the shape of the data, the second column displays the editable status, and the third column displays the number of null values in the data.

The code also adds a divider between the columns using `st.sidebar.divider()`. Finally, it adds an informational message to the sidebar using st.sidebar.info.

#### 2.2 Main Bar
Now, let's move to the main bar (no pun intended - 😉).

```
#### > Exploration Page
st.header("Exploratory Data Analysis")
st.markdown("Simple Data Visualization/Exploration Tool for quickly Probing, Visualizing and Analyzing Data Sets")

st.divider()
st.markdown('#### Univariate Analysis')
# st.markdown("<h5 style='text-align: center;'>Univariate Analysis</h5>", unsafe_allow_html=True)
```

This code block creates a header with the text “Exploratory Data Analysis” using _st.header_. It also creates a markdown section with the text “Simple Data Visualization/Exploration Tool for quickly Probing, Visualizing and Analyzing Data Sets” using _st.markdown_.

The code then adds a divider using _st.divider_ and creates another markdown section with the text “Univariate Analysis” using _st.markdown_.

```
col1, col2 = st.columns(2)
with col1:
    num_option = st.selectbox("Select Distribution of a Numerical Variable", numeric_columns)
    fig = px.histogram(df, x=num_option, opacity=0.75)
    fig.update_layout(bargap=0.2, uniformtext_minsize=12, uniformtext_mode='hide')
    st.plotly_chart(fig, use_container_width=True)
with col2:
    cat_option = st.selectbox("Select Distribution of a Categorical Variable", category_columns)
    fig = px.histogram(df, x=cat_option)
    fig.update_layout(bargap=0.2)
    st.plotly_chart(fig, use_container_width=True)
st.divider()
```

This code block creates displays two histograms side by side. The first histogram shows the distribution of a numerical variable selected by the user using a dropdown menu. The second histogram shows the distribution of a categorical variable selected by the user using another dropdown menu.

The histograms are created using the Plotly Express library and are displayed using the _st.plotly_chart_ function. The `use_container_width=True` parameter ensures that the charts are displayed at full width.

```
st.markdown('#### Bivariate Analysis')
st.write("Distribution of the Selected Columns")
col3, col4, col5 = st.columns(3)
with col3:
    num_option1 = st.selectbox("Select Numerical Variable", numeric_columns)
with col4:
    numeric_columns_x = numeric_df.drop(columns=num_option1).columns.tolist()
    num_option2 = st.selectbox("Select another Numerical Variable", numeric_columns_x)
with col5:
    cat_option1 = st.selectbox("Select a Categorical Identifier", category_columns)
fig = px.histogram(df, x=num_option2, y=num_option1, color=cat_option1,
                   marginal="box",  # or violin, rug
                   hover_data=df.columns)
st.plotly_chart(fig, use_container_width=True)
st.divider()

st.subheader("Data Page")
st.warning("Check the _Editable Dataframes_ option on the side widget to enable you change your values")
```

This code snippet performs _bivariate analysis_ on a dataset- creating a user interface for selecting the columns of interest. 

The first line of code creates a markdown header with the text "Bivariate Analysis". The second line writes the text "Distribution of the Selected Columns" to the user interface. 

The next three lines create three columns using `st.columns(3)`. The first column contains a select box for choosing a numerical variable, the second column contains another select box for choosing another numerical variable, and the third column contains a select box for choosing a categorical identifier. 

The next few lines create a histogram using _Plotly_ with the selected variables and categorical identifier. Finally, the last two lines create a subheader with the text "Data Page" and a warning message advising users to check the _Editable Dataframes_ option on the side widget to enable them to change their values.

```
st.sidebar.info("Made with ❤ by the AfroLogicInsect")
```

## 3. Test & Deploy

It is important that we create a version for all this work and store them. Start by adding all items to Git Bash and deploy to GitHub, you can follow these steps:

First, navigate to the directory where your project is located using the cd command in your command prompt/terminal window.

Initialize a new Git repository by running the following command:

`git init`

Add the files you want to include in your repository using the following command:

`git add .` (Yes, the full stop is part of it)

This will add all files in the current directory to your repository. (If you only want to add specific files, replace . with the file names)
Commit your changes using the following command:

`git commit -m "Initial commit"`

Create a new repository on GitHub by clicking on the “New” button on your GitHub dashboard.

Give your repository a name and click on “Create repository”.
Copy the URL of your new repository.
Add a remote to your local repository using the following command:

`git remote add origin <repository URL>`

Push your changes to GitHub using the following command:

`git push -u origin master`

This will push all of your changes to GitHub and create a new branch called _**master**_. You can now view your repository on GitHub and see all of your files and commits.

If you’re new to Git Bash and Python, here are some resources that can help you get started: [Git Bash Tutorial](https://www.git-tower.com/blog/git-bash/), [GitHub Tutorial](https://dev.to/casualcoders/how-to-create-and-push-a-new-git-repo-to-github-3j15).

And that's it! With just a few lines of code, we were able to create this web-app that can perform EDA on any dataset. You can check out the source code on my GitHub [repository](https://github.com/akanimohod19a/autoeda/blob/basic_dist/app.py): 

I hope you enjoyed this blog post and learned something new. If you have any questions or feedback, please leave a comment below or contact me: danielamahtoday@gmail.com

Thanks for reading and happy coding!