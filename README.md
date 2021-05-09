### Background

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now
possible to collect a large amount of data about personal activity
relatively inexpensively. These type of devices are part of the
quantified self movement - a group of enthusiasts who take measurements
about themselves regularly to improve their health, to find patterns in
their behavior, or because they are tech geeks. One thing that people
regularly do is quantify how much of a particular activity they do, but
they rarely quantify how well they do it. In this project, your goal
will be to use data from accelerometers on the belt, forearm, arm, and
dumbell of 6 participants. They were asked to perform barbell lifts
correctly and incorrectly in 5 different ways. More information is
available from the website here:
<http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har>
(see the section on the Weight Lifting Exercise Dataset).

### Data

The training data for this project are available here:
<https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv>

The test data are available here:
<https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv>

The data for this project come from this source:
<http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har>.

Data Preparation
----------------

### Importing Packages

This loads the necessary R packages for this project. Caret for
classification training and models, mainly for Decision Tree,
randomForest for Random Forest model, ggplot2 for exploratory analysis,
and rattle for tree visualization.

    ## Loading required package: lattice

    ## Loading required package: ggplot2

    ## Loading required package: tibble

    ## Loading required package: bitops

    ## Rattle: A free graphical interface for data science with R.
    ## Version 5.4.0 Copyright (c) 2006-2020 Togaware Pty Ltd.
    ## Type 'rattle()' to shake, rattle, and roll your data.

    ## randomForest 4.6-14

    ## Type rfNews() to see new features/changes/bug fixes.

    ## 
    ## Attaching package: 'randomForest'

    ## The following object is masked from 'package:rattle':
    ## 
    ##     importance

    ## The following object is masked from 'package:ggplot2':
    ## 
    ##     margin

### Downloading Data

The training and testing data required for this project are downloaded
in this section.

    trainingFile <- "pml-training.csv"
    testingFile <- "pml-testing.csv"

    download.file("http://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv", trainingFile)
    download.file("http://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv", testingFile)

### Data Preprocessing

Preprocessing is used to clean-up the raw data. All N/A and empty values
are omitted.

Subsets will be created to remove unnecessary columns that might affect
the prediction of the models. These columns are user\_name,
raw\_timestamp\_part\_1, raw\_timestamp\_part\_2, cvtd\_timestamp,
new\_window, and num\_window.

    training <- read.csv(trainingFile, na.strings=c("NA","#DIV/0!", ""))
    testing <- read.csv(testingFile, na.strings=c("NA", "#DIV/0!", ""))

    training <- training[,colSums(is.na(training)) == 0]
    testing <- testing[,colSums(is.na(testing)) == 0]

    training <- training[,-c(1:7)]
    testing <- testing[,-c(1:7)]

### Train-Test Split

Because pml-testing.csv will be used to test the accuracy of the models,
pml-training.csv will serve as training and testing sets. Because of
this, it is separated into 0.70 (70%) training and 0.30 (30%) testing
data.

    splittedData <- createDataPartition(y=training$classe, p=0.7, list=FALSE)
    sTraining <- training[splittedData,] 
    sTesting <- training[-splittedData,]

### Exploratory Analysis

The data is composed of five (5) classifications. The bar plot shows the
distribution of uniquely identified data inside the classe column.

    qplot(classe, main="Distribution of classe variable", xlab="classe values", ylab="Frequency", data=sTraining)

![](Exercise-Activity-Prediction_files/figure-markdown_strict/unnamed-chunk-5-1.png)

Prediction Models
-----------------

### Decision Tree

Decision Tree algorithm will be used to predict the outcome of the data.

    DT_model <- train(classe ~ ., data=sTraining, method="rpart")

    DT_model

    ## CART 
    ## 
    ## 13737 samples
    ##    52 predictor
    ##     5 classes: 'A', 'B', 'C', 'D', 'E' 
    ## 
    ## No pre-processing
    ## Resampling: Bootstrapped (25 reps) 
    ## Summary of sample sizes: 13737, 13737, 13737, 13737, 13737, 13737, ... 
    ## Resampling results across tuning parameters:
    ## 
    ##   cp          Accuracy   Kappa     
    ##   0.03275353  0.5233400  0.38136492
    ##   0.06198081  0.3977403  0.17822522
    ##   0.11616316  0.3233932  0.05982214
    ## 
    ## Accuracy was used to select the optimal model using the largest value.
    ## The final value used for the model was cp = 0.03275353.

    DT_model$finalModel

    ## n= 13737 
    ## 
    ## node), split, n, loss, yval, (yprob)
    ##       * denotes terminal node
    ## 
    ##  1) root 13737 9831 A (0.28 0.19 0.17 0.16 0.18)  
    ##    2) roll_belt< 130.5 12571 8677 A (0.31 0.21 0.19 0.18 0.11)  
    ##      4) pitch_forearm< -33.55 1128   12 A (0.99 0.011 0 0 0) *
    ##      5) pitch_forearm>=-33.55 11443 8665 A (0.24 0.23 0.21 0.2 0.12)  
    ##       10) magnet_dumbbell_y< 426.5 9521 6828 A (0.28 0.18 0.24 0.19 0.11)  
    ##         20) roll_forearm< 123.5 5917 3489 A (0.41 0.18 0.18 0.17 0.057) *
    ##         21) roll_forearm>=123.5 3604 2381 C (0.074 0.17 0.34 0.23 0.19) *
    ##       11) magnet_dumbbell_y>=426.5 1922  967 B (0.044 0.5 0.045 0.23 0.19) *
    ##    3) roll_belt>=130.5 1166   12 E (0.01 0 0 0 0.99) *

    fancyRpartPlot(DT_model$finalModel)

![](Exercise-Activity-Prediction_files/figure-markdown_strict/unnamed-chunk-8-1.png)

    pred_DT <- predict(DT_model, newdata=sTesting, typce="class")

### Confusion Matrix (Decision Tree)

The following confusion matrix shows the errors and overall statistics
of the prediction model.

    confusionMatrix(pred_DT, as.factor(sTesting$classe))

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction    A    B    C    D    E
    ##          A 1506  459  483  427  162
    ##          B   38  393   42  178  173
    ##          C  128  287  501  359  270
    ##          D    0    0    0    0    0
    ##          E    2    0    0    0  477
    ## 
    ## Overall Statistics
    ##                                          
    ##                Accuracy : 0.4889         
    ##                  95% CI : (0.476, 0.5017)
    ##     No Information Rate : 0.2845         
    ##     P-Value [Acc > NIR] : < 2.2e-16      
    ##                                          
    ##                   Kappa : 0.3322         
    ##                                          
    ##  Mcnemar's Test P-Value : NA             
    ## 
    ## Statistics by Class:
    ## 
    ##                      Class: A Class: B Class: C Class: D Class: E
    ## Sensitivity            0.8996  0.34504  0.48830   0.0000  0.44085
    ## Specificity            0.6364  0.90919  0.78514   1.0000  0.99958
    ## Pos Pred Value         0.4959  0.47694  0.32427      NaN  0.99582
    ## Neg Pred Value         0.9410  0.85260  0.87903   0.8362  0.88809
    ## Prevalence             0.2845  0.19354  0.17434   0.1638  0.18386
    ## Detection Rate         0.2559  0.06678  0.08513   0.0000  0.08105
    ## Detection Prevalence   0.5161  0.14002  0.26253   0.0000  0.08139
    ## Balanced Accuracy      0.7680  0.62711  0.63672   0.5000  0.72022

### Random Forest

Random Forest algorithm will be used to predict the outcome of the data.

    RF_model <- randomForest(as.factor(classe) ~ ., data=sTraining, method="class")

    RF_model

    ## 
    ## Call:
    ##  randomForest(formula = as.factor(classe) ~ ., data = sTraining,      method = "class") 
    ##                Type of random forest: classification
    ##                      Number of trees: 500
    ## No. of variables tried at each split: 7
    ## 
    ##         OOB estimate of  error rate: 0.55%
    ## Confusion matrix:
    ##      A    B    C    D    E  class.error
    ## A 3904    2    0    0    0 0.0005120328
    ## B   16 2635    7    0    0 0.0086531226
    ## C    0   15 2378    3    0 0.0075125209
    ## D    0    0   26 2224    2 0.0124333925
    ## E    0    0    1    4 2520 0.0019801980

    pred_RF <- predict(RF_model, newdata=sTesting, type="class")

### Confusion Matrix (Random Forest)

The following confusion matrix shows the errors and overall statistics
of the prediction model.

    confusionMatrix(pred_RF, as.factor(sTesting$classe))

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction    A    B    C    D    E
    ##          A 1670    3    0    0    0
    ##          B    3 1136    5    0    0
    ##          C    0    0 1019   13    0
    ##          D    0    0    2  951    7
    ##          E    1    0    0    0 1075
    ## 
    ## Overall Statistics
    ##                                          
    ##                Accuracy : 0.9942         
    ##                  95% CI : (0.9919, 0.996)
    ##     No Information Rate : 0.2845         
    ##     P-Value [Acc > NIR] : < 2.2e-16      
    ##                                          
    ##                   Kappa : 0.9927         
    ##                                          
    ##  Mcnemar's Test P-Value : NA             
    ## 
    ## Statistics by Class:
    ## 
    ##                      Class: A Class: B Class: C Class: D Class: E
    ## Sensitivity            0.9976   0.9974   0.9932   0.9865   0.9935
    ## Specificity            0.9993   0.9983   0.9973   0.9982   0.9998
    ## Pos Pred Value         0.9982   0.9930   0.9874   0.9906   0.9991
    ## Neg Pred Value         0.9991   0.9994   0.9986   0.9974   0.9985
    ## Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839
    ## Detection Rate         0.2838   0.1930   0.1732   0.1616   0.1827
    ## Detection Prevalence   0.2843   0.1944   0.1754   0.1631   0.1828
    ## Balanced Accuracy      0.9984   0.9978   0.9953   0.9923   0.9967

Conclusion
----------

### Result

Considering the confusion matrices and the results of both Decision Tree
and Random Forest models, the latter produced an accuracy of 0.993 (95%
CI of 0.9906, 0.995) compared to 0.4989 (95% CI of 0.486, 0.5118) of the
former. For this project, the Random Forest algorithm is chosen.

The prediction for the pml-testing.csv will be as follows:

    prediction <- predict(RF_model, newdata=testing, type="class")

    prediction

    ##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
    ##  B  A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
    ## Levels: A B C D E

### Out-of-sample Error

Out-of-sample error is a measure of how accurately an algorithm is able
to predict outcome values for previously unseen data. This is calculated
as 1 - accuracy of the prediction model. Unlike the Decision Tree which
produced a not-so-good accuracy for this project, the Random Forest did
well with an accuracy of 0.993 which resulted to a minimal out-of-sample
error. This means that for every prediction, there is a below 1% chance
of the data getting misclassified.
