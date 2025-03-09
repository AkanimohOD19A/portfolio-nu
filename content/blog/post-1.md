---
title: 'Effortlessly Link Streamlit to Google Sheets for Real-time Analysis'
date: '2025-02-16T07:34:32+01:00'
draft: true
github_link: "https://github.com/AkanimohOD19A/medical_charges_v2"
description: "real-time: Streamlit to Google Sheets"
tags: ["data-science", "machine-learning", "streamlit", "googlesheet", "tutorial", "lifecycle"]
author: "AfroLogicInsect"
image: /images/blog-placeholder.png
---
As a researcher one major headache is the cost of data gathering, simple solutions can be quite manual or very expensive, however we can create a solution that solves this, that is both simple and free, using your **free** Google Account and Streamlit, a UI framework that “turns data scripts into shareable web apps in minutes all in pure Python, no front‑end experience required”. In short, with this tutorial we will build one together. As mentioned, we will need two things you Google Apps Sheet Keys and some knowledge of Streamlit. We will be creating a real-time interface between these two.

## 1 Set Up Google Credentials
Please read [this](https://towardsdatascience.com/how-to-access-google-sheet-data-using-the-python-api-and-convert-to-pandas-dataframe-5ec020564f0e) or the [Google Documentation](https://developers.google.com/sheets/api/quickstart/python) on how to setup your Google Sheet data using the Python API.

## 2 Build a web-app with Streamlit
There are so many templates from Streamlit [insert st link] that we will just pick one and modify, please compliment what you learn with their beautiful documentation [Insert doc]. First things first - create a directory, where all our codes will live. In this dir., create a [inster secrets instruction] `secrets.toml` file, and store your secrets.

Now create a directory that contains `app.py` file that would have our code: 
1. Create a `requirements.txt` file and add the following:
```
gspread
numpy
oauth2client
pandas
scikit_learn
streamlit
plotly
``` 

Now, from your directory navigate to your terminal and run `pip install -r requirements.txt` to install these dependencies.

2. Import our dependencies
```
import streamlit as st
import gspread
from oauth2client.service_account import ServiceAccountCredentials
```

3. Retrieve your credentials
3.1. Create an `.env` file that will store your secrets

3.3. Retrieve your credentials, this includes your scope, and clients, like so.
```
## CREDENTIALS - Custom DEPENDENCIES
CREDENTIALS = {
    "type": "service_account",
    "project_id": "<YOUR-PROJECT-ID>",
    "private_key_id": os.environ["PRIVATE_KEY_ID"],
    "private_key": os.environ["PRIVATE_KEY"],
    "client_email": os.environ["CLIENT_EMAIL"],
    "client_id": os.environ["CLIENT_ID"],
    "auth_url": "https://accounts.google.com/o/oauth2/auth",
    "token_uri": "https://oauth2.googleapis.com/token",
    "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
    "client_x509_cert_url": os.environ["CLIENT_X509_CERT_URL"]
    }

scope = ['https://www.googleapis.com/auth/spreadsheets', 'https://www.googleapis.com/auth/drive']

credentials = ServiceAccountCredentials.from_json_keyfile_dict(CREDENTIALS, scope)

client = gspread.authorize(credentials)
```
*CREDENTIALS* simply retrieves the secrets from your `.env` as well as confirm the json key file we downloaded, then *client* provided the neccessary authorization. 

Remember, to change your *<YOUR-PROJECT-ID>* to your actual Google Project ID.

4. Create an *Update Sheet* function
To connect to your Google sheet and insert a new row
```
def update_sheet(client):
    sheet = client.open(<SHEETNAME>)
    sheet_insurance = sheet.get_worksheet(0)
    sheet_insurance.insert_rows(df.values.tolist())
```
What this function does is, open with the <SHEETNAME>, gets the first [0,1,...] sheet and insert new values into it.

```
# => Retrieving Values
def get_value(val, my_dict):
    for key, value in my_dict.items():
        if val == key:
            return value


# => Retrieving Values II
def get_fvalue(val):
    feature_dict = {"yes": 1, "no": 0}
    for key, value in feature_dict.items():
        if val == key:
            return value
```

A pre-set of dictionary assigned values for certain attributes.
```
### Dictionary/Labels
sex_map = {'male': 1.0, 'female': 0.0}
smoker_map = {'yes': 1.0, 'no': 0.0}
region_map = {'southeast': 3.0, 'southwest': 4.0,
              'northwest': 2.0, 'northeast': 1.0}
```
These functions will now enable us to retrieve the assigned values into our form, which we build next.

5. We will then a create a form like page, to retrieve basic demographic details, like so:
```
## Page Title
st.title('Real-time Monitoring Streamlit and Google Sheets')

## Attributes
# Age
age = st.sidebar.number_input("Age", 1, 100, 18, 1)
# Sex
sex = st.sidebar.radio("Sex", tuple(sex_map.keys()))
# Region
region = st.sidebar.radio("Region", tuple(region_map.keys()))
# Body Mass Index 
bmi = st.sidebar.number_input("Body Mass Index", 10.00, 100.00, 15.96, 0.10)
# Children
children = st.sidebar.number_input("Number of children", min_value=0, value=1, step=1)
# Smoker
smoker = st.sidebar.radio("Do you Smoke?",tuple(smoker_map.keys()))


# Time
entry_date = datetime.now().strftime("%d-%m-%Y")
entry_dir = 'contrib' + '_' + entry_date
```

Now, after we have entered these values, the following will collect all these features and wrangle them into a simple array, like so:

```
## Feature Values
feature_values = [age, get_value(sex, sex_map),
                  bmi, children, get_fvalue(smoker),
                  get_value(region, region_map)]

### Collect
pretty_results = {'age': [age], 'sex': [sex],
                  'bmi': [bmi], 'children': [children],
                  'smoker': [smoker], 'region': [region]}

data_entry_table = pd.DataFrame(pretty_results)

## Google Sheet Version
df = data_entry_table.copy()
df['entry_date'] = entry_dir

## Update the Google Sheet
update_sheet(client)
```
The *feature_values* list simply uses the initial `get_(f)value` functions to retrieve assigned values, *pretty_results* simplies __beautifies__ by converting into a dictionary.

```
st.caption("Data Entry Table")
st.table(data_entry_table)
```

### Optional. To download a copy of the entered details, add the following to your code:

```
if st.sidebar.checkbox("Save a Copy"):

## Download Copy
with open(csv_path, "rb") as file:
   btn = st.download_button(
      label="Download a copy",
      data=file,
      file_name="my-contrib.csv",
      mime="application/octet-stream"
   )
   st.write('Thanks for contributing!')
```

```
st.sidebar.write("Built with ❤️| AfrologicInsect")
```

Phew! Now we are done!

Simply navigate from your directory, to launch your terminal and execute `streamlit run app.py` [Insert Image] it's 🔥 but it gets better. There are so many possible applications for this, especially Lifecycles - Survey/Data Gathering, Customer Churn, Quantitative Analysis, etc.

## 3 That's it!
Create a [Github Repository](https://github.com/AkanimohOD19A/medical_charges_v2), if you do not have one, follow the [instructions](https://www.bing.com/ck/a?!&&p=cb2ba92beba1e6acJmltdHM9MTY5NjAzMjAwMCZpZ3VpZD0xMmRkNmNiYS02ZTQ2LTZkMWQtMDE4MC03ZjI4NmY5YjZjNWEmaW5zaWQ9NTQ1Ng&ptn=3&hsh=3&fclid=12dd6cba-6e46-6d1d-0180-7f286f9b6c5a&psq=deploy+streamlit+app&u=a1aHR0cHM6Ly9kb2NzLnN0cmVhbWxpdC5pby9zdHJlYW1saXQtY29tbXVuaXR5LWNsb3VkL2RlcGxveS15b3VyLWFwcCM6fjp0ZXh0PURlcGxveSUyMHlvdXIlMjBhcHAlMjAxJTIwQWRkJTIweW91ciUyMGFwcCUyMHRvLGFwcCUyMGxhdW5jaCUyMC4uLiUyMDUlMjBZb3VyJTIwYXBwJTIwVVJMJTIw&ntb=1) to push to an online repository.
Push your code repository to GitHub and deploy your application, remember to share. 

While you think of what to build we this, the next tutorial would show to link this with **Dash**, a low-code framework for rapidly building data apps in Python, to create dashboards you would be proud of.

Sample Application:
[Medical Charges] (https://medical-charges.streamlit.app/)