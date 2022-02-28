Cherry Blossom prediction: 2023-2031 Model for Liestal, Kyoto and
Washington DC
================

<!--   rmarkdown::github_document
    code_folding: hide
    code_download: true -->

In this page, we will illustrate how we extrapolated our prediction to
year 2023-2031 for three cities(Liestal, Kyoto, and Washington DC).

Considering the recent rising trend in temperature, we used data after
year 1988.

The result follows:

``` r
m.kyoto<-lm(data=kyoto[kyoto$year>1988,], bloom_doy~year+I(year^2))
m.liestal<-lm(data=liestal[liestal$year>1988,], bloom_doy~year+I(year^2))
m.DC<-lm(data=DC[DC$year>1988,], bloom_doy~year+I(year^2))

summary(m.kyoto)
```

    ## 
    ## Call:
    ## lm(formula = bloom_doy ~ year + I(year^2), data = kyoto[kyoto$year > 
    ##     1988, ])
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -6.1676 -1.6322  0.1771  2.6460  6.7582 
    ## 
    ## Coefficients:
    ##               Estimate Std. Error t value Pr(>|t|)  
    ## (Intercept) -8.131e+04  3.055e+04  -2.661   0.0124 *
    ## year         8.129e+01  3.048e+01   2.667   0.0122 *
    ## I(year^2)   -2.030e-02  7.601e-03  -2.670   0.0121 *
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 3.536 on 30 degrees of freedom
    ## Multiple R-squared:  0.2315, Adjusted R-squared:  0.1802 
    ## F-statistic: 4.518 on 2 and 30 DF,  p-value: 0.01927

``` r
summary(m.liestal)
```

    ## 
    ## Call:
    ## lm(formula = bloom_doy ~ year + I(year^2), data = liestal[liestal$year > 
    ##     1988, ])
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -17.6667  -5.4402  -0.5408   5.4742  16.6165 
    ## 
    ## Coefficients:
    ##               Estimate Std. Error t value Pr(>|t|)
    ## (Intercept) -1.238e+05  7.565e+04  -1.637    0.112
    ## year         1.236e+02  7.546e+01   1.638    0.112
    ## I(year^2)   -3.082e-02  1.882e-02  -1.638    0.112
    ## 
    ## Residual standard error: 8.754 on 30 degrees of freedom
    ## Multiple R-squared:  0.0822, Adjusted R-squared:  0.02102 
    ## F-statistic: 1.343 on 2 and 30 DF,  p-value: 0.2762

``` r
summary(m.DC)
```

    ## 
    ## Call:
    ## lm(formula = bloom_doy ~ year + I(year^2), data = DC[DC$year > 
    ##     1988, ])
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -14.4726  -4.5964  -0.3476   4.4280  11.3005 
    ## 
    ## Coefficients:
    ##               Estimate Std. Error t value Pr(>|t|)
    ## (Intercept) -6.380e+04  5.914e+04  -1.079    0.289
    ## year         6.375e+01  5.899e+01   1.081    0.288
    ## I(year^2)   -1.590e-02  1.471e-02  -1.081    0.288
    ## 
    ## Residual standard error: 6.843 on 30 degrees of freedom
    ## Multiple R-squared:  0.03831,    Adjusted R-squared:  -0.0258 
    ## F-statistic: 0.5976 on 2 and 30 DF,  p-value: 0.5565

``` r
nine.year<-data.frame(kyoto=m.kyoto$coefficients, liestal=m.liestal$coefficients, DC=m.DC$coefficients)
write.csv(nine.year, "9yearcoef.csv")
```
