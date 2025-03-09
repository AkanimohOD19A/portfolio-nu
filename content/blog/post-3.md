---
title: 'Model Selection & Hyper Parameter Tuning in Binary Data Classification (Practical Example)'
date: '2025-02-16T07:34:32+01:00'
draft: true
github_link: "https://github.com/AkanimohOD19A"
description: "real-time: Streamlit and monitoring query response"
tags: ["data-science", "machine-learning", "streamlit", "ChatGPT", "tutorial", "python", "EDA"]
author: "AfroLogicInsect"
image: /images/blog-placeholder.png
---

How can you identify Fraud using a Dataset on Energy Consumption in a relationship between Service Provider and Energy consumers? To determine the class of Fraud, 0 or 1, where 0 stands for No Instance of Fraud and 1 otherwise.

This data was recently hosted on [Zindi](https://zindi.africa/competitions/indabax-nigeria-23) for a prediction exercise. Let's see how we can approach this, selecting a Model and Tuning the selected model for the best Parameters which makes the best accuracy score which will in turn make the best predictions. It is in the same vein, that this is not addressed for Exploratory Data Analysis or other forms of Feature Selection or Engineering. The Author has assumed that you had moved past this as well as have partitioned your data in Train (X_train) and Test (X_test) Data in a workflow.

## Model Selection 💽

First, not every model is useful in producing a metric, hence we invoke a list of applicable Models, since our problem is Binary Classification (Yes/No, Fraudulent/Not Fraudulent, 1/0, etc.) - applicable models (at least from the Scikit Learn library) is Logistic Regression, as well as Support Vector Machines (SVM), Decision Trees, Random Forests, Gradient Boosting, etc. 
Use the code chunk below to import them:

```
## Modelling Dependencies
from sklearn.linear_model import LogisticRegression
from sklearn.svm import SVC
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier

## Evaluation Dependencies
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score, roc_auc_score, log_loss
from sklearn.model_selection import cross_val_score
```

Now that we have our models, we create a list of them and 
```
## Model Selection
models = []
models.append(('LR', LogisticRegression()))
models.append(('SVC', SVC())) 
models.append(('DTC', DecisionTreeClassifier()))
models.append(('RTC', RandomForestClassifier()))
models.append(('GBC', GradientBoostingClassifier()))
```
Next, we evaluate:

```
results = []
names = []
for name, model in models:
    fit_model = model.fit(X_train, y_train)
    y_pred = fit_model.predict(X_test)

    accuracy = accuracy_score(y_test, y_pred)
    precision = precision_score(y_test, y_pred)
    recall = recall_score(y_test, y_pred)
    f1 = f1_score(y_test, y_pred)
    roc_auc = roc_auc_score(y_test, y_pred)
    log_loss = log_loss(y_test, y_pred)

    ## Cross Validation
    cv = cross_val_score(model, train_df, y_df, cv=7)

    results.append((accuracy, precision, recall, f1, roc_auc, log_loss))
    names.append(name)
    print()
    print(cv)
    print()
    print('{}:Accuracy {} - Precision {} - Recall {} - F1 Score {} - ROC AUC {} - Log Loss {}'.format(name,
                                                                                                    round(accuracy, 3),                                                                                                    round(precision, 3),                                                                                                    round(recall, 3),                                                                                                    round(f1, 3),                                                                                    round(roc_auc, 3),                                                                                 round(log_loss, 3))))
```
In this example, the Accuracy, _Precision, Recall, F1 Score, ROC AUC_, and _Log Loss_ metrics are used to evaluate the performance of a model. 

The **Accuracy** measures how often the model makes correct predictions. The **Precision** measures the proportion of true positives among all positive predictions. The **Recall** measures the proportion of true positives among all actual positives. The **F1 Score** is the harmonic mean of precision and recall. The **ROC AUC** measures the ability of the model to distinguish between positive and negative classes. The **Log Loss** measures the performance of a classification model where the prediction output is a probability value between 0 and 1.

## Hyper-parameter Tuning  📻

At this stage our code chunks have run and we have decided on the model with the highest accuracy using the **_ROC AUC_** metric, which evaluates this performance and determines the best model based off the highest evaluation score, then we start tuning its parameters to achieve better results.

Keep in mind that for the _ROC AUC_ metric, a higher score is better. A score of 0.5 indicates that the model is no better than random guessing, while a score of 1.0 indicates perfect performance. 

Let's assume that the best score was from the Logistic Regression Model

Logistic Regression already has default parameters, but we can do better, we can say for instance that instead of the parameter solver = 'sag' we want solver to be 'saga' or so and so, then see how that impacts performance. It is very likened to tuning the radio dial. In the following example we are tuning the _penalty_,_C_,_solver_,_max_iter_ parameters.

```
param_grid = {'penalty' : ['l1', 'l2', 'elasticnet', 'none'],
    'C' : np.logspace(-4, 4, 20),
    'solver' : ['lbfgs','newton-cg','liblinear','sag','saga'],
    'max_iter' : [100, 1000, 2500, 5000]
    }
```

Tip: To quickly figure out the parameters in your model, run '?[model name]'

```
LGR = LogisticRegression() # Initiate Model
LGR_cv = GridSearchCV(estimator=LGR, param_grid=param_grid, cv = 3, verbose=2, n_jobs=-1) # Introduce parameters, cross-validation and output logs
LGR_cv.fit(X_train, y_train) # Fit on Data
```

You would then see a log output like:
**Fitting 3 folds for each of 1600 candidates, totalling 4800 fits** indicating the dimension of this search.

Once, this code finishes its run, we need to retrieve the best parameters, we can run the following:
```
# Retrieve best parameters
best_params = LGR_cv.best_params_
best_params
```
Introduce these parameters into your model, like so.
```
model = LogisticRegression(penalty=best_params['penalty'],
C=best_params['C'],
solver=best_params['solver'],
max_iter=best_params['max_iter'])
```
Fit, again.
```
model.fit(X_train, y_train)
```

```
predictions = model.predict(unpreicted_df)
```
Then predict away 🚀🚀.