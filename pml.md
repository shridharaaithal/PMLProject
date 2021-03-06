---
title: "Classifying Type of Exercise Based on Activity Data"
author: "Shridhara Aithal"
date: "Friday, August 21, 2015"
output: html_document
---

Classifying Type of Barbell Lifting
===================================

## Overview

Human activity recognition involves collecting data from wearable devices, using which the type of activity a person is involved in is determined. Such data is used for various types of analyses. In this report, we use the activity data collected while the participants were performing barbell lifts [1]. The objective is to build a model that classifies and predicts how barbells are lifted.

## Data Loading and Preprocessing

We first load the data "pml-training.csv". We assume that the data file is present in the current working directory. While loading the data, we consider both "NA" and blank fields as missing values.


```r
hra <- read.csv("pml-training.csv", na.strings = c("NA", ""))
```
The data contains many variables that we will not be using for fitting the model. We first remove the serial number, user name, window, and time stamp columns. We then remove variables that have more than 50% NA in them. The rationale is that if the data is not available half the time, they are not useful for predicting the outcome. (It is possible that missing data has an impact on the outcome, however, for the purpose of this report, we ignore them.)


```r
library(plyr)
library(dplyr)
library(tidyr)
hra <- select(hra, -X, -user_name, -raw_timestamp_part_1, -raw_timestamp_part_2, 
              -cvtd_timestamp, -new_window, -num_window)

#Count percent of NAs in each column and change the result to key-value pair
hraNA <- gather(summarise_each(hra, funs(s = sum(is.na(.))/nrow(hra))))
hraNA$key <- as.character(hraNA$key)

#Select only those columns that have <50% NAs
hra <- hra[, hraNA[hraNA$value < 0.5, "key"]]
```

## Model Generation and Prediction

We first split the data 60-40 as training and test set. We fit a model to the training data and use the fitted model to first estimate the out of sample error and then predict the outcome for the test data. We use the following parameters for building the model:

* Training/Testing split: 60% - 40%
* Classifier Algorithm: Stochastic Gradient Boosting (Trees with Gradient Boosting)
* Resampling method: 10-fold cross validation
* Outcome: Variable named "classe"
* Predictors: All variables (other than classe) that were not eliminated during preprocessing


```r
library(caret)
set.seed(1416)
train <- createDataPartition(y = hra$classe, p=0.6, list=F)
hraTrain <- hra[train, ]
hraTest <- hra[-train, ]

modFit <- train(classe ~ ., method="gbm", data=hraTrain, 
                trControl = trainControl(method = "cv", number = 10, repeats = 1), 
                verbose = F)
```

We first use the model to compute in-sample error and estimate out of sample error.

```r
cmIn <- confusionMatrix(hraTrain$classe, predict(modFit, hraTrain))
errIn <- 100 - round(cmIn$overall["Accuracy"] * 100, 2)
```
In sample error of our model is 2.43%. We estimate the out of sample error of the model to be slightly greater than 2.43%

We now use the model to predict the outcome for our test dataset and compute the prediction accuracy and out of sample error.

```r
cmOut <- confusionMatrix(hraTest$classe, predict(modFit, hraTest))

accOut <- round(cm$overall["Accuracy"] * 100, 2)
errOut <- 100 - acc

cmOut$table
```

```
##           Reference
## Prediction    A    B    C    D    E
##          A 2191   27    4    7    3
##          B   51 1416   45    5    1
##          C    0   43 1303   19    3
##          D    0    2   50 1222   12
##          E    4   14   17   20 1387
```

From the confusion matrix table, we see that the prediction has high accuracy. Concretely, we have an accuracy of 95.83% and an out-of-sample error rate of 4.17%, which is greater than the in sample error we computed eariler.


## Conclusion
We used the Human Activity Recognition dataset to build a classifier to identify the type of barbell lifting. We trained a Stochastic Gradient Boosting model with 60% data using 10-fold cross-validation. We then predicted the outcome for the remaining 40% of the data. Our model has an accuracy of 95.83% with an out-of-sample error rate of 4.17%, which matched our estimate.

## Reference

1. Velloso, E.; Bulling, A.; Gellersen, H.; Ugulino, W.; Fuks, H. Qualitative Activity Recognition of Weight Lifting Exercises. Proceedings of 4th International Conference in Cooperation with SIGCHI (Augmented Human '13) . Stuttgart, Germany: ACM SIGCHI, 2013. 
