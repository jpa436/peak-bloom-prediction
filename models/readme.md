Models
================
Lee Park
2/28/2022

# Models considered:

## For predicting 2022

1.  Linear Models : Simple Linear Model, Lasso Regression (alpha =
    0.001), Ridge Regression (alpha = 0.25)
2.  Tree Based Models (ccp\_alpha = 0.87): Tree Regression, Tree
    Regression + Bagging, Random Forest
3.  MARS (max\_term = 40) : Mars, Mars with Adaptive Boosting

The table below is the mean and variance of RMSE from 100 cross validation with the train-test ratio at 7 to 3.
In the codes, they might have 20 cross validation but we later ran 100 cross validation to make more accurate result.

``` r
models = c('Simple Linear Model', 'Lasso Regression','Ridge Regression', 
           'Tree Regression','Tree + Bagging', 'Random Forest',
           'MARS','MARS with Adaptive Boosting')
mean = c(5.705897, 6.751583674373004, 5.694200015127804, 7.170382345709941, 6.142025322693755, 6.271237557481661,
         6.432694677236944, 6.11578807293314)
variance = c(0.2258837, 0.16127463125347957,  0.12597581682538359, 0.3775812306858149, 0.15284087784897424, 0.17063272712196434, 
             0.36151446858248293, 0.24195940467583985 )
data.frame(models, 'mean RMSE' =  mean, 'variance RMSE' = variance)
```

    ##                        models mean.RMSE variance.RMSE
    ## 1         Simple Linear Model  5.705897     0.2258837
    ## 2            Lasso Regression  6.751584     0.1612746
    ## 3            Ridge Regression  5.694200     0.1259758
    ## 4             Tree Regression  7.170382     0.3775812
    ## 5              Tree + Bagging  6.142025     0.1528409
    ## 6               Random Forest  6.271238     0.1706327
    ## 7                        MARS  6.432695     0.3615145
    ## 8 MARS with Adaptive Boosting  6.115788     0.2419594




## For predicting 2023 - 2031

1. For Washington DC, Kyoto, and Liestal: Simple Linear Model (bloom_doy ~ year + I(year^2)) for each city individually
2. For Toronto: Generalized Additive Model
