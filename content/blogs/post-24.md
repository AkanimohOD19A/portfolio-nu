---
title: 'Generalized Additive Models (GAMs) in R: Handling Non-linearity in Dolphin Behavior Analysis'
date: '2025-07-06T07:34:32+01:00'
draft: false
github_link: "https://youtu.be/TUfgTzO_n60?list=PLLE0uKjUE6Vey0iOCy6jR4_7BLVAr62eF"
description: "Diving into ensemble methods and see how R makes it incredibly easy to build, deploy, and interact with sophisticated machine learning models."
tags: ["Generalized Additive Models", "Data Analytics", "Additive", "Machine Learning", "Data Science"]
author: "AfroLogicInsect"
image: /images/post-24.png
---
<!-- 
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wyh43zvg9fm232nnd69s.gif) -->

_Initially published on May 27, 2025_
GAMs (Generalized Additive Models) handle non-linearity in model development through their flexible approach. The "additive" in the model implies that the response variable can be modeled as a sum of smooth functions of predictor variables, allowing for non-linear relationships without requiring specific parametric forms.

## Understanding Splines in GAMs

GAMs use what are called "splines" - these act as smoothing factors for variables that show non-linear tendencies, particularly when dealing with continuous numeric variables. Splines work by dividing the data into segments and fitting polynomial functions to each segment, connected at knots to ensure smoothness.


## Implementing GAMs in R: Dolphin Behavior Case Study
Let's examine a simple implementation of GAMs in R using the mgcv package, analyzing a dataset of common bottlenose dolphin behavior. Link to dataset here.

After exploratory data analysis (EDA), we observe that not all variables linearly explain the behavior target variable - particularly continuous variables like speed and distance. We'll accommodate this in our GAM formula:

`library(mgcv)

# Basic GAM model with smooth terms for non-linear variables
dolphin_formula <- behav ~ sm.ps(speed) + sm.ps(distance) + rr + lin + timeper + grpsize + calf

dolphin_model <- vgam(
  dolphin_formula,
  data = train_df,
  family = cumulative(parallel= TRUE)
)`

## Model Evaluation
When we evaluate the model:

`summary(dolphin_model)`
We should see output showing:

- Estimated degrees of freedom for each smooth term
- Significance of smooth terms
- Model fit statistics (R², deviance explained)

![model summary table](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/60m1guf2lwhge48nn9gs.png)

## vGAM vs GAM in R
Key differences between vgam (from the VGAM package) and gam (from gam/mgcv):

- vgam offers vector generalized additive models (more response distributions)
- mgcv::gam has more sophisticated smoothing parameter selection
- vgam can fit reduced-rank smooths

**Final Thoughts**
GAMs are powerful tools when:

Relationships are non-linear but smooth
You have continuous predictors
You need more flexibility than linear models but more structure than pure machine learning

Dataset: https://www.kaggle.com/datasets/erenakbulut/common-bottlenose-dolphin-behavior

RPubs: RPubs - Document

Repo: R_Playlist/GAM at master · AkanimohOD19A/R_Playlist

Youtube: https://youtu.be/TUfgTzO_n60?list=PLLE0uKjUE6Vey0iOCy6jR4_7BLVAr62eF


