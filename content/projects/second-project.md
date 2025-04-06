---
title: "MLOPs: Sentiment Analysis with DISTILBert"
date: 2024-04-06
draft: false
github_link: "https://github.com/AkanimohOD19A/mlops-sentiment-distilbert"
links:
    - icon: fa-solid fa-cloud
    - url: https://share.streamlit.io/user/akanimohod19a
author: "AfroLogicInsect"
tags: 
    - "Python"
    - "DISTILBert"
    - "NLP"
    - "Machine Learning Operations"
    - "Machine Learning Monitoring"
    - "CI/CD"
    - "Docker"
    - "Render"
    - "HuggingFace Models"
image: /images/placeholder.jpg
description: Utilizing MLOPs for automating the entire ML LifeCycle of NLP-Emotion Classifier
project_tags: ["Python", "DISTILBert", "NLP", "MLOp", 
               "CI/CD", "Docker", "Render", "HuggingFace"]
status: "completed"
weight: 1
---


# Emotion Classification API Summary

A production-ready MLOps solution for text emotion classification using a fine-tuned DistilBERT model.

## Key Features
- 🚀 **FastAPI** service for low-latency inference
- 🤗 **Hugging Face** model hosting ([AfroLogicInsect/emotionClassifier](https://huggingface.co/AfroLogicInsect/emotionClassifier))
- 🔄 Automated **CI/CD pipeline** with GitHub Actions
- 🐳 **Containerized** deployment on [Render](https://mlops-sentiment-distilbert.onrender.com/)

## Technical Stack
- **Model**: DistilBERT fine-tuned for 6 emotions (anger, fear, joy, love, sadness, surprise)
- **Infrastructure**: Docker, GitHub Actions, Render
- **Monitoring**: Basic metrics via Render dashboard

## Usage Examples
```bash
# cURL
curl -X POST "https://mlops-sentiment-distilbert.onrender.com/predict" \
  -H "Content-Type: application/json" \
  -d '{"text": "I'm excited!"}'
```

```python
# Python
import requests
response = requests.post("https://mlops-sentiment-distilbert.onrender.com/predict", 
                        json={"text": "I'm excited!"})
print(response.json())
```

## Project Structure
```
emotion-classifier/
├── app/                    # FastAPI application
├── tests/                  # Unit tests
├── .github/workflows/      # CI/CD automation
└── Dockerfile              # Container config
```

## Resources
- [GitHub Repository](https://github.com/AkanimohOD19A/mlops-sentiment-distilbert.git)
- [Live Demo](https://mlops-sentiment-distilbert.onrender.com/)
- [Training Notebook](notebooks/training_notebook.ipynb) (in repo)
- [Hugging Face Model](https://huggingface.co/AfroLogicInsect/emotionClassifier)

## Workflow Diagrams
1. [Training to Deployment](#flowchart-training-to-deployment)
2. [CI/CD Pipeline](#cicd-workflow)
3. [Model Serving](#model-serving)

The solution demonstrates complete MLOps implementation from model training to production deployment.
