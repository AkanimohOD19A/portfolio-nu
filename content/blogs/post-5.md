---
title: 'git init cohere_streamlit pt1'
date: '2025-02-16T07:34:32+01:00'
draft: false
github_link: "https://github.com/AkanimohOD19A/initial-cohere"
description: "git init cohere_streamlit"
tags: ["data-science", "machine-learning", "cohere", "streamlit", "YOLOv6", "tutorial", "lifecycle"]
author: "AfroLogicInsect"
image: static/images/blogs-placeholder.png
---

In this post, we will generally review how to create a web app that generates text based on keywords using Streamlit and Cohere. Streamlit is a framework that lets you create interactive web apps with Python and Cohere is a natural language generation (NLG) platform that lets you generate high-quality text from keywords, prompts, or data from a simple API call.

The web app we are going to create is inspired by this one: https://initialcohere-ideagenerator.streamlit.app/. It allows the user to enter some keywords and choose some characteristics for the text, such as the format, the length, and the tone. Then, it uses Cohere's Keyword API to generate a text snippet that matches the user's input. The user can also see some suggested responses that can help them improve or rewrite the text.

To create this web app, we will need the following:

- A [Streamlit](https://streamlit.io/) account and a [Cohere] (https://coherent.ai/) account. You can sign up for free on their websites.
- A [GitHub](https://github.com/) account and a repository where we will store our code and deploy our app.

Let's get started!

Step 1: Write the code for the web app

The code for the web app is written in Python and uses Streamlit to create the user interface then Cohere to generate the text. You can find the full [tutorial](https://txt.cohere.com/deploy-cohere-streamlit/) and code in this [repository](https://github.com/AkanimohOD19A/initial-cohere/blob/master/app.py) where I add the following functions:
1. Ability to download generated ideas
2. Selection from the 4 different models

Here is a brief overview of what the code does:

- First, install and import the necessary modules, such as cohere, Streamlit, requests, and os. It also sets some constants, such as the Cohere API key, the Cohere endpoint, and the Streamlit theme.
```
pip install cohere
pip install streamlit
pip install requests
pip install pandas
```
- Next, it defines some helper functions that will be used later in the code. These functions are:
  - `generate_text`: This function takes the user's input (keywords and characteristics) and sends a request to the Coherent Keyword API. It returns the generated text or an error message if something goes wrong.
  - `get_suggestions`: This function takes the generated text and returns some suggested responses that can help the user improve or rewrite the text. It uses a simple heuristic that checks for some common issues, such as spelling, grammar, punctuation, repetition, clarity, and coherence. It also limits the suggestions to five words or less.
- Then, we create the main layout of the web app using Streamlit's widgets. It uses `st.title` to display the title of the app, `st.sidebar` to create a sidebar where the user can enter their input, `st.text_area` to display the generated text, and `st.markdown` to display some instructions and information.

- Finally, it adds some logic to handle the user's input and output. It uses `st.button` to create a button that triggers the text generation process when clicked. It also uses `st.spinner` to show a loading message while waiting for the response from Cohere. It then calls the helper functions defined earlier to generate the text, get the suggestions, and format the text. It displays the results using `st.text_area` and `st.markdown`.

That's it for the code! You can run it locally on your machine by simply running `streamlit run app.py` in your terminal. You should see something like this:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4ruzvxngb7jky9i1vud5.png)

Step 2: Deploy the web app

Now that we have written our code, we need to deploy it so that anyone can access it online. To do this, we need to follow these steps:

- Create a file named `requirements.txt` in our repository that lists all the modules that our app depends on. In our case, we only need Streamlit, Cohere and pandas.
```
cohere==4.11.2
streamlit==1.21.0
pandas==1.5.3
```
- Create a file named `secrets.toml` within a `.streamlit` directory and store your keys. This ensures that when you deploy the API keys are not exposed. You would similarly add your keys under Secrets in your Streamlit 'Advanced Settings' when deploying your app.
- Initiate a git repository that stores your code after which you _push_ the code to a GitHub repository.
- Now, Click on the widget on the top-right of your web-app and select 'Deploy App', fill out the required forms and That's it! We have successfully deployed our web app on Streamlit. 

Congratulations! You have learned how to create a web app that generates text based on keywords using Streamlit and Cohere. I hope you enjoyed this tutorial and found it useful. If you have any questions or feedback, please let me know in the comments below. Thank you for reading!