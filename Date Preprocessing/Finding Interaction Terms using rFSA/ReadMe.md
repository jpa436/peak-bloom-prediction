Cherry Blossom prediction: Finding interactions
================

<!--   rmarkdown::github_document
    code_folding: hide
    code_download: true -->

### <b> 1. Finding interaction terms using FSA </b>

In this page, we will demonstrate how we chose interaction terms. First,
we used FSA to seek for possibility of interactions. To avoid prevalence
of interaction term that might have been explained by dropped main
effect, we included all main effects in the model to start.

``` r
library(rFSA)

#set the formula 
formula<-as.formula("bloom_doy~lat+alt+year+year.sq+Date_doy_tavg+Date_doy_tmax+Date_doy_tmin+tavg_below_5+tavg_above_10+tavg_moving+tmin_moving+tmax_moving+prcp..2+prcp..1+prcp.0+prcp.1+prcp.2+tavg..2+tavg..1+tavg.0+tavg.1+tavg.2+tmax..2+tmax..1+tmax.0+tmax.1+tmax.2+tmin..2+tmin..1+tmin.0+tmin.1+tmin.2+long.y+long.x+slope+dg2_coef+intc+tavg.3+tmin.3+tmax.3")

# Search for interaction term using FSA
FSA(formula, data=cherry, fitfunc=lm, fixvar=c("lat","alt","year","year.sq","Date_doy_tavg","Date_doy_tmax",
                                               "Date_doy_tmin","tavg_below_5","tavg_above_10","tavg_moving",
                                               "tmin_moving","tmax_moving","prcp..2","prcp..1","prcp.0","prcp.1",
                                               "prcp.2","tavg..2","tavg..1","tavg.0","tavg.1","tavg.2","tmax..2",
                                               "tmax..1","tmax.0","tmax.1","tmax.2","tmin..2","tmin..1","tmin.0","tmin.1",
                                               "tmin.2","long.y","long.x","slope","dg2_coef","intc","tavg.3","tmin.3","tmax.3",
                                               "lat*tavg_above_10", "tavg_below_5*tavg.1", "tavg_below_5*prcp.0", "tavg.0*tmax.2"
                                               , "prcp..2*prcp.2", "alt*prcp..1", "lat*long.y", "prcp..2*prcp.0", "prcp.2*tavg.0",
                                               "Date_doy_tmax*Date_doy_tmin", "tmin_moving*tmin.1", "tavg_below_5*prcp.2",
                                               "year.sq*Date_doy_tavg", "tmin..1*long.y", "tmax..1*long.x", "year*prcp.2",
                                               "alt*tavg_moving", "prcp..1*tmax.3", "prcp..1*tmax.3"), quad=T, interactions=T )
```

    ##                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   formula
    ## Original Fit                                                                                                                                                                                                                             bloom_doy ~ lat + alt + year + year.sq + Date_doy_tavg + Date_doy_tmax +      Date_doy_tmin + tavg_below_5 + tavg_above_10 + tavg_moving +      tmin_moving + tmax_moving + prcp..2 + prcp..1 + prcp.0 +      prcp.1 + prcp.2 + tavg..2 + tavg..1 + tavg.0 + tavg.1 + tavg.2 +      tmax..2 + tmax..1 + tmax.0 + tmax.1 + tmax.2 + tmin..2 +      tmin..1 + tmin.0 + tmin.1 + tmin.2 + long.y + long.x + slope +      dg2_coef + intc + tavg.3 + tmin.3 + tmax.3
    ## FS1          bloom_doy~lat+alt+year+year.sq+Date_doy_tavg+Date_doy_tmax+Date_doy_tmin+tavg_below_5+tavg_above_10+tavg_moving+tmin_moving+tmax_moving+prcp..2+prcp..1+prcp.0+prcp.1+prcp.2+tavg..2+tavg..1+tavg.0+tavg.1+tavg.2+tmax..2+tmax..1+tmax.0+tmax.1+tmax.2+tmin..2+tmin..1+tmin.0+tmin.1+tmin.2+long.y+long.x+slope+dg2_coef+intc+tavg.3+tmin.3+tmax.3+lat*tavg_above_10+tavg_below_5*tavg.1+tavg_below_5*prcp.0+tavg.0*tmax.2+prcp..2*prcp.2+alt*prcp..1+lat*long.y+prcp..2*prcp.0+prcp.2*tavg.0+Date_doy_tmax*Date_doy_tmin+tmin_moving*tmin.1+tavg_below_5*prcp.2+year.sq*Date_doy_tavg+tmin..1*long.y+tmax..1*long.x+year*prcp.2+alt*tavg_moving+prcp..1*tmax.3+prcp..1*tmax.3+long.y*long.x
    ##              criterion times
    ## Original Fit  4740.077    NA
    ## FS1           4463.984     1

Starting with all main effects, we kept seaching for interaction terms
until AIC does not get better. The interaction term we found are:

<tt>lat\*tavg\_above\_10</tt>, <tt>tavg\_below\_5\*tavg.1</tt>,
<tt>tavg\_below\_5\*prcp.0</tt>, <tt>tavg.0\*tmax.2</tt>,
<tt>prcp..2\*prcp.2</tt>, <tt>alt\*prcp..1</tt>, <tt>lat\*long.y</tt>,
<tt>prcp..2\*prcp.0</tt>, <tt>prcp.2\*tavg.0</tt>,
<tt>Date\_doy\_tmax\*Date\_doy\_tmin</tt>,
<tt>tmin\_moving\*tmin.1</tt>, <tt>tavg\_below\_5\*prcp.2</tt>,
<tt>year.sq\*Date\_doy\_tavg</tt>, <tt>tmin..1\*long.y</tt>,
<tt>tmax..1\*long.x</tt>, <tt>year\*prcp.2</tt>,
<tt>alt\*tavg\_moving</tt>, <tt>prcp..1\*tmax.3</tt>,
<tt>prcp..1\*tmax.3</tt>.

### <b> 2. Backward selection</b>

Then, we applied forward and backward selection to trim down the model.

``` r
# Full model.
formula<-as.formula("bloom_doy~lat+alt+year+year.sq+Date_doy_tavg+Date_doy_tmax+Date_doy_tmin+tavg_below_5+tavg_above_10+tavg_moving+tmin_moving+tmax_moving+prcp..2+prcp..1+prcp.0+prcp.1+prcp.2+tavg..2+tavg..1+tavg.0+tavg.1+tavg.2+tmax..2+tmax..1+tmax.0+tmax.1+tmax.2+tmin..2+tmin..1+tmin.0+tmin.1+tmin.2+long.y+long.x+slope+dg2_coef+intc+tavg.3+tmin.3+tmax.3+lat*tavg_above_10+tavg_below_5*tavg.1+tavg_below_5*prcp.0+tavg.0*tmax.2+prcp..2*prcp.2+alt*prcp..1+lat*long.y+prcp..2*prcp.0+prcp.2*tavg.0+Date_doy_tmax*Date_doy_tmin+tmin_moving*tmin.1+tavg_below_5*prcp.2+year.sq*Date_doy_tavg+tmin..1*long.y+tmax..1*long.x+year*prcp.2+alt*tavg_moving+prcp..1*tmax.3+long.y*long.x")

full.model <- lm(formula, data = cherry)

library(MASS)

# Backward selection
# Stepwise regression model
step.model.back <- stepAIC(full.model, direction = "backward", trace = FALSE)
summary(step.model.back)
```

    ## 
    ## Call:
    ## lm(formula = bloom_doy ~ lat + alt + year + year.sq + Date_doy_tavg + 
    ##     Date_doy_tmax + Date_doy_tmin + tavg_below_5 + tavg_above_10 + 
    ##     tavg_moving + tmin_moving + tmax_moving + prcp..2 + prcp..1 + 
    ##     prcp.0 + prcp.2 + tavg.0 + tavg.1 + tmax..1 + tmax.2 + tmin..2 + 
    ##     tmin..1 + tmin.0 + tmin.1 + long.y + long.x + slope + intc + 
    ##     tavg.3 + tmax.3 + lat:tavg_above_10 + tavg_below_5:tavg.1 + 
    ##     tavg_below_5:prcp.0 + tavg.0:tmax.2 + prcp..2:prcp.2 + alt:prcp..1 + 
    ##     lat:long.y + prcp..2:prcp.0 + prcp.2:tavg.0 + Date_doy_tmax:Date_doy_tmin + 
    ##     tmin_moving:tmin.1 + tavg_below_5:prcp.2 + year.sq:Date_doy_tavg + 
    ##     tmin..1:long.y + tmax..1:long.x + year:prcp.2 + alt:tavg_moving + 
    ##     prcp..1:tmax.3 + long.y:long.x, data = cherry)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -18.7943  -2.9927  -0.1229   2.5875  27.5984 
    ## 
    ## Coefficients:
    ##                               Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                 -3.682e+02  2.642e+03  -0.139 0.889196    
    ## lat                          3.841e+00  5.977e-01   6.426 2.48e-10 ***
    ## alt                          7.281e-04  6.594e-03   0.110 0.912110    
    ## year                         2.095e-01  2.648e+00   0.079 0.936973    
    ## year.sq                     -4.331e-05  6.647e-04  -0.065 0.948060    
    ## Date_doy_tavg                1.792e+00  5.911e-01   3.032 0.002526 ** 
    ## Date_doy_tmax                8.764e-04  2.319e-02   0.038 0.969865    
    ## Date_doy_tmin                2.623e-02  1.692e-02   1.550 0.121509    
    ## tavg_below_5                 4.809e-01  3.483e-02  13.806  < 2e-16 ***
    ## tavg_above_10                7.926e-01  3.333e-01   2.378 0.017690 *  
    ## tavg_moving                 -8.502e-02  4.990e-02  -1.704 0.088878 .  
    ## tmin_moving                  2.405e-02  3.127e-02   0.769 0.442108    
    ## tmax_moving                  4.570e-02  2.602e-02   1.756 0.079470 .  
    ## prcp..2                     -8.755e-02  1.077e-01  -0.813 0.416382    
    ## prcp..1                     -4.596e-01  1.052e-01  -4.367 1.46e-05 ***
    ## prcp.0                      -2.943e-02  1.221e-01  -0.241 0.809561    
    ## prcp.2                       1.121e+01  4.056e+00   2.762 0.005895 ** 
    ## tavg.0                       2.790e-01  6.150e-02   4.537 6.77e-06 ***
    ## tavg.1                       2.358e-01  5.448e-02   4.329 1.73e-05 ***
    ## tmax..1                      8.820e-02  2.481e-02   3.555 0.000404 ***
    ## tmax.2                      -8.113e-02  2.008e-02  -4.040 5.97e-05 ***
    ## tmin..2                      8.110e-02  1.624e-02   4.994 7.56e-07 ***
    ## tmin..1                     -3.895e-02  3.285e-02  -1.186 0.236094    
    ## tmin.0                      -1.494e-01  5.210e-02  -2.868 0.004258 ** 
    ## tmin.1                      -1.565e-02  5.034e-02  -0.311 0.755978    
    ## long.y                       1.032e+02  2.744e+01   3.761 0.000184 ***
    ## long.x                      -1.517e+01  3.667e+00  -4.138 3.96e-05 ***
    ## slope                       -2.941e+00  1.484e+00  -1.982 0.047900 *  
    ## intc                        -4.679e-02  2.573e-02  -1.819 0.069392 .  
    ## tavg.3                       6.598e-02  3.479e-02   1.896 0.058346 .  
    ## tmax.3                      -7.470e-02  2.965e-02  -2.520 0.011983 *  
    ## lat:tavg_above_10           -3.140e-02  9.857e-03  -3.185 0.001514 ** 
    ## tavg_below_5:tavg.1         -2.011e-03  3.242e-04  -6.203 9.70e-10 ***
    ## tavg_below_5:prcp.0         -3.728e-03  1.173e-03  -3.178 0.001549 ** 
    ## tavg.0:tmax.2                1.357e-03  2.640e-04   5.139 3.62e-07 ***
    ## prcp..2:prcp.2              -5.078e-02  8.532e-03  -5.952 4.28e-09 ***
    ## alt:prcp..1                  9.023e-04  2.074e-04   4.352 1.56e-05 ***
    ## lat:long.y                  -2.959e+00  7.143e-01  -4.143 3.87e-05 ***
    ## prcp..2:prcp.0               3.246e-02  7.549e-03   4.300 1.97e-05 ***
    ## prcp.2:tavg.0               -8.150e-03  2.153e-03  -3.785 0.000167 ***
    ## Date_doy_tmax:Date_doy_tmin -1.486e-03  4.191e-04  -3.546 0.000418 ***
    ## tmin_moving:tmin.1          -4.521e-04  1.925e-04  -2.348 0.019163 *  
    ## tavg_below_5:prcp.2         -4.753e-03  1.965e-03  -2.419 0.015827 *  
    ## year.sq:Date_doy_tavg       -4.375e-07  1.497e-07  -2.924 0.003576 ** 
    ## tmin..1:long.y               1.133e-01  3.609e-02   3.139 0.001772 ** 
    ## tmax..1:long.x               6.153e-02  2.024e-02   3.040 0.002458 ** 
    ## year:prcp.2                 -4.969e-03  2.040e-03  -2.435 0.015136 *  
    ## alt:tavg_moving             -8.160e-05  4.457e-05  -1.831 0.067552 .  
    ## prcp..1:tmax.3               2.070e-03  8.813e-04   2.349 0.019107 *  
    ## long.y:long.x               -3.336e+01  1.387e+01  -2.406 0.016396 *  
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 5.136 on 670 degrees of freedom
    ## Multiple R-squared:  0.9224, Adjusted R-squared:  0.9168 
    ## F-statistic: 162.6 on 49 and 670 DF,  p-value: < 2.2e-16

``` r
plot(step.model.back$residuals)
```

![](Code-finding-interactions_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

### <b> 3. Forward selection </b>

Since backward selection found better model than forward selection, we
finalized the list of variables that are included in our model. The list
of variables are provided below:

<tt>lat\*tavg\_above\_10</tt>, <tt>tavg\_below\_5\*tavg.1</tt>,
<tt>tavg\_below\_5\*prcp.0</tt>, <tt>tavg.0\*tmax.2</tt>,
<tt>prcp..2\*prcp.2</tt>, <tt>alt\*prcp..1</tt>, <tt>lat\*long.y</tt>,
<tt>prcp..2\*prcp.0</tt>, <tt>prcp.2\*tavg.0</tt>,
<tt>Date\_doy\_tmax\*Date\_doy\_tmin</tt>,
<tt>tmin\_moving\*tmin.1</tt>, <tt>tavg\_below\_5\*prcp.2</tt>,
<tt>year.sq\*Date\_doy\_tavg</tt>, <tt>tmin..1\*long.y</tt>,
<tt>tmax..1\*long.x</tt>, <tt>year\*prcp.2</tt>,
<tt>alt\*tavg\_moving</tt>, <tt>prcp..1\*tmax.3</tt>,
<tt>long.y\*long.x</tt>

``` r
# Forward selection
# Stepwise regression model
step.model.for <- stepAIC(full.model, direction = "forward", trace = FALSE)
summary(step.model.for)
```

    ## 
    ## Call:
    ## lm(formula = bloom_doy ~ lat + alt + year + year.sq + Date_doy_tavg + 
    ##     Date_doy_tmax + Date_doy_tmin + tavg_below_5 + tavg_above_10 + 
    ##     tavg_moving + tmin_moving + tmax_moving + prcp..2 + prcp..1 + 
    ##     prcp.0 + prcp.1 + prcp.2 + tavg..2 + tavg..1 + tavg.0 + tavg.1 + 
    ##     tavg.2 + tmax..2 + tmax..1 + tmax.0 + tmax.1 + tmax.2 + tmin..2 + 
    ##     tmin..1 + tmin.0 + tmin.1 + tmin.2 + long.y + long.x + slope + 
    ##     dg2_coef + intc + tavg.3 + tmin.3 + tmax.3 + lat * tavg_above_10 + 
    ##     tavg_below_5 * tavg.1 + tavg_below_5 * prcp.0 + tavg.0 * 
    ##     tmax.2 + prcp..2 * prcp.2 + alt * prcp..1 + lat * long.y + 
    ##     prcp..2 * prcp.0 + prcp.2 * tavg.0 + Date_doy_tmax * Date_doy_tmin + 
    ##     tmin_moving * tmin.1 + tavg_below_5 * prcp.2 + year.sq * 
    ##     Date_doy_tavg + tmin..1 * long.y + tmax..1 * long.x + year * 
    ##     prcp.2 + alt * tavg_moving + prcp..1 * tmax.3 + long.y * 
    ##     long.x, data = cherry)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -18.0499  -2.9556  -0.1196   2.5307  27.2205 
    ## 
    ## Coefficients:
    ##                               Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                 -5.962e+02  2.772e+03  -0.215 0.829803    
    ## lat                          3.778e+00  6.109e-01   6.185 1.09e-09 ***
    ## alt                          6.712e-06  6.971e-03   0.001 0.999232    
    ## year                         4.384e-01  2.779e+00   0.158 0.874701    
    ## year.sq                     -9.935e-05  6.975e-04  -0.142 0.886784    
    ## Date_doy_tavg                1.658e+00  6.039e-01   2.746 0.006200 ** 
    ## Date_doy_tmax                2.989e-03  2.376e-02   0.126 0.899939    
    ## Date_doy_tmin                2.521e-02  1.740e-02   1.449 0.147838    
    ## tavg_below_5                 4.756e-01  3.574e-02  13.307  < 2e-16 ***
    ## tavg_above_10                8.599e-01  3.468e-01   2.480 0.013402 *  
    ## tavg_moving                 -1.007e-01  5.126e-02  -1.964 0.049938 *  
    ## tmin_moving                  3.910e-02  3.362e-02   1.163 0.245331    
    ## tmax_moving                  5.296e-02  2.661e-02   1.990 0.046993 *  
    ## prcp..2                     -1.470e-01  1.193e-01  -1.232 0.218402    
    ## prcp..1                     -4.567e-01  1.087e-01  -4.202 3.01e-05 ***
    ## prcp.0                      -5.562e-02  1.258e-01  -0.442 0.658530    
    ## prcp.1                      -1.564e-02  5.173e-02  -0.302 0.762579    
    ## prcp.2                       1.210e+01  4.201e+00   2.880 0.004102 ** 
    ## tavg..2                     -1.213e-02  1.198e-01  -0.101 0.919395    
    ## tavg..1                     -1.895e-01  1.410e-01  -1.344 0.179412    
    ## tavg.0                       3.750e-01  1.601e-01   2.342 0.019476 *  
    ## tavg.1                       2.202e-01  1.304e-01   1.689 0.091660 .  
    ## tavg.2                       5.682e-02  1.370e-01   0.415 0.678518    
    ## tmax..2                     -2.460e-02  6.134e-02  -0.401 0.688481    
    ## tmax..1                      1.817e-01  7.191e-02   2.527 0.011750 *  
    ## tmax.0                      -4.829e-02  8.046e-02  -0.600 0.548578    
    ## tmax.1                       6.738e-03  6.989e-02   0.096 0.923223    
    ## tmax.2                      -9.011e-02  6.723e-02  -1.340 0.180599    
    ## tmin..2                      1.096e-01  6.583e-02   1.666 0.096253 .  
    ## tmin..1                      5.119e-02  8.076e-02   0.634 0.526403    
    ## tmin.0                      -1.967e-01  8.658e-02  -2.272 0.023436 *  
    ## tmin.1                      -1.568e-02  7.246e-02  -0.216 0.828772    
    ## tmin.2                      -5.985e-02  7.495e-02  -0.799 0.424863    
    ## long.y                       1.018e+02  2.784e+01   3.655 0.000278 ***
    ## long.x                      -1.468e+01  3.752e+00  -3.912 0.000101 ***
    ## slope                       -2.800e+00  1.553e+00  -1.803 0.071799 .  
    ## dg2_coef                    -1.156e-01  9.121e-01  -0.127 0.899224    
    ## intc                        -4.358e-02  2.701e-02  -1.613 0.107125    
    ## tavg.3                       1.250e-01  8.660e-02   1.444 0.149248    
    ## tmin.3                      -3.737e-02  4.621e-02  -0.809 0.418901    
    ## tmax.3                      -1.004e-01  4.593e-02  -2.186 0.029138 *  
    ## lat:tavg_above_10           -3.352e-02  1.019e-02  -3.289 0.001060 ** 
    ## tavg_below_5:tavg.1         -1.996e-03  3.376e-04  -5.912 5.42e-09 ***
    ## tavg_below_5:prcp.0         -3.732e-03  1.189e-03  -3.138 0.001778 ** 
    ## tavg.0:tmax.2                1.351e-03  2.725e-04   4.959 9.02e-07 ***
    ## prcp..2:prcp.2              -5.029e-02  8.614e-03  -5.838 8.28e-09 ***
    ## alt:prcp..1                  8.723e-04  2.147e-04   4.063 5.42e-05 ***
    ## lat:long.y                  -2.927e+00  7.252e-01  -4.037 6.06e-05 ***
    ## prcp..2:prcp.0               3.395e-02  7.676e-03   4.422 1.14e-05 ***
    ## prcp.2:tavg.0               -8.125e-03  2.176e-03  -3.735 0.000204 ***
    ## Date_doy_tmax:Date_doy_tmin -1.485e-03  4.294e-04  -3.457 0.000581 ***
    ## tmin_moving:tmin.1          -5.175e-04  2.007e-04  -2.579 0.010126 *  
    ## tavg_below_5:prcp.2         -4.892e-03  1.977e-03  -2.474 0.013603 *  
    ## year.sq:Date_doy_tavg       -4.040e-07  1.529e-07  -2.643 0.008423 ** 
    ## tmin..1:long.y               1.179e-01  3.651e-02   3.229 0.001305 ** 
    ## tmax..1:long.x               6.021e-02  2.087e-02   2.885 0.004047 ** 
    ## year:prcp.2                 -5.391e-03  2.112e-03  -2.553 0.010903 *  
    ## alt:tavg_moving             -8.263e-05  4.550e-05  -1.816 0.069790 .  
    ## prcp..1:tmax.3               2.237e-03  8.970e-04   2.494 0.012868 *  
    ## long.y:long.x               -3.303e+01  1.408e+01  -2.346 0.019275 *  
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 5.154 on 660 degrees of freedom
    ## Multiple R-squared:  0.9231, Adjusted R-squared:  0.9162 
    ## F-statistic: 134.2 on 59 and 660 DF,  p-value: < 2.2e-16

### <b> 4.Make a new dataset containing interaction effects </b>

Now, we will make a dataset include all main effects and interaction
effects.

``` r
attach(cherry)
cherry$lattavg_above_10<-lat*tavg_above_10
cherry$tavg_below_5tavg.1<-tavg_below_5*tavg.1
cherry$tavg_below_5prcp.0<-tavg_below_5*prcp.0
cherry$tavg.0tmax.2<-tavg.0*tmax.2
cherry$prcp..2prcp.2<-prcp..2*prcp.2
cherry$altprcp..1   <-alt*prcp..1
cherry$latlong.y<-lat*long.y
cherry$prcp..2prcp.0<-prcp..2*prcp.0
cherry$prcp.2tavg.0<-prcp.2*tavg.0
cherry$Date_doy_tmaxDate_doy_tmin<-Date_doy_tmax*Date_doy_tmin
cherry$tmin_movingtmin.1<-tmin_moving*tmin.1
cherry$tavg_below_5prcp.2<-tavg_below_5*prcp.2
cherry$year.sqDate_doy_tavg<-year.sq*Date_doy_tavg
cherry$tmin..1long.y<-tmin..1*long.y
cherry$tmax..1long.x<-tmax..1*long.x
cherry$yearprcp.2<-year*prcp.2
cherry$alttavg_moving<-alt*tavg_moving
cherry$prcp..1tmax.3<-prcp..1*tmax.3
cherry$long.ylong.x<-long.y*long.x
write.csv(cherry, "final final.csv")
```
