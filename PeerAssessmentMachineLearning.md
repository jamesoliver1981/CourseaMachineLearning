---
title: "Prediction of Sport Activity through Wearble Devices"
author: "James Oliver"
date: "17 November 2015"
output: html_document
---

##Introduction / Executive Summary  
This report details how data taken from the Human Activity Study carried out by Groupware.  Details of the study can be found [here](http://groupware.les.inf.puc-rio.br/har).  This study measured people carrying out 6 activities.  
The purpose of this report and model is to predict which of the activities are being carried out given a set a variables.
This report details how the data was cleaned, the method used to predict the outcomes and the quality of the outcome, including the out of sample error rate.  

##How the model was built  
Include sources of data
Validations
This report will include all code so that the results are reproducible.  
The data is provided as a training and test dataset.  
The test dataset has only 20 variables which is used to test the accuracy of the final model.  
To avoid confusion, as a test dataset is required to test the model before it is applied to the final 20 observations, these 20 observations are referred from now as the final dataset - named as fin and fin2 in the code.

Examaning the data through the review of summary, identified that whilst there are 160 variables, many of these have high majorities (more than 95%) blank or NA values.  
Therefore a function was used to identify those with high data completeness and used to restrict the data appropriately.


```r
setwd("~/Rworkingdir/MachineLearning")
fileurl<-"https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"

download.file(fileurl,destfile = "~/Rworkingdir/machinelearning/training.csv")
train<- read.csv("training2.csv")

fileurl<- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
download.file(fileurl,destfile = "testing.csv")
fin<- read.csv("pml_testing.csv")

getFractionMissing <- function(df) {
        colCount <- ncol(df)
        returnDf <- data.frame(index=1:ncol(df),
                               columnName=rep("undefined", colCount),
                               FractionMissing=rep(-1, colCount),
                               stringsAsFactors=FALSE)
        for(i in 1:colCount) {
                colVector <- df[,i]
                missingCount <- length(which(colVector == "") * 1)
                missingCount <- missingCount + sum(is.na(colVector) * 1)
                returnDf$columnName[i] <- as.character(names(df)[i])
                returnDf$FractionMissing[i] <- missingCount / length(colVector)
        }
        
        assess<<- returnDf
}
getFractionMissing(train)
assess<-subset(assess,assess$FractionMissing==0)
dim(assess)
```

```
## [1] 60  3
```

```r
restrict<- as.character(assess$columnName)


train2<- train[restrict]
```

Reviewing the data at this point, shows that there are still a number of variables which will not relate to the predict accuracy of the model.  Specifically these are the time related variables.  These are subsequently dropped.


```r
library(dplyr)
train2<-select(train2,-(raw_timestamp_part_1:num_window))
```
The data now appears to be reasonable clean and therefore can be prepared for training and testing.


```r
library(caret)
train3<-train2
intrain<-createDataPartition(y=train3$classe,p=0.7,list=FALSE)
train2<-train3[intrain,]
test1<-train3[-intrain,]
```

As an initial effort, a random forest classification model is created.  
The output of the model is below.


```r
library(randomForest)
set.seed(1234)
mod3<- randomForest(classe~.-X,data=train2,type=classification,mtry=10)
mod3
```

```
## 
## Call:
##  randomForest(formula = classe ~ . - X, data = train2, type = classification,      mtry = 10) 
##                Type of random forest: classification
##                      Number of trees: 500
## No. of variables tried at each split: 10
## 
##         OOB estimate of  error rate: 0.54%
## Confusion matrix:
##      A    B    C    D    E class.error
## A 3901    3    0    0    2 0.001280082
## B   11 2641    6    0    0 0.006395786
## C    0   10 2380    6    0 0.006677796
## D    0    0   25 2225    2 0.011989343
## E    0    0    2    7 2516 0.003564356
```

The bottom part of this shows the predicted out of sample error rate as 0.55%.  
This can be seen as those not being classified in the correct bucket.  
To revalidate this, the model is tested against test1, a subset of the training dataset.


```r
pred<-predict(mod3,test1)
test1$predRight<- pred==test1$classe
table(pred,test1$classe)
```

```
##     
## pred    A    B    C    D    E
##    A 1674    1    0    0    0
##    B    0 1138    0    0    0
##    C    0    0 1025    4    0
##    D    0    0    1  959    2
##    E    0    0    0    1 1080
```

```r
15/nrow(test1)*100
```

```
## [1] 0.2548853
```
This shows that 15 of the 5885 variables are incorrectl classified.  This gives a real out of sample error rate of 0.26%.

Other models were tried however this was the most successful as it used the bootstrap method.  
The bootstrap method takes random sampling of the data with replacement.
This can lead to an underestimation of the error. However in this case, it appears to have overestimated the error.  

To further validate the answer, a model with random forest with cross validation and another boosted trees model with cross validation was used.  These models were tested against the final dataset, with all 3 generating the same answers.
To resrict run time these are not run but are shown below  

###Random Forest with Cross Valiation rather than Bootsrap
set.seed(4321)
modfit<-train(classe~.-X,method="rf",data=train2,trControl=
                      trainControl(method="cv",number = 3, 
                                   allowParallel = T))


###Boosted Trees
set.seed(4444)
mod4<- train(classe~.-X,method="gbm",data=train2, 
             trControl=trainControl(method="cv",number =3,allowParallel = T))





##How Cross Validation Was Used  
REview notes to see what is said about cross validation - note it's in the model... ELaborate in that a bit more

##Review of Model  
Include here the expected out of sample error rate is


```r
summary(cars)
```

```
##      speed           dist       
##  Min.   : 4.0   Min.   :  2.00  
##  1st Qu.:12.0   1st Qu.: 26.00  
##  Median :15.0   Median : 36.00  
##  Mean   :15.4   Mean   : 42.98  
##  3rd Qu.:19.0   3rd Qu.: 56.00  
##  Max.   :25.0   Max.   :120.00
```

You can also embed plots, for example:

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png) 

Note that the `echo = FALSE` parameter was added to the code chunk to prevent printing of the R code that generated the plot.
