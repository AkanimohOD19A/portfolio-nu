---
title: 'Local Workflow: Orchestrating Data Ingestion into Airtable'
date: '2025-02-16T07:34:32+01:00'
draft: false
github_link: "https://github.com/AkanimohOD19A/scheduling_airtable_insertion"
description: "DataFlow orchestration with AirTable automated ingestion"
tags: ["MLOPs", "DevOps", "Automation", "AirTable", "data-science"]
author: "AfroLogicInsect"
image: blog-images/blogs-placeholder.png
---


![Local Workflow](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/smc6a9htjvvlzzq2em94.jpg)

## Introduction

The overall data lifecycle starts with generating data and storing it in some way, somewhere. Let's call this the early-stage data lifecycle and we'll explore how to automate data ingestion into **Airtable** using a local workflow. We'll cover setting up a development environment, designing the ingestion process, creating a batch script, and scheduling the workflow - keeping things simple, local/reproducible and accessible. 
First, let's talk about [Airtable](https://www.airtable.com/solutions/project-management#:~:text=Roll%20up%2C%20drill%20down,programs%2C%20projects%2C%20and%20tasks.). **Airtable** is a powerful and flexible tool that blends the simplicity of a spreadsheet with the structure of a database. I find it perfect for organizing information, managing projects, tracking tasks, and it has a free tier!



## Preparing the Environment
###### Setting up the development environment
We would be developing this project with python, so lunch your favorite IDE and create a virtual environment
```bash
# from your terminal
python -m venv <environment_name>
<environment_name>\Scripts\activate
```

To get started with Airtable, head over to [Airtable's](https://www.google.com/url?sa=E&source=gmail&q=https://airtable.com/) website. Once you've signed up for a free account, you'll need to create a new Workspace. Think of a Workspace as a container for all your related tables and data.

Next, create a new Table within your Workspace. A Table is essentially a spreadsheet where you'll store your data. Define the Fields (columns) in your Table to match the structure of your data.

Here's a snippet of the fields used in the tutorial, it's a combination of _Texts_, _Dates_ and _Numbers_: 

![glimpse fields](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/lg3rbebi55edejrq2fv5.png)

To connect your script to Airtable, you'll need to generate an API Key or [Personal Access Token](https://support.airtable.com/docs/creating-personal-access-tokens). This key acts as a password, allowing your script to interact with your Airtable data. To generate a key, navigate to your Airtable account settings, find the API section, and follow the instructions to create a new key.

**Remember to keep your API key secure. Avoid sharing it publicly or committing it to public repositories. **

###### Installing necessary dependencies (Python, libraries, etc.)
Next, `touch requirements.txt`. Inside this _.txt_ file put the following packages:
```txt
pyairtable
schedule
faker
python-dotenv
```
now run `pip install -r requirements.txt` to install the required packages.
###### Organizing the project structure
This step is where we create the scripts, *.env* is where we will store our credentials, *autoRecords.py* - to randomly generate data for the defined fields and the **ingestData.py** to insert the records to Airtable.

###### Designing the Ingestion Process: Environment Variables

![Part_Creds](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qsnaucx04jyb8xykez12.jpg)

```.env
"https://airtable.com/app########/tbl######/viw####?blocks=show"
BASE_ID = 'app########'
TABLE_NAME = 'tbl######'
API_KEY = '#######'
```


## Designing the Ingestion Process: Automated Records
Sounds good, let's put together a focused subtopic content for your blog post on this employee data generator. 

###### Generating Realistic Employee Data for Your Projects

When working on projects that involve employee data, it's often helpful to have a reliable way to generate realistic sample data. Whether you're building an HR management system, an employee directory, or anything in between, having access to robust test data can streamline your development and make your application more resilient.

In this section, we'll explore a Python script that generates random employee records with a variety of relevant fields. This tool can be a valuable asset when you need to populate your application with realistic data quickly and easily.

###### Generating Unique IDs

The first step in our data generation process is to create unique identifiers for each employee record. This is an important consideration, as your application will likely need a way to uniquely reference each individual employee. Our script includes a simple function to generate these IDs:

```python
def generate_unique_id():
    """Generate a Unique ID in the format N-#####"""
    return f"N-{random.randint(10000, 99999)}"
```

This function generates a unique ID in the format "N-#####", where the number is a random 5-digit value. You can customize this format to suit your specific needs.

###### Generating Random Employee Records

Next, let's look at the core function that generates the employee records themselves. The `generate_random_records()` function takes the number of records to create as input and returns a list of dictionaries, where each dictionary represents an employee with various fields:

```python
def generate_random_records(num_records=10):
    """
    Generate random records with reasonable constraints
    :param num_records: Number of records to generate
    :return: List of records formatted for Airtable
    """
    records = []

    # Constants
    departments = ['Sales', 'Engineering', 'Marketing', 'HR', 'Finance', 'Operations']
    statuses = ['Active', 'On Leave', 'Contract', 'Remote']

    for _ in range(num_records):
        # Generate date in the correct format
        random_date = datetime.now() - timedelta(days=random.randint(0, 365))
        formatted_date = random_date.strftime('%Y-%m-%dT%H:%M:%S.000Z')

        record = {
            'fields': {
                'ID': generate_unique_id(),
                'Name': fake.name(),
                'Age': random.randint(18, 80),
                'Email': fake.email(),
                'Department': random.choice(departments),
                'Salary': round(random.uniform(30000, 150000), 2),
                'Phone': fake.phone_number(),
                'Address': fake.address().replace('\n', '\\n'),  # Escape newlines
                'Date Added': formatted_date,
                'Status': random.choice(statuses),
                'Years of Experience': random.randint(0, 45)
            }
        }
        records.append(record)

    return records
```

This function uses the Faker library to generate realistic-looking data for various employee fields, such as name, email, phone number, and address. It also includes some basic constraints, such as limiting the age range and salary range to reasonable values.

The function returns a list of dictionaries, where each dictionary represents an employee record in a format that is compatible with Airtable.

###### Preparing Data for Airtable

Finally, let's look at the `prepare_records_for_airtable()` function, which takes the list of employee records and extracts the 'fields' portion of each record. This is the format that Airtable expects for importing data:

```python
def prepare_records_for_airtable(records):
    """Convert records from nested format to flat format for Airtable"""
    return [record['fields'] for record in records]
```

This function simplifies the data structure, making it easier to work with when integrating the generated data with Airtable or other systems.

**Putting It All Together**

To use this data generation tool, we can call the `generate_random_records()` function with the desired number of records, and then pass the resulting list to the `prepare_records_for_airtable()` function:

```python
if __name__ == "__main__":
    records = generate_random_records(2)
    print(records)
    prepared_records = prepare_records_for_airtable(records)
    print(prepared_records)
```

This will generate 2 random employee records, print them in their original format, and then print the records in the flat format suitable for Airtable.

Run: ```python
python autoRecords.py
```
Output:

```
## Raw Data
[{'fields': {'ID': 'N-11247', 'Name': 'Christine Cummings', 'Age': 22, 'Email': 'perezbill@example.net', 'Department': 'Finance', 'Salary': 149928.17, 'Phone': '(999)961-2703', 'Ad
dress': 'USNV Wheeler\\nFPO AP 08774', 'Date Added': '2024-11-06T15:50:39.000Z', 'Status': 'On Leave', 'Years of Experience': 8}}, {'fields': {'ID': 'N-48578', 'Name': 'Stephanie O
wen', 'Age': 41, 'Email': 'nicholasharris@example.net', 'Department': 'Engineering', 'Salary': 56206.04, 'Phone': '981-354-1421', 'Address': '872 Shelby Neck Suite 854\\nSeanbury, IL 24904', 'Date Added': '2024-10-15T15:50:39.000Z', 'Status': 'Active', 'Years of Experience': 25}}]

## Tidied Up Data
[{'ID': 'N-11247', 'Name': 'Christine Cummings', 'Age': 22, 'Email': 'perezbill@example.net', 'Department': 'Finance', 'Salary': 149928.17, 'Phone': '(999)961-2703', 'Address': 'US
NV Wheeler\\nFPO AP 08774', 'Date Added': '2024-11-06T15:50:39.000Z', 'Status': 'On Leave', 'Years of Experience': 8}, {'ID': 'N-48578', 'Name': 'Stephanie Owen', 'Age': 41, 'Email': 'nicholasharris@example.net', 'Department': 'Engineering', 'Salary': 56206.04, 'Phone': '981-354-1421', 'Address': '872 Shelby Neck Suite 854\\nSeanbury, IL 24904', 'Date Added': '2024-10-15T15:50:39.000Z', 'Status': 'Active', 'Years of Experience': 25}]
```

## Integrating Generated Data with Airtable
In addition to generating realistic employee data, our script also provides functionality to seamlessly integrate that data with Airtable

###### Setting up the Airtable Connection

Before we can start inserting our generated data into Airtable, we need to establish a connection to the platform. Our script uses the `pyairtable` library to interact with the Airtable API. We start by loading the necessary environment variables, including the Airtable API key and the Base ID and Table Name where we want to store the data:

```python
import os
from dotenv import load_dotenv
from pyairtable import Api
import logging

# Load environment vars
load_dotenv()

# Credentials
API_KEY = os.getenv("API_KEY")
BASE_ID = os.getenv("BASE_ID")
TABLE_NAME = os.getenv("TABLE_NAME")
```

With these credentials, we can then initialize the Airtable API client and get a reference to the specific table we want to work with:

```python
def main():
    # Initiate Connection
    api = Api(API_KEY)
    table = api.table(BASE_ID, TABLE_NAME)
```

###### Inserting the Generated Data

Now that we have the connection set up, we can use the `generate_random_records()` function from the previous section to create a batch of employee records, and then insert them into Airtable:

```python
def main():
    # ... (connection setup)

    num_records = 50

    try:
        # Generate and prep. data
        auto_records = generate_random_records(num_records)
        prepd_records = prep_for_insertion(auto_records)

        # Insert Data
        print("inserting records... ")
        created_records = table.batch_create(prepd_records)
        print(f"Successfully inserted {len(created_records)} records")

    except Exception as e:
        logger.error(f"An error occurred: {str(e)}")
        raise
```

The `prep_for_insertion()` function is responsible for converting the nested record format returned by `generate_random_records()` into the flat format expected by the Airtable API. Once the data is prepared, we use the `table.batch_create()` method to insert the records in a single bulk operation.

###### Error Handling and Logging

To ensure our integration process is robust and easy to debug, we've also included some basic error handling and logging functionality. If any errors occur during the data insertion process, the script will log the error message to help with troubleshooting:

```python
import logging

# Set Up logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

def main():
    # ... (connection setup and data insertion)

    try:
        # Insert Data
        # ...
    except Exception as e:
        logger.error(f"An error occurred: {str(e)}")
        raise
```

By combining the powerful data generation capabilities of our earlier script with the integration features shown here, you can quickly and reliably populate your Airtable-based applications with realistic employee data. 

## Scheduling Automated Data Ingestion with a Batch Script

To make the data ingestion process fully automated, we can create a batch script (.bat file) that will run the Python script on a regular schedule. This allows you to set up the data ingestion to happen automatically without manual intervention.

Here's an example of a batch script that can be used to run the `ingestData.py` script:

```batch
@echo off
echo Starting Airtable Automated Data Ingestion Service...

:: Project directory
cd /d <absolute project directory>

:: Activate virtual environment
call <absolute project directory>\<virtual environment>\Scripts\activate.bat

:: Run python script
python ingestData.py

:: Keep the window open if there's no error
if %ERRORLEVEL% NEQ 0 (
    echo An error ocurred! Error code: %ERRORLEVEL%
    pause
)
```

Let's break down the key parts of this script:

1. `@echo off`: This line suppresses the printing of each command to the console, making the output cleaner.
2. `echo Starting Airtable Automated Data Ingestion Service...`: This line prints a message to the console, indicating that the script has started.
3. `cd /d C:\Users\buasc\PycharmProjects\scrapEngineering`: This line changes the current working directory to the project directory where the `ingestData.py` script is located.
4. `call C:\Users\buasc\PycharmProjects\scrapEngineering\venv_airtable\Scripts\activate.bat`: This line activates the virtual environment where the necessary Python dependencies are installed.
5. `python ingestData.py`: This line runs the `ingestData.py` Python script.
6. `if %ERRORLEVEL% NEQ 0 (... )`: This block checks if the Python script encountered an error (i.e., if the `ERRORLEVEL` is not zero). If an error occurred, it prints an error message and pauses the script, allowing you to investigate the issue.

To schedule this batch script to run automatically, you can use the Windows Task Scheduler. Here's a brief overview of the steps:

1. Open the Start menu and search for "Task Scheduler".
Or
Windows + R and ![Windows + R](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ca6r96l16yk2eemw1jqt.png)
2. In the Task Scheduler, create a new task and give it a descriptive name (e.g., "Airtable Data Ingestion").
3. In the "Actions" tab, add a new action and specify the path to your batch script (e.g., `C:\Users\buasc\PycharmProjects\scrapEngineering\ingestData.bat`).
4. Configure the schedule for when you want the script to run, such as daily, weekly, or monthly.
5. Save the task and enable it.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/acepfjyxxiqu8vy30rhw.png)


Now, the Windows Task Scheduler will automatically run the batch script at the specified intervals, ensuring that your Airtable data is updated regularly without manual intervention.

## Conclusion
This can be an invaluable tool for testing, development, and even demonstration purposes.

Throughout this guide, you've learnt how to set up the necessary development environment, designed an ingestion process, created a batch script to automate the task, and scheduled the workflow for unattended execution. Now, we have a solid understanding of how to leverage the power of local automation to streamline our data ingestion operations and unlock valuable insights from an _Airtable_ - powered data ecosystem.

Now that you've set up the automated data ingestion process, there are many ways you can build upon this foundation and unlock even more value from your Airtable data. I encourage you to experiment with the code, explore new use cases, and share your experiences with the community.

Here are some ideas to get you started:
- Customize the Data Generation
- Leverage the Ingested Data [Markdown-based exploratory data analysis (EDA), Build interactive dashboards or visualizations using tools like Tableau, Power BI, or Plotly, Experiment with machine learning workflows (predicting employee turnover or identifying top performers)]
- Integrate with Other Systems [cloud functions, webhooks, or data warehouses]

The possibilities are endless! I'm excited to see how you build upon this automated data ingestion process and unlock new insights and value from your Airtable data. Don't hesitate to experiment, collaborate, and share your progress. I'm here to support you along the way.

See the full code https://github.com/AkanimohOD19A/scheduling_airtable_insertion, full video tutorial is on the way.

