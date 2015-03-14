# HAR_report
Mendelevium  
Friday, March 13, 2015  

## Background
Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).

## Data
The training data for this project are available here: 
https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

The test data are available here: 
https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv

The data for this project come from this source: http://groupware.les.inf.puc-rio.br/har. If you use the document you create for this class for any purpose please cite them as they have been very generous in allowing their data to be used for this kind of assignment. 

## Setup
Load package and set data path.

```r
library(caret)
```

```
## Warning: package 'caret' was built under R version 3.1.3
```

```
## Loading required package: lattice
## Loading required package: ggplot2
```

```
## Warning: package 'ggplot2' was built under R version 3.1.3
```

```r
library(randomForest)
```

```
## Warning: package 'randomForest' was built under R version 3.1.3
```

```
## randomForest 4.6-10
## Type rfNews() to see new features/changes/bug fixes.
```

```r
library(e1071)
```

```
## Warning: package 'e1071' was built under R version 3.1.3
```

```r
## variables
trainFile <- "./data/ML/pml-training.csv"
testFile <- "./data/ML/pml-testing.csv"
```

## Load data 
Bad entries are also replaced  with "na".

```r
if (!exists("trainData")) {
    trainData <- read.csv(trainFile, header=TRUE, na.strings=c("NA","NaN", " ",""))
}
if (!exists("testData")) {
    testData <- read.csv(testFile, header=TRUE, na.strings=c("NA","NaN", " ",""))
}
```
The training set contains 19622 rows and 160 columns. 
The testing set contains 20 rows and 160 columns. 
The "classe" variable in the training set is the outcome to predict.

## Clean data 
Keep only relevant columns (NAs, timestamp and window columns removed).

```r
keeps <- colSums(is.na(trainData)) == 0
newTrain <- trainData[keeps]
newTest <- testData[keeps]

removes <- grep("timestamp|window", names(newTrain))
newTrain <- newTrain[-removes]
newTest <- newTest[-removes]
```
After cleaning:
The new training set contains 19622 rows and 55 columns. 
The new testing set contains 20 rows and 55 columns. 

## Data partition
Create a data partition: 75% will be use for training and 25% for pre-testing.

```r
set.seed(123)
inTrain <- createDataPartition(y=newTrain$classe, p=0.75, list=FALSE)
train <- newTrain[inTrain, ]
preTest <- newTrain[-inTrain, ]
```

## Cross validation
A 7-fold cross validation is performed to minimize variance and bias.

```r
control <- trainControl(method="cv", 7)
```

## Trainning
A random forest algorithm is selected for its classification and regression capabilities. The number of trees is limited to 100 in order to reduce the computing time required since RF is not the most efficient method.

```r
fit <- train(classe ~ . - X, data=train, method="rf", trControl=control, ntree=100)
fit
```

```
## Random Forest 
## 
## 14718 samples
##    54 predictor
##     5 classes: 'A', 'B', 'C', 'D', 'E' 
## 
## No pre-processing
## Resampling: Cross-Validated (7 fold) 
## 
## Summary of sample sizes: 12615, 12617, 12615, 12616, 12615, 12614, ... 
## 
## Resampling results across tuning parameters:
## 
##   mtry  Accuracy   Kappa      Accuracy SD  Kappa SD   
##    2    0.9910318  0.9886548  0.002870744  0.003631326
##   29    0.9919828  0.9898583  0.002999674  0.003794938
##   57    0.9876338  0.9843559  0.004065217  0.005144408
## 
## Accuracy was used to select the optimal model using  the largest value.
## The final value used for the model was mtry = 29.
```

## Prediction on pre-test set

```r
predTest <- predict(fit, newdata = preTest)
confmat <- confusionMatrix(preTest$classe, predTest)
confmat
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1394    1    0    0    0
##          B    5  942    2    0    0
##          C    0    8  843    4    0
##          D    0    0    9  794    1
##          E    0    0    0    0  901
## 
## Overall Statistics
##                                           
##                Accuracy : 0.9939          
##                  95% CI : (0.9913, 0.9959)
##     No Information Rate : 0.2853          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.9923          
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9964   0.9905   0.9871   0.9950   0.9989
## Specificity            0.9997   0.9982   0.9970   0.9976   1.0000
## Pos Pred Value         0.9993   0.9926   0.9860   0.9876   1.0000
## Neg Pred Value         0.9986   0.9977   0.9973   0.9990   0.9998
## Prevalence             0.2853   0.1939   0.1741   0.1627   0.1839
## Detection Rate         0.2843   0.1921   0.1719   0.1619   0.1837
## Detection Prevalence   0.2845   0.1935   0.1743   0.1639   0.1837
## Balanced Accuracy      0.9981   0.9944   0.9921   0.9963   0.9994
```
The estimated accuracy of the model is 99.39% and the error is 0.61%.

## Prediction on test set

```r
pred <- predict(fit, newdata = newTest)
pred
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```
