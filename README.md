#Multiple Linear Regression
#install and activate the packages below
install.packages("caTools")
library(caTools)
install.packages("forecast")
library(forecast)
install.packages("caret")
library(caret)

#data preprocessing
#already set the work directory
wine.df = read.csv("winequality-red.csv")
#set seed in order to get the same result eacch time we run our code.
set.seed(1)
#partitioning data into training set(%75) and valdation set(%25)
split = sample.split(rownames(wine.df), SplitRatio = 0.75)
train.df = subset(wine.df, split == T)
valid.df = subset(wine.df, split == F)
#Multiple Linear Regression model
#creting the regressor with all the independant variables.
regressor = lm(quality ~ ., data = train.df)
summary(regressor)
#creating the predictor
y_pred = predict(regressor, valid.df)
y_pred
#using stepwise elimination to determine which predictors are the most statistically significant.
regressor2 = step(regressor, direction = "both")
summary(regressor2)
##creting another regressor with the the predictors that are mostly statistically significant.
#run new prediction with regressor2 to see the difference in our prediction
y_pred2 = predict(regressor2, valid.df)
y_pred2
#check the accuracy of the model
accuracy(y_pred, valid.df$quality)
accuracy(y_pred2, valid.df$quality)

########################################################################
#logistic Regression

#data preprocessing
#already set the work directory
wine.df = read.csv("winequality-red.csv")
#binary classification of wine into good qulaity and not good quality(poor).
#ratings 7 and 8 considered as good wines(=1) and ratings below 7
# considered as poor(=0).
wine.df$quality = factor(ifelse(wine.df$quality > 6, 1, 0), levels = c(0,1))
#set seed in order to get the same result eacch time we run our code.
set.seed(1)
#partitioning data into training set(%75) and valdation set(%25)
split = sample.split(rownames(wine.df), SplitRatio = 0.75)
train.df = subset(wine.df, split == T)
valid.df = subset(wine.df, split == F)
#normalizing our independant variable
train.df[, -12] = scale(train.df[, -12])
valid.df[, -12] = scale(valid.df[, -12])
#Creating our generalized linear model classifier
classifier = glm(formula = quality ~ .,
                 family = binomial,
                 data = train.df)
summary(classifier)
#run stepwise elimination to optimize the model
classifier2 = step(classifier, direction = "both")
summary(classifier2)
#classifier 2 has lower AIC, therfor it performs better. I'm gonna use it
#for the prediction.

#predicting the probability of a wine to be considered as good quality.
prob_pred = predict(classifier2, type = "response", newdata = valid.df[-12])
prob_pred
#creating threshhold for our prediction (prob_pred). Probabilities above
# %50 classify as good and below %50 classify as not good.
y_pred = ifelse(prob_pred > 0.50, 1, 0)
y_pred
#making the confusion matrix to evaluate our model
cm = table(valid.df[, 12], y_pred)
cm
confusionMatrix(cm)
#classifier 2 has even more accuracy rate in prediction.
