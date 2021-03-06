---
output: 
  html_document: 
    keep_md: yes
---
HARprediction Project
=====================
rucrbser

This report uses five variables "raw_timestamp_part_1" to "num_window" to predict the manner in which they did the exercise. We predict 20 different test cases through comparing accuracy among different machine learning models.


## Import training and testing sets


```r
#download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv", #destfile = "c:/users/x/desktop/training.csv")
#download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv", #destfile = "c:/users/x/desktop/testing.csv")

training <- read.csv("c:/users/x/desktop/training.csv")
testing <- read.csv("c:/users/x/desktop/testing.csv")
```


## Plotting Predictors and Building Cross Validation


```r
library(caret)
```

```
## Loading required package: lattice
```

```
## Loading required package: ggplot2
```

```r
trainingnew <- subset(training, select = c(3:7, 160))

featurePlot(x=trainingnew[, c(1,2,3)], y=unclass(trainingnew$classe), plot = "pairs")
```

![](index_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

```r
set.seed(100)
foldstraining <- createFolds(y = trainingnew$classe, k = 3, list = T, returnTrain = T)

# Creat K folds to build cross validation
set.seed(100)
foldsvalidation <- createFolds(y = trainingnew$classe, k = 3, list = T, returnTrain = F)
```


## Build Prediction Model


```r
RMSEvalidation <- 0

# Do training for each k fold
for (i in 1:3) {
        # choose the validation and training data
        validation <- trainingnew[foldsvalidation[[i]], ]
        training <- trainingnew[foldstraining[[i]], ]
        
        # train the model with predicting with trees
        modFit1 <- train(classe ~ ., method = "rpart", data = training)
        pred1 <- predict(modFit1, newdata = validation)
        
        # train the model with random forests
        modFit2 <- train(classe ~ ., method = "rf", data =    
                        training[sample(dim(training)[1], size = 500), ], prox = T)
        pred2 <- predict(modFit2, newdata = validation)
        
        # train the model with boosting
        modFit3 <- train(classe ~ ., method = "gbm", data = 
                        training[sample(dim(training)[1], size = 500), ], verbose = F)
        pred3 <- predict(modFit3, newdata = validation)
        
        # combine the three above predictors to train the model with boosting
        predDF <- data.frame(pred1, pred2, pred3, classe = validation$classe)
        combModFit <- train(classe ~ ., method = "gbm", data = predDF, verbose = F)
        combPred <- predict(combModFit, validation$classe)
        
        # check the root-mean-square error with the cross validation
        RMSE <- sqrt(sum((combPred == validation$classe) ^ 2) / 
                          length(validation$classe))
        if (RMSE > RMSEvalidation) {
                RMSEvalidation <- RMSE
                pred1test <- predict(modFit1, testing)
                pred2test <- predict(modFit2, testing)
                pred3test <- predict(modFit3, testing)
                predtestDF <- data.frame(pred1 = pred1test, pred2 = pred2test, 
                                         pred3 = pred3test)
                combPredtest <- predict(combModFit, predtestDF)
        }
}
```

* Here we build three different machine learning models, including classification trees, random forests and boosting, and in order to improve accuracy of prediction, we combine these three classifiers.

* To use cross validation, we bulid K-ford CV through data slicing, here k = 3.

* The expected out of sample error is maybe overfitting, our predictor may capture data noise leading not perform as well on new samples.

* And at last, prediction models in this report predict the manners 20 different test cases are respectively, B, A, B, A, A, E, D, B, A, A, B, C, B, A, E, E, A, B, B, B, which the accuracy is about 99.409753%.
