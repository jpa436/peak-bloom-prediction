Cherry Blossom prediction: Model comparison
================

<!--   rmarkdown::github_document
    code_folding: hide
    code_download: true -->

In this section, we will illustrate how we delivered our final model. We
considered linear regression without interaction, linear regression
contains interactions, Lasso, ridge, and PCA regression.

## 1. Linear model : Base model

First model we considered is linear regression without interaction
terms. We scored model by separating training set and test set. After
fitting the model with training set, we compared the test set with the
predicteda value. We used RMSE to compare the performance of the models.

The table below presents the result, the RMSE of linear base model was
around 7.

``` r
library(caret)
library(tidyverse)
set.seed(003)
# Define base linear model 

# Cross validation
df.base<-cherry[,c(1:41)]
Linear.base<-data.frame(Model=c(),R2=c(), RMSE=c(), MAE=c(), AIC=c(), BIC=c())
for(i in 1:20){
  # Separate training and test set.
  training.samples <- df.base$bloom_doy %>%  createDataPartition(p = 0.70, list = FALSE)
  train.data  <- df.base[training.samples, ]
  test.data <- df.base[-training.samples, ]
  # Build the model
  model1 <- lm(bloom_doy~., data = train.data)
  # Make predictions and compute the R2, RMSE, MAE, AIC, and BIC
  predictions <- model1 %>% predict(test.data)
  result<-data.frame( Model=c("Linear base"),
                      R2 =caret::R2(predictions, test.data$bloom_doy),
                      RMSE = RMSE(predictions, test.data$bloom_doy),
                      MAE = MAE(predictions, test.data$bloom_doy),
                      AIC=AIC(model1),
                      BIC=BIC(model1))
  Linear.base<-rbind(Linear.base,result)
}

Linear.base
```

    ##          Model        R2     RMSE      MAE      AIC      BIC
    ## 1  Linear base 0.8828521 6.285181 4.622371 3373.050 3550.647
    ## 2  Linear base 0.8711205 6.496839 4.818496 3356.360 3533.957
    ## 3  Linear base 0.8809499 6.221971 4.889934 3378.470 3556.068
    ## 4  Linear base 0.8677460 6.284658 4.611111 3373.765 3551.362
    ## 5  Linear base 0.8556265 6.692089 4.901821 3347.811 3525.409
    ## 6  Linear base 0.8794058 6.364754 4.694029 3360.217 3537.815
    ## 7  Linear base 0.8315595 7.285274 5.204955 3284.986 3462.583
    ## 8  Linear base 0.8650900 6.609976 4.746101 3341.110 3518.707
    ## 9  Linear base 0.8434564 7.119685 5.142484 3307.673 3485.271
    ## 10 Linear base 0.8171885 7.714408 5.729047 3295.757 3473.354
    ## 11 Linear base 0.8910997 6.331943 4.563553 3367.429 3545.026
    ## 12 Linear base 0.8512818 6.835226 4.735424 3337.227 3514.825
    ## 13 Linear base 0.8601659 6.775780 5.168318 3350.321 3527.919
    ## 14 Linear base 0.8876546 5.918839 4.525623 3390.446 3568.044
    ## 15 Linear base 0.8858495 6.373431 4.791665 3369.411 3547.009
    ## 16 Linear base 0.8627942 6.698514 4.902752 3336.937 3514.535
    ## 17 Linear base 0.8812241 6.452967 4.624588 3363.978 3541.575
    ## 18 Linear base 0.8160196 7.434203 5.083069 3296.942 3474.539
    ## 19 Linear base 0.8618403 6.349034 4.641692 3366.061 3543.659
    ## 20 Linear base 0.8565910 6.789966 5.021099 3337.476 3515.074

## 2. Linear model : Full model

Next model we considered is full linear model. We cross validated this
model, and the RMSE of the model is about 5.7.

``` r
library(tidyverse)
library(caret)
library(glmnet)
set.seed(003)
# Define the full model

# Cross-validation
df<-cherry
Linear.full<-data.frame(Model=c(),R2=c(), RMSE=c(), MAE=c(), AIC=c(), BIC=c())
for(i in 1:20){
  # Separate training and test set.
  training.samples <- df$bloom_doy %>%  createDataPartition(p = 0.70, list = FALSE)
  train.data  <- df[training.samples, ]
  test.data <- df[-training.samples, ]
  # Build the model
  model2 <- lm(bloom_doy~., data = train.data)
  # Make predictions and compute the R2, RMSE, MAE, AIC, and BIC
  predictions <- model2 %>% predict(test.data)
  result<-data.frame( Model=c("Linear full"),
                      R2 =caret::R2(predictions, test.data$bloom_doy),
                      RMSE = RMSE(predictions, test.data$bloom_doy),
                      MAE = MAE(predictions, test.data$bloom_doy),
                      AIC=AIC(model2),
                      BIC=BIC(model2))
  Linear.full<-rbind(Linear.full,result)
}

Linear.full
```

    ##          Model        R2     RMSE      MAE      AIC      BIC
    ## 1  Linear full 0.8983155 5.814196 4.238576 3159.815 3417.754
    ## 2  Linear full 0.8903276 6.055872 4.478146 3113.872 3371.811
    ## 3  Linear full 0.9235098 4.993721 3.793682 3200.647 3458.586
    ## 4  Linear full 0.8826069 5.921625 4.245855 3122.545 3380.484
    ## 5  Linear full 0.8958066 5.749256 4.227635 3143.724 3401.663
    ## 6  Linear full 0.9023933 5.738848 4.213057 3132.776 3390.715
    ## 7  Linear full 0.8843214 6.056853 4.319954 3098.721 3356.660
    ## 8  Linear full 0.9061456 5.580299 4.082243 3149.416 3407.356
    ## 9  Linear full 0.8821879 6.168772 4.289982 3112.013 3369.953
    ## 10 Linear full 0.8613200 6.727259 4.700177 3092.947 3350.886
    ## 11 Linear full 0.9208509 5.391254 3.884930 3170.076 3428.015
    ## 12 Linear full 0.8952427 5.737498 3.860381 3158.413 3416.352
    ## 13 Linear full 0.9043481 5.605201 4.210126 3163.434 3421.373
    ## 14 Linear full 0.9393141 4.408386 3.499125 3241.970 3499.910
    ## 15 Linear full 0.9155617 5.480040 4.098571 3169.380 3427.319
    ## 16 Linear full 0.9015765 5.698069 3.965643 3137.685 3395.624
    ## 17 Linear full 0.9179296 5.324678 3.862153 3173.628 3431.567
    ## 18 Linear full 0.8707724 6.138206 4.501270 3110.109 3368.048
    ## 19 Linear full 0.8902503 5.661986 4.142401 3147.591 3405.530
    ## 20 Linear full 0.8937041 5.865924 4.445376 3134.239 3392.179

Next, we defined two functions that we will use to calculate AIC and BIC
of PCA, Ridge and Lasso.

``` r
# Define AIC and BIC function for ...
aic.glmnet<-function(fit){
  tLL <- fit$nulldev - deviance(fit)
  k <- fit$df
  n <- fit$nobs
  AICc <- -tLL+2*k+2*k*(k+1)/(n-k-1)
  return(AICc)
}
bic.glmnet<-function(fit){
  tLL <- fit$nulldev - deviance(fit)
  k <- fit$df
  n <- fit$nobs
  BIC<-log(n)*k - tLL
  return(BIC)
}
```

## 3. Ridge model

Now, we will cross validate the Ridge model. We used <tt>glmnet</tt>
function of <tt>glmnet</tt> package. The RMSE was around 5.7 when we
used alpha 1 and lambda 0.001.

``` r
####### cross validate Ridge
library(caret)
library(tidyverse)
library(glmnet)
set.seed(003)

##
lambdas <- 10^seq(2, -3, by = -.1)
y.ridge<-df$bloom_doy
x.ridge<-as.matrix(cherry[,-c(1)])

###
Ridge<-data.frame(Model=c(),R2=c(), RMSE=c(), MAE=c(), AIC=c(), BIC=c())
for(i in 1:20){
  training.samples <- df$bloom_doy %>%  createDataPartition(p = 0.7, list = FALSE)
  train.data  <- x.ridge[training.samples, ]
  test.data <- x.ridge[-training.samples, ]
  y_train<-y.ridge[training.samples]
  y_test<-y.ridge[-training.samples]
  
  # Build the model
  model3 <- glmnet(train.data, y_train, nlambda = 25, alpha = 1, family = 'gaussian', lambda =0.001)
  # Make predictions and compute the R2, RMSE, MAE, AIC, and BIC
  predictions <- model3 %>% predict(as.matrix(test.data))
  result<-data.frame( Model=c("Ridge"),
                      R2 = unname(caret::R2(predictions, y_test)),
                      RMSE = RMSE(predictions, y_test),
                      MAE = MAE(predictions, y_test),
                      AIC = aic.glmnet(model3),
                      BIC = bic.glmnet(model3))
  Ridge<-rbind(Ridge,result)
}


Ridge
```

    ##    Model        R2     RMSE      MAE       AIC       BIC
    ## 1  Ridge 0.9018646 5.711732 4.181709 -144846.5 -144612.9
    ## 2  Ridge 0.8948768 5.882259 4.359382 -147807.1 -147577.1
    ## 3  Ridge 0.9250690 4.935785 3.745504 -145587.4 -145361.1
    ## 4  Ridge 0.8831412 5.904920 4.278099 -153112.3 -152882.3
    ## 5  Ridge 0.8981986 5.674628 4.113186 -150418.9 -150188.9
    ## 6  Ridge 0.9043971 5.687400 4.091621 -145513.7 -145283.7
    ## 7  Ridge 0.8793206 6.190971 4.381596 -150057.2 -149827.2
    ## 8  Ridge 0.9046927 5.604561 4.040686 -147083.3 -146853.3
    ## 9  Ridge 0.8817883 6.172111 4.277402 -148501.1 -148274.8
    ## 10 Ridge 0.8708862 6.482150 4.681284 -148847.1 -148617.2
    ## 11 Ridge 0.9188243 5.456678 3.904117 -137789.1 -137559.1
    ## 12 Ridge 0.8957406 5.721137 3.846353 -149258.3 -149028.3
    ## 13 Ridge 0.9048179 5.585981 4.153924 -145756.9 -145526.9
    ## 14 Ridge 0.9386930 4.412035 3.507599 -147072.1 -146838.5
    ## 15 Ridge 0.9198608 5.316207 3.956130 -141472.9 -141242.9
    ## 16 Ridge 0.9015203 5.703182 3.936099 -146969.5 -146739.5
    ## 17 Ridge 0.9187855 5.296049 3.844396 -142193.9 -141964.0
    ## 18 Ridge 0.8695044 6.158802 4.464048 -155381.2 -155151.2
    ## 19 Ridge 0.8905433 5.646745 4.060485 -154960.9 -154727.3
    ## 20 Ridge 0.8935487 5.869259 4.384611 -147879.5 -147653.2

## 4. Lasso model

The next model we tested is Lasso. We used <tt>glmnet</tt> function of
<tt>glmnet</tt> package. The RMSE was around 6.6 when we used
alpha=0.001.

``` r
######### cross validate LASSO

library(caret)
library(tidyverse)
library(glmnet)
set.seed(003)
Lasso<-data.frame(Model=c(),R2=c(), RMSE=c(), MAE=c(), AIC=c(), BIC=c())
for(i in 1:20){
  y.lasso<-df$bloom_doy
  x.lasso<-as.matrix(cherry[,-c(1)])
  training.samples <- df$bloom_doy %>%  createDataPartition(p = 0.7, list = FALSE)
  train.data  <- x.lasso[training.samples, ]
  test.data <- x.lasso[-training.samples, ]
  y_train<-y.lasso[training.samples]
  y_test<-y.lasso[-training.samples]
  
  # Build the model
  model4 <- cv.glmnet(as.matrix(train.data), y_train, alpha = 0.001)
  
  # Make predictions and compute the R2, RMSE, MAE, AIC, and BIC
  predictions <- model4 %>% predict(as.matrix(test.data), s="lambda.min")
  result<-data.frame( Model=c("Lasso"),
                      R2 =  caret::R2(unname(predictions), y_test),
                      RMSE = RMSE(predictions, y_test),
                      MAE = MAE(predictions, y_test)
  )
  Lasso<-rbind(Lasso,result)
}

Lasso
```

    ##    Model        R2     RMSE      MAE
    ## 1  Lasso 0.8727221 6.516244 4.820548
    ## 2  Lasso 0.8784719 6.447906 5.058703
    ## 3  Lasso 0.8694583 6.211853 4.606443
    ## 4  Lasso 0.8918774 6.195623 4.851015
    ## 5  Lasso 0.8805806 6.321405 4.710872
    ## 6  Lasso 0.8561429 6.940578 5.320391
    ## 7  Lasso 0.8378210 7.201428 5.373793
    ## 8  Lasso 0.7927835 7.453017 5.351604
    ## 9  Lasso 0.8878058 5.969292 4.591526
    ## 10 Lasso 0.8800864 6.510583 4.915182
    ## 11 Lasso 0.8953644 6.199398 4.825779
    ## 12 Lasso 0.8762839 6.548071 4.956386
    ## 13 Lasso 0.8537233 6.665592 4.883605
    ## 14 Lasso 0.8507785 6.770417 5.245580
    ## 15 Lasso 0.8688135 6.533927 4.912497
    ## 16 Lasso 0.8807086 6.246359 4.968802
    ## 17 Lasso 0.8580632 6.597780 5.129712
    ## 18 Lasso 0.8610243 6.638272 4.900121
    ## 19 Lasso 0.8337816 7.169417 5.254145
    ## 20 Lasso 0.8490558 7.111942 5.309666

## 5. PCA model

The final method we tested is PCA regression. We used <tt>pca</tt>
function of <tt>pls</tt> package. The RMSE of the method was around 5.8
when we used 45 components.

``` r
library(tidyverse)
library(pls)
library(caret)
set.seed(003)
########### cross validate PCA
library(pls)
df<-cherry
PCA<-data.frame(Model=c(),R2=c(), RMSE=c(), MAE=c(), AIC=c(), BIC=c())
for(i in 1:20){
  training.samples <- df$bloom_doy %>%  createDataPartition(p = 0.7, list = FALSE)
  train.data  <- df[training.samples, ]
  test.data <- df[-training.samples, ]
  #y_train<-y[training.samples]
  y_test<-test.data$bloom_doy
  
  # Build the model
  model5 <- pcr(bloom_doy~., data = train.data, scale = TRUE, validation = "CV")
  # Make predictions and compute the R2, RMSE, MAE, AIC, and BIC
  prediction <- model5 %>% predict(test.data)
  predictions<-unname(unlist(prediction[,,45]))
  result<-data.frame( Model5=c("PCA"),
                      R2 =caret::R2(predictions, y_test),
                      RMSE = RMSE(predictions, y_test),
                      MAE = MAE(predictions, y_test))
  PCA<-rbind(PCA,result)
}

validationplot(model5, val.type="R2")
```

![](Code-crossvalidation-2022_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

## CV RESULT

The table below highlights the performance of each model. In this cross
validation, Ridge showed the lowest RMSE. However, since the result was
different when we used python, the models were examined twice to compare
the result.

``` r
CV.RESULT<-data.frame(MODEL=c("Base Linear", "Full Linear", "Ridge", "Lasso", "PCA"),
           MEAN.R2=c(mean(Linear.base$R2), mean(Linear.full$R2), mean(Ridge$R2), mean(Lasso$R2), mean(PCA$R2)),           
           MEAN.RMSE=c(mean(Linear.base$RMSE), mean(Linear.full$RMSE), mean(Ridge$RMSE), mean(Lasso$RMSE), mean(PCA$RMSE)), 
           VARIANCE.RMSE=c(var(Linear.base$RMSE),var(Linear.full$RMSE),var(Ridge$RMSE),var(Lasso$RMSE),var(PCA$RMSE)))
CV.RESULT
```

    ##         MODEL   MEAN.R2 MEAN.RMSE VARIANCE.RMSE
    ## 1 Base Linear 0.8624758  6.651737    0.20230278
    ## 2 Full Linear 0.8988243  5.705897    0.22588370
    ## 3       Ridge 0.8998037  5.670630    0.21114228
    ## 4       Lasso 0.8637674  6.612455    0.15493230
    ## 5         PCA 0.8988010  5.699656    0.09097566

``` r
#write.csv(CV.RESULT, "cvresult.csv")
```

## Coefficients of final Lasso model

``` r
# Coefficient of Lasso model
set.seed(003)
y.lasso<-df$bloom_doy
x.lasso<-as.matrix(cherry[,-c(1)])
# Build the model
model.Lasso <- cv.glmnet(as.matrix(x.lasso), y.lasso, alpha = 1)

# Get coefficient
Lasso.coef<-coef(model.Lasso)
Lasso.coef
```

    ## 60 x 1 sparse Matrix of class "dgCMatrix"
    ##                                       s1
    ## (Intercept)                 1.348276e+02
    ## lat                         1.130547e+00
    ## alt                        -3.884403e-03
    ## year                       -5.616266e-02
    ## year.sq                     .           
    ## Date_doy_tavg               5.727103e-02
    ## Date_doy_tmax               .           
    ## Date_doy_tmin               2.773780e-02
    ## tavg_below_5                3.677564e-01
    ## tavg_above_10               .           
    ## tavg_moving                 .           
    ## tmin_moving                 .           
    ## tmax_moving                 3.168721e-03
    ## prcp..2                     .           
    ## prcp..1                    -2.330140e-01
    ## prcp.0                      .           
    ## prcp.1                      3.143898e-02
    ## prcp.2                      4.342202e-01
    ## tavg..2                     .           
    ## tavg..1                     .           
    ## tavg.0                      5.219097e-03
    ## tavg.1                      1.683816e-01
    ## tavg.2                      .           
    ## tmax..2                     .           
    ## tmax..1                     4.271414e-02
    ## tmax.0                      3.035158e-04
    ## tmax.1                      .           
    ## tmax.2                     -9.222218e-02
    ## tmin..2                     3.394619e-02
    ## tmin..1                    -2.412029e-03
    ## tmin.0                      .           
    ## tmin.1                      .           
    ## tmin.2                     -3.032819e-02
    ## long.y                      .           
    ## long.x                     -1.528280e+01
    ## slope                      -2.039200e-01
    ## dg2_coef                   -5.375748e-02
    ## intc                        .           
    ## tavg.3                      .           
    ## tmin.3                     -8.152518e-03
    ## tmax.3                     -3.369468e-02
    ## tavg_below_5tavg.1         -1.647901e-03
    ## lattavg_above_10           -2.140019e-03
    ## tavg_below_5prcp.0         -1.400676e-03
    ## tavg.0tmax.2                1.339099e-03
    ## prcp..2prcp.2              -3.616142e-02
    ## altprcp..1                  6.936453e-04
    ## latlong.y                  -1.872140e-01
    ## prcp..2prcp.0               1.302126e-02
    ## prcp.2tavg.0               -3.823305e-04
    ## Date_doy_tmaxDate_doy_tmin -9.593649e-04
    ## tmin_movingtmin.1          -3.614454e-04
    ## tavg_below_5prcp.2          .           
    ## year.sqDate_doy_tavg        .           
    ## tmin..1long.y               9.101657e-02
    ## tmax..1long.x               .           
    ## yearprcp.2                  .           
    ## alttavg_moving             -5.033901e-05
    ## prcp..1tmax.3               5.568866e-04
    ## long.ylong.x                1.539539e+01

## Coefficients of Ridge model

``` r
library(caret)
library(tidyverse)
library(glmnet)
set.seed(003)

##
df<-cherry
lambdas <- 10^seq(2, -3, by = -.1)
y.ridge<-df$bloom_doy
x.ridge<-as.matrix(cherry[,-c(1)])
# Build the model
model.ridge <- glmnet(x.ridge, y.ridge, nlambda = 25, alpha = 0, family = 'gaussian', lambda =0.001)

# Get coefficient 
model.ridge$beta
```

    ## 59 x 1 sparse Matrix of class "dgCMatrix"
    ##                                       s0
    ## lat                         2.592334e+00
    ## alt                        -4.749386e-03
    ## year                       -9.521346e-02
    ## year.sq                     1.194681e-05
    ## Date_doy_tavg               7.306853e-01
    ## Date_doy_tmax               3.283176e-03
    ## Date_doy_tmin               2.378150e-02
    ## tavg_below_5                4.814620e-01
    ## tavg_above_10               7.452627e-01
    ## tavg_moving                -1.041064e-01
    ## tmin_moving                 3.422607e-02
    ## tmax_moving                 5.776955e-02
    ## prcp..2                    -1.642158e-01
    ## prcp..1                    -4.167437e-01
    ## prcp.0                     -1.408242e-02
    ## prcp.1                      1.346731e-02
    ## prcp.2                      6.647243e-01
    ## tavg..2                    -1.473081e-01
    ## tavg..1                    -3.513573e-02
    ## tavg.0                      3.992273e-01
    ## tavg.1                      2.485466e-01
    ## tavg.2                      7.915773e-03
    ## tmax..2                     4.329561e-02
    ## tmax..1                     1.176376e-01
    ## tmax.0                     -6.987612e-02
    ## tmax.1                     -9.067895e-03
    ## tmax.2                     -6.564326e-02
    ## tmin..2                     1.728202e-01
    ## tmin..1                    -5.407032e-02
    ## tmin.0                     -2.027726e-01
    ## tmin.1                     -1.979193e-02
    ## tmin.2                     -3.205844e-02
    ## long.y                      4.299508e+01
    ## long.x                     -1.913847e+01
    ## slope                      -2.533367e+00
    ## dg2_coef                   -4.279518e-02
    ## intc                       -3.861068e-02
    ## tavg.3                      1.231680e-01
    ## tmin.3                     -3.977027e-02
    ## tmax.3                     -9.341228e-02
    ## tavg_below_5tavg.1         -2.038762e-03
    ## lattavg_above_10           -2.959615e-02
    ## tavg_below_5prcp.0         -4.002439e-03
    ## tavg.0tmax.2                1.290758e-03
    ## prcp..2prcp.2              -4.809969e-02
    ## altprcp..1                  9.067146e-04
    ## latlong.y                  -1.398787e+00
    ## prcp..2prcp.0               3.433733e-02
    ## prcp.2tavg.0               -7.808070e-03
    ## Date_doy_tmaxDate_doy_tmin -1.512715e-03
    ## tmin_movingtmin.1          -5.218937e-04
    ## tavg_below_5prcp.2         -4.300524e-03
    ## year.sqDate_doy_tavg       -1.686115e-07
    ## tmin..1long.y               1.515926e-01
    ## tmax..1long.x               6.466074e-02
    ## yearprcp.2                  3.066792e-04
    ## alttavg_moving             -8.261394e-05
    ## prcp..1tmax.3               1.857955e-03
    ## long.ylong.x               -6.324951e+00
