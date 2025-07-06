---
title: 'Tree-Based Models for Alzheimer Disease Classification: A tidymodels Approach'
date: '2025-07-06T07:34:32+01:00'
draft: false
github_link: "https://youtu.be/Ov0ExO8A-aU"
description: "Diving into ensemble methods and see how R makes it incredibly easy to build, deploy, and interact with sophisticated machine learning models."
tags: ["Tree Based Models", "Data Analytics", "Ensembles", "Machine Learning", "Deployment", "Data Science"]
author: "AfroLogicInsect"
image: /images/post-23.png
---

_Initially published on May 29, 2025_

## Introduction
Alzheimer's disease, the most common form of dementia, affects millions worldwide. Early and accurate diagnosis is crucial for treatment and care planning. In this article, I explore how tree-based machine learning models can help classify dementia status using neuroimaging data from the OASIS dataset. What's particularly exciting is how the tidymodels framework in R provides an elegant, consistent interface for implementing and comparing these powerful algorithms.

## The Dataset
The Open Access Series of Imaging Studies (OASIS) provides MRI data from 416 subjects aged 18-96, including 100 with Alzheimer's diagnosis. Our classification target is dementia status (present/absent), with predictors including:

- Demographic variables (age, gender, education)
- MRI-derived measures (whole brain volume, estimated total intracranial volume)
- Clinical dementia rating (CDR)

```
static_df <- read.csv("oasis_cross-sectional.csv") %>% 
  clean_names() %>% 
  select(-hand) %>% 
  mutate(across(
      where(~ is.numeric(.) && any(is.na(.))), ~ coalesce(., mean(., na.rm = TRUE))),
    cdr = as.factor(cdr),
    dementia = as.factor(ifelse(cdr == 0, 0, 1))
```

## The tidymodels Workflow
The beauty of tidymodels lies in its consistent framework for model building:

- Data preprocessing with recipes
- Model specification with parsnip
- Tuning with tune and dials
- Evaluation with yardstick

This workflow remains identical across different model types - only the underlying algorithm changes.

1. Decision Trees: The Interpretable Foundation

Decision trees provide transparent classification rules that clinicians can understand. Our implementation:

```
dct_model <- decision_tree(
  mode = "classification",
  cost_complexity = tune(),
  tree_depth = tune()
) %>% 
  set_engine("rpart")
```

**Key advantages:**

- Visual representation of decision paths
- Automatic feature selection
- Handles mixed data types naturally

2. Random Forests: The Power of Ensembles

Random forests improve accuracy by aggregating many decorrelated trees:

```
rf_model <- rand_forest(
  mode = "classification",
  trees = tune(),
  mtry = tune()
) %>% 
  set_engine("ranger")
```

**Why they shine:**

- Reduces overfitting through bagging
- Provides feature importance metrics
- Handles high-dimensional data well

3. Gradient Boosted Machines: Sequential Improvement

GBMs iteratively improve by focusing on previous errors:

```
gbm_model <- gbm.fit(x = select(train_df, -dementia),
y = train_df$dementia,
            distribution = "multinomial",
            n.trees = 5000,
            shrinkage = 0.01)
```

**Strengths:**

- Often achieves state-of-the-art accuracy
- Flexible loss functions
- Handles class imbalance well

4. XGBoost: The Championship Algorithm

XGBoost adds regularization and efficient computation to GBM:

```
xg_model <- boost_tree(
  trees = tune(),
  learn_rate = tune(),
  tree_depth = tune()
) %>% 
  set_engine("xgboost") %>% 
  set_mode("classification")
```

**Why it's special:**

- Parallel processing for speed
- Built-in cross-validation
- Regularization prevents overfitting

## Model Evaluation
All models can be evaluated consistently:

```
predictions <- final_results %>% 
collect_predictions()

conf_mat <- conf_mat(predictions, truth = dementia, estimate = .pred_class)

autoplot(conf_mat)
```

![Confusion Matrix](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mit9d22wrftjjywrn9m2.png)

## Why This Matters
Tree-based models offer several advantages for medical classification:

1. Interpretability: Especially important in healthcare decisions
2. Handling missing data: Robust to imperfect clinical datasets
3. Nonlinear relationships: Capture complex biological interactions
4. Automatic feature selection: Identify key diagnostic markers

The tidymodels framework makes it remarkably straightforward to implement, compare, and productionize these models while maintaining rigorous statistical practices.

## Conclusion
From simple decision trees to sophisticated boosted ensembles, tree-based models provide a powerful toolkit for model development tasks, in our case a simple classification detection of Alzheimer’s. The tidymodels ecosystem in R democratizes access to these techniques through its consistent, tidy interface. As neuroimaging datasets grow larger and more complex, these methods will become increasingly valuable in the quest to understand and diagnose dementia earlier and more accurately.

What excites me most is how accessible these advanced techniques have become - with relatively concise code, we can implement models that would have required specialized expertise just years ago. The intersection of statistical learning and healthcare has remains promising!

RPubs: RPubs - Tree Based Models
Full Code: R_Playlist/TBM at master · AkanimohOD19A/R_Playlist
Youtube: https://youtu.be/Ov0ExO8A-aU