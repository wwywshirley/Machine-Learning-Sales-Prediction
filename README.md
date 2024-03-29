# Black-Friday-Sales
This study examines shopping that is done on “Black Friday,” routinely the busiest shopping day of the year in the United States.  The primary research question was to predict and describe who were the biggest spenders during this shopping event.  The purpose is to utilize these findings to better inform and focus retail marketing efforts.  It is significant because for many retailers this shopping day can mean the difference between a profit and a loss for the fiscal quarter, and sometimes the fiscal year.  The biggest spenders during “Black Friday” are generally  males, between the ages of 26 to 35.  They are most likely to spend over $2000- on specific product classes.  Specific product identification numbers are good predictors of who will spend the most.  It is interesting to note that the biggest spenders were younger males, which is contrary to popular U.S. stereotypes.  
Our dataset is comprised of sales transactions captured at a retail store on “Black Fridays.”  It has 537,577 rows and 12 columns. predict purchase amount using the data features, thus using the dataset to form a regression problem. Customers will be split into two classes (i.e. the Big Spenders and Small Buyers; aka High/Low spending classes).  Customer behavior is explained from this examination of  multiple shopping experiences.  


## Basic Data Exploration
### Data Structure
The data source is named the Black Friday data set and it was obtained from Kaggle.com and originally utilized as part of a competition on the Analytics Vidhya website. It is a sample of the transactions made in a retail store. The data was utilized to better understand customer purchase behavior against different products. The various algorithms, with an emphasis on regression,  were used to predict the dependent variable (the amount of purchase) with the help of the information contained in the other variables. Classifications were also accomplished with this  dataset since several variables are categorical. The approaches were "Predicting the age of the consumer" and "Predict the category of goods bought.” This dataset is amenable to clustering so different clusters were determined.
```
df_unique_Users <- df %>% group_by(User_ID) %>% 
  summarise(Gender = Gender[1], 
            Age = Age[1], 
            Occupation = Occupation[1], 
            City_Category = City_Category[1], 
            Stay_In_Current_City_Years = Stay_In_Current_City_Years[1], 
            Marital_Status = Marital_Status[1], 
            Purchase = sum(Purchase), 
            Items_Count = n())
#View Data
dim(df)s
dim(df_unique_Users) #Number of Unique Users = 5891   Number of columns = 9

head(df, 20) 
head(df_unique_Users, 20)

str(df) 
str(df_unique_Users)

plot_str(df) # To create a Network Graph for our data set
plot_str(df_unique_Users)
```
![](images/1.png)

## Data Pre-prosessing
### Missing Value
```
sapply(df, function(x) sum(is.na(x))) 
plot_missing(df)
```
![](images/000003.png)

```
#For Product_Category_2
fix_PC2<-rpart(Product_Category_2 ~ User_ID+Product_ID+Age+Gender, data=df[!is.na(df$Product_Category_2),], method="anova")
df$Product_Category_2[is.na(df$Product_Category_2)] <- predict(fix_PC2, df[is.na(df$Product_Category_2),])

#For Product_Category_3
fix_PC3<-rpart(Product_Category_3 ~ User_ID+Product_ID+Age+Gender, data=df[!is.na(df$Product_Category_3),], method="anova")
df$Product_Category_3[is.na(df$Product_Category_3)] <- predict(fix_PC3, df[is.na(df$Product_Category_3),])

head(df,10)
summary(df)
dim(df) #After pre-processing, our data set still has 537577 rows and 12 columns. i.e. no data loss
plot_missing(df) #Once again, check for missing values
```
![](images/000004.png)
### Type Conversions
```
df$User_ID <- as.factor(df$User_ID)
df$Product_ID <- as.factor(df$Product_ID)
df$Gender <- as.factor(if_else(df$Gender == 'M', 'Male', 'Female'))
df$Age <- as.factor(df$Age)
df$Occupation <- as.factor(df$Occupation)
df$City_Category <- as.factor(df$City_Category)
df$Stay_In_Current_City_Years <- as.factor(df$Stay_In_Current_City_Years)
df$Marital_Status <- as.factor(if_else(df$Marital_Status == 1, 'Married', 'Single'))
df$Product_Category_1 <- as.integer(df$Product_Category_1)
df$Product_Category_2 <- as.integer(df$Product_Category_2)
df$Product_Category_3 <- as.integer(df$Product_Category_3)
df$Purchase <- as.numeric(df$Purchase)
```
### Checking Outliners
```
#### Do some Univariate Analysis
stat_function = function(x){
    if(class(x)=="integer"|class(x)=="numeric"){
        var_type = class(x)
        length = length(x)
        miss_val = sum(is.na(x))
        mean = mean(x,na.rm = T)
        std = sd(x,na.rm = T)
        var = var(x,na.rm = T)
        cv = std/mean
        min = min(x)
        max = max(x,na.rm = T)
        pct = quantile(x,na.rm = T,p=c(0.75,0.85,0.90,0.95,0.99,1.0))
        return(c(var_type=var_type,length=length,miss_val=miss_val,mean=mean,std=std,var=var,cv=cv,min=min,max=max,pct=pct))
        }
}
#####Name of a numeric variable and categorical (factor)variable
num_var = names(df)[sapply(df,is.numeric)]
cat_var = names(df)[!sapply(df,is.numeric)]
###
#
mystat = apply(df[num_var],2,stat_function)
t(mystat)
###Checking for Outliers
options(scipen = 9999)
boxplot(df[num_var],horizontal = T,col = rainbow(1:10))
```
![](images/0000003.png)
## Data Distribution
```
prop.table(table(df$Marital_Status))  #Married? Unmarried?
prop.table(table(df$Gender)) #Male vs Female?
prop.table(table(df$Age)) #Age group?
prop.table(table(df$City_Category)) #City_Category with the highest customers?
prop.table(table(df$Stay_In_Current_City_Years)) #Does duration of city stay play a role?

#Histogram showing Distributions of all Continuous Features
hist(df$Purchase, 
     main="Distribution of Purchase",
    xlab="Amount ($)",
    col="darkmagenta",
    )
hist(df$Occupation, 
     main="Distribution of Occupation",
    xlab="Occupation ID",
    col="blue",
    )
hist(df$Product_Category_1, 
     main="Distribution of Product_Category_1",
    xlab="Product_Category_1",
    col="green",
    )
hist(df$Product_Category_2, 
     main="Distribution of Product_Category_2",
    xlab="Product_Category_2",
    col="red",
    )
hist(df$Product_Category_3, 
     main="Distribution of Product_Category_3",
    xlab="Product_Category_3",
    col="orange",
    )
hist(df$User_ID, 
     main="Distribution of User_ID",
    xlab="User_ID",
    col="chocolate",
    )
              
#Bar graphs showing Distributions of all Descrete Features
plot_bar(df, ) #Bar graphs showing Distributions of all Descrete Features
```
![](images/4.png)![](images/5.png)![](images/6.png)![](images/7.png)![](images/8.png)![](images/9.png)![](images/00000f.png)

#
## Linear Regression
Started from simple linear regression to do the prediction. In the model, we have set unless dependent variables null and selected variables like gender, age, occupation, years in current city, marital status and items count. The adjusted R-squared value is 0.6915.

```
##Dropping dependent variable values to be predicted
df1<-df_unique_Users
df1$User_ID <- NULL
df1$Product_ID <- NULL

##Dividing data into training and testing
sample = sample(1:nrow(df1),size = floor(nrow(df1)*0.7))
train = df1[sample,]
test = df1[-sample,]
##Start to build the model
lm_fit = lm(log(Purchase)~., data = train)
summary(lm_fit)
plot(lm_fit)
###
pred <- predict(lm_fit, test, interval="confidence")
actual <- log(test$Purchase) 
rmse <- (mean((pred - actual)^2))^0.5
rmse
#AIC(pred)
#confusionMatrix(log(test$Purchase), actual, pred, dnn=list('actual','predicted'))# estimate = `Linear regression`)
#table(actual, pred, dnn=list('actual','predicted'))

#pred
plot(pred)
```
![](images/w.png)![](images/ww.png)![](images/www.png)![](images/wwww.png)

We plotted the residuals vs. fitted values as a useful way to check linearity and homoscedasticity. There are some large negative residual values at bottom right corner of the plot which means that these values do not meet the linear model assumption. To assess the assumption of linearity, we hope to ensure that the residuals are between -2 and 2. And we deleted those values not located in this range. 
The residuals vs. leverage plots also helps us to identify the influential data in our model. The 
points we are not looking for are those in the upper right or lower right. From the plot, we can see some outliers in the lower right corner which tend to have a critical impact on our model, so we remove these values to reach unbiased results. 

## Multiple Linear Regression
### Correlation Matrix
```
corMatrix$var1 = rownames(corMatrix)
corMatrix %>%
    gather(key = var2, value = r, 1:7) %>%
        ggplot(aes(x = var1, y = var2, fill = r)) +
        geom_tile() +
        geom_text(aes(label = round(r, 2)), size = 3)+
        scale_fill_gradient2(low = '#00a6c8', high='#eb3300', mid = 'white') +
        coord_equal()  +
        theme(axis.text.x = element_text(angle = 90, hjust = 1))
```
![](images/matrix.png)

### Estimate Model Using Train Data
```
model <- lm(Purchase ~ Gender + Age + Occupation + Marital_Status + City_Category + Stay_In_Current_City_Years, data = trainInt, method = 'anova')
equation <- noquote(paste('Purchase =',
                            round(model$coefficients[1],0), '+',
                            round(model$coefficients[2],0), '* Gender', '+',
                            round(model$coefficients[3],0), '* Age', '+',
                            round(model$coefficients[4],0), '* Occupation', '+',
                            round(model$coefficients[5],0), '* Marital_Status', '+',
                            round(model$coefficients[6],0), '* City_Category', '+',
                            round(model$coefficients[7],0), '* Stay_In_Current_City_Years'))
equation
summary(model)
anova(model)
```
Coefficients:</br>
                           Estimate Std. Error t value Pr(>|t|)   </br> 
(Intercept)                6999.143     47.714 146.689  < 2e-16 </br>
Gender                      676.445     18.915  35.763  < 2e-16 </br>
Age                          36.737      6.358   5.778 7.55e-09 </br>
Occupation                    9.917      1.254   7.905 2.68e-15 </br>
Marital_Status              -55.853     17.321  -3.225  0.00126  </br>
City_Category               448.607     10.737  41.780  < 2e-16 </br>
Stay_In_Current_City_Years   14.695      6.274   2.342  0.01916   </br>
Signif. codes:  0 ‘’ 0.001 ‘’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1</br>
Residual standard error: 4962 on 376296 degrees of freedom</br>
Multiple R-squared:  0.008745,	Adjusted R-squared:  0.008729 </br>
F-statistic: 553.3 on 6 and 376296 DF,  p-value: < 2.2e-16</br>

### Make Predictions Based on Train Data
```
predTrain <- predict(model)
sseTrain <- sum((predTrain - trainInt$Purchase) ^ 2)
sstTrain <- sum((mean(train$Purchase) - trainInt$Purchase) ^ 2)
print ("Model R2 (Train Data)")
modelR2Train <- 1 - sseTrain/sstTrain
modelR2Train
print ("Model RMSE (Train Data)")
rmseTrain <- sqrt(mean((predTrain - trainInt$Purchase) ^ 2))
rmseTrain
```
R2:0.00874475362766736</br>
RMSE:4961.7446993561

### Make Preductions Based on Test Data

```
predTest <- predict(model, newdata = testInt)
sseTest <- sum((predTest - testInt$Purchase) ^ 2)
sstTest <- sum((mean(test$Purchase) - testInt$Purchase) ^ 2)
print ("Model R2 (Test Data)")
modelR2Test <- 1 - sseTest/sstTest
modelR2Test
print ("Model RMSE (Test Data)")
rmseTest <- sqrt(mean((predTest - testInt$Purchase) ^ 2))
rmseTest
```
R2:0.00818251081229127</br>
RMSE:4954.61021817518

## Boosting Tree
```
formula<-as.formula(Purchase ~.)

# mape for RF
mape<-function(real,predicted){
  return(round(mean(abs((real-predicted)/real)), 3))
}

# Mape for xgb
mape_xgb_fun<- function(preds, dtrain){
   library(Metrics)
   labels <- getinfo(dtrain, "label")
   my_mape <- Metrics::mape(labels, preds)
   return(list(metric = "mape", value = round(my_mape,3)))
}
```
```
matrix_xgb = model.matrix(formula, train[, -c("User_ID","Product_ID")])

best_grid <- expand.grid(subsample =  0.5, 
                        colsample_bytree = 0.75,
                        eta = 0.01, 
                        max_depth =  8)  

xgb_best<-xgboost(booster='gbtree',
               data=matrix_xgb,
               label=train$Purchase,
               params = best_grid,
               nrounds = 1000,
               feval  = mape_xgb_fun, # evaluation metrics
               maximize = FALSE, # indicating we want mape to be minimized
               objective='reg:linear',
               early_stopping_rounds = 10,
               verbose = FALSE)
print(xgb_best)
```
No. of features: 91 </br>
niter: 258</br>
best_iteration : 248 </br>
best_ntreelimit : 248 </br>
best_score : 0.264 </br>
nfeatures : 91 </br>
evaluation_log:
iter train_mape</br>
        1      0.989</br>
        2      0.978</br>
      ---         </br>
      257      0.264</br>
      258      0.264
```
test_xgb_remain<-predict(xgb_best, newdata = model.matrix(formula, remain_train[, -c("User_ID","Product_ID")]))
mape_xgb_remain <- mape(real=remain_train$Purchase, predicted = test_xgb_remain)
mape_xgb_remain
```
0.271

## Random Forest
Permutation : Gini Importance or Mean Decrease in Impurity (MDI) calculates each feature importance as the sum over the number of splits (accross all tress) that include the feature, proportionaly to the number of samples it splits.

Permutation Importance or Mean Decrease in Accuracy (MDA) is assessed for each feature by removing the association between that feature and the target. This is achieved by randomly permuting the values of the feature and measuring the resulting increase in error. The influence of the correlated features is also removed.

The best performing Random Forest has been the following:
```
grid <- c(6)
i <- 1
eval <- c()
for (i in 1:length(grid)){

  name <- paste("rf_",i)
  name <- gsub(" ","",name)
  print(name)
  rf <- ranger(formula, train[, -c("User_ID","Product_ID")], mtry = grid[i], verbose=TRUE, seed =42, num.trees = 1000, importance = "permutation") 
  rf
  print(rf)
  
  test_rf_remain <- predict(rf, data = remain_train)$predictions
  mape_rf_remain <- mape(real = remain_train$Purchase, predicted = test_rf_remain)
  print("The MAPE of the remaining training dataset test is: ")
  print(mape_rf_remain)
  
  ## To store the different mapes with the different grids 
  eval <- rbind(eval,name, mape_rf_remain)
  print(eval)
}
```
Type:                             Regression </br>
Number of trees:                  1000 </br>
Sample size:                      120954 </br>
Number of independent variables:  16 </br>
Mtry:                             6 </br>
Target node size:                 5 </br>
Variable importance mode:         permutation </br>
Splitrule:                        variance </br>
OOB prediction error (MSE):       6350550 </br>
R squared (OOB):                  0.7430875 </br>
Predicting.. Progress: 40%. Estimated remaining time: 46 seconds.</br>
Predicting.. Progress: 81%. Estimated remaining time: 14 seconds.</br>
Aggregating predictions.. Progress: 73%. Estimated remaining time: 11 seconds.</br>
[1] "The MAPE of the remaining training dataset test is: "</br>
[1] 0.272</br>
                [,1]   </br>
 name           "rf_1" </br>
 mape_rf_remain "0.272"
