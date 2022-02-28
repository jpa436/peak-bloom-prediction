## Imputing NA values for weather variables

Cherry blossom: Imputation of NA
================

<!--   rmarkdown::github_document
    code_folding: hide
    code_download: true -->

In this page, I will demonstrate how we imputed NAs in our dataset.
Let’s see first few rows of <tt>korea\_complete</tt> dataset.

There are several <tt>NA</tt>s in <tt>tmin</tt> and <tt>prec</tt> data.
We would like to replace these <tt>NA</tt>s with reasonable numbers.

``` r
knitr::kable(head(korea_complete))
```

| stationid   | location                  |      lat |     long |    alt | year | bloom\_date | bloom\_doy | date       | tmax | tmin | prcp | tavg | month | rollong\_year |
|:------------|:--------------------------|---------:|---------:|-------:|-----:|:------------|-----------:|:-----------|-----:|-----:|-----:|-----:|------:|--------------:|
| KSM00047105 | South Korea/Daegwallyeong | 37.67713 | 128.7183 | 772.57 | 1982 | 1982-05-12  |        132 | 1981-10-01 |  240 |  150 |   NA |  188 |    -2 |          1983 |
| KSM00047105 | South Korea/Daegwallyeong | 37.67713 | 128.7183 | 772.57 | 1982 | 1982-05-12  |        132 | 1981-10-02 |  210 |  140 |   NA |  163 |    -2 |          1983 |
| KSM00047105 | South Korea/Daegwallyeong | 37.67713 | 128.7183 | 772.57 | 1982 | 1982-05-12  |        132 | 1981-10-03 |  210 |   NA |   NA |  155 |    -2 |          1983 |
| KSM00047105 | South Korea/Daegwallyeong | 37.67713 | 128.7183 | 772.57 | 1982 | 1982-05-12  |        132 | 1981-10-04 |  200 |   90 |   NA |  162 |    -2 |          1983 |
| KSM00047105 | South Korea/Daegwallyeong | 37.67713 | 128.7183 | 772.57 | 1982 | 1982-05-12  |        132 | 1981-10-05 |  250 |  140 |   NA |  196 |    -2 |          1983 |
| KSM00047105 | South Korea/Daegwallyeong | 37.67713 | 128.7183 | 772.57 | 1982 | 1982-05-12  |        132 | 1981-10-06 |  220 |  150 |   NA |  173 |    -2 |          1983 |

We divided cases of missing data:

<b>1) When one of <tt>tmax</tt>, <tt>tmin</tt>, and <tt>tavg</tt> is
missing. </b>

We replaced the missing values with regression prediction. For instance,
to replace the following missing values, we ran linear regression with
<tt>tmin</tt> on <tt>tmax</tt> and <tt>tavg</tt>. In the following case,
we replaced the first <tt>NA</tt> of <tt>tmin</tt> with

*t**m**i**n* =  − 20.375867 − 0.505408 × *t**m**a**x* + 1.473154 × *t**a**v**g*
.

We did the same regression to replace <tt>tmax</tt> and <tt>tavg</tt>.
We used the data of each country when we regress.

``` r
knitr::kable(korea_complete[c(7,8),])
```

| stationid   | location                  |      lat |     long |    alt | year | bloom\_date | bloom\_doy | date       | tmax | tmin | prcp | tavg | month | rollong\_year |
|:------------|:--------------------------|---------:|---------:|-------:|-----:|:------------|-----------:|:-----------|-----:|-----:|-----:|-----:|------:|--------------:|
| KSM00047105 | South Korea/Daegwallyeong | 37.67713 | 128.7183 | 772.57 | 1982 | 1982-05-12  |        132 | 1981-10-07 |  230 |   NA |   NA |  165 |    -2 |          1983 |
| KSM00047105 | South Korea/Daegwallyeong | 37.67713 | 128.7183 | 772.57 | 1982 | 1982-05-12  |        132 | 1981-10-08 |  210 |   NA |   NA |  163 |    -2 |          1983 |

``` r
summary(lm(data=korea_complete, tmin~tmax+tavg))
```

    ## 
    ## Call:
    ## lm(formula = tmin ~ tmax + tavg, data = korea_complete)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -125.172  -11.687    1.851   13.668   86.037 
    ## 
    ## Coefficients:
    ##               Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) -20.375867   0.315025  -64.68   <2e-16 ***
    ## tmax         -0.505408   0.006143  -82.27   <2e-16 ***
    ## tavg          1.473154   0.006524  225.81   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 19.69 on 29480 degrees of freedom
    ##   (17072 observations deleted due to missingness)
    ## Multiple R-squared:  0.9174, Adjusted R-squared:  0.9174 
    ## F-statistic: 1.637e+05 on 2 and 29480 DF,  p-value: < 2.2e-16

<b>2) When two values are missing in one row.</b> Let’s see the
following case. In the following case, both <tt>tmax</tt> and
<tt>tmin</tt> are missing. In this case, we used replaced the missing
value with moving average method. In the case below, the <tt>tmax</tt>
was replaced by (236 + 190)/2 = 213. We used moving average 4, and used
exponential weights.

``` r
knitr::kable(korea_complete[c(30229:30231),])
```

| stationid   | location          |      lat |    long |   alt | year | bloom\_date | bloom\_doy | date       | tmax | tmin | prcp | tavg | month | rollong\_year |
|:------------|:------------------|---------:|--------:|------:|-----:|:------------|-----------:|:-----------|-----:|-----:|-----:|-----:|------:|--------------:|
| KSM00047159 | South Korea/Busan | 35.10468 | 129.032 | 69.56 | 1989 | 1989-04-03  |         93 | 1989-04-22 |  236 |  149 |   NA |  182 |     4 |          1990 |
| KSM00047159 | South Korea/Busan | 35.10468 | 129.032 | 69.56 | 1989 | 1989-04-03  |         93 | 1989-04-23 |   NA |   NA |   NA |  159 |     4 |          1990 |
| KSM00047159 | South Korea/Busan | 35.10468 | 129.032 | 69.56 | 1989 | 1989-04-03  |         93 | 1989-04-24 |  190 |   NA |   71 |  129 |     4 |          1990 |

<b>3) When <tt>prec</tt> is missing, we replaced it with 0. </b>

<tt>Temp$prcp\[is.na(Temp$prcp)\]&lt;-0</tt>

``` r
############## Imputation of NAs
filenames<-c("korea_complete","kyoto_complete","switzerland_complete", "washingtondc_complete", "japan_complete", "us_complete")

for(k in 1:length(filenames)){
  Temp<-get(filenames[k]) # define dataset here
  print(paste0(filenames[k], " Start!"))

  ## Check if the variable has different name.
  tmin.v<-Temp$tmin
  tmax.v<-Temp$tmax
  tavg.v<-Temp$tavg
  null.v<-c(is.null(tmin.v), is.null(tmax.v), is.null(tavg.v))
  if(sum(null.v)>0){
    print(paste0(filenames[k], " was not executed"))
  }else{
    # regress tmax, tmin, tavg on the others
    md.max<-lm(data=Temp, tmax~tmin+tavg)
    md.min<-lm(data=Temp, tmin~tmax+tavg)
    md.avg<-lm(data=Temp, tavg~tmax+tmin)
    
    for(i in 1:nrow(Temp)){
      df<-Temp[i,]
      tmin<-df$tmin
      tmax<-df$tmax
      tavg<-df$tavg
      NA.arr<-is.na(c(tmin, tmax, tavg))
      if(sum(NA.arr==c(1,0,0))==3){
        # replace missing tmin with prediction from the regression
        tmin<-predict(md.min,data.frame(tmax=tmax,tavg=tavg))
      }
      if(sum(NA.arr==c(0,1,0))==3){
         # replace missing tmin with prediction from the regression
        tmax<-predict(md.max,data.frame(tmin=tmin,tavg=tavg))
      }
      if(sum(NA.arr==c(0,0,1))==3){
         # replace missing tmin with prediction from the regression
        tavg<-predict(md.avg,data.frame(tmax=tmax,tmin=tmin))
      }
      Temp$tmin[i]<-tmin
      Temp$tmax[i]<-tmax
      Temp$tavg[i]<-tavg
      if(i%%10000==0){
        print(i)
      }
    }
    ## impute missing variables with MA(4), with exponential weight
    Temp$tavg<-imputeTS::na_ma(Temp$tavg)
    Temp$tmin<-imputeTS::na_ma(Temp$tmin)
    Temp$tmax<-imputeTS::na_ma(Temp$tmax)
    ## impute missing variable when prcp is missing. 
    Temp$prcp[is.na(Temp$prcp)]<-0
    ## write out the processed dataset
    write_csv(Temp, paste0(filenames[k], "_na_imputed.csv"))
    print(paste0(filenames[k], " Finished!"))
  }
}
```

<b>4) For Liestal, we could not attain <tt>tavg</tt> at all. </b> Hence,
we averaged <tt>tmin</tt> and <tt>tmax</tt> to approximate
<tt>tavg</tt>.

<tt>Temp$tavg&lt;-0.5\*Temp$tmax+0.5\*Temp$tmin</tt>

``` r
##################liestal
filenames<-"liestal_complete"
  Temp<-get(filenames) # define dataset here
  print(paste0(filenames[k], " Start!"))
  
  Temp$tavg<-0.5*Temp$tmax+0.5*Temp$tmin
  md.max<-lm(data=Temp, tmax~tmin+tavg)
  md.min<-lm(data=Temp, tmin~tmax+tavg)
  md.avg<-lm(data=Temp, tavg~tmax+tmin)
  
  #summary(md.max)
  
  for(i in 1:nrow(Temp)){
    df<-Temp[i,]
    tmin<-df$tmin
    tmax<-df$tmax
    tavg<-df$tavg
    NA.arr<-is.na(c(tmin, tmax, tavg))
    if(sum(NA.arr==c(1,0,0))==3){
      tmin<-predict(md.min,data.frame(tmax=tmax,tavg=tavg))
    }
    if(sum(NA.arr==c(0,1,0))==3){
      tmax<-predict(md.max,data.frame(tmin=tmin,tavg=tavg))
    }
    if(sum(NA.arr==c(0,0,1))==3){
      tavg<-predict(md.avg,data.frame(tmax=tmax,tmin=tmin))
    }
    Temp$tmin[i]<-tmin
    Temp$tmax[i]<-tmax
    Temp$tavg[i]<-tavg
    Temp$prcp[is.na(Temp$prcp)]<-0
    if(i%%10000==0){
      print(i)
    }
  }
  ## impute missing variables with MA(4), with exponential weight
  
  Temp$tavg<-imputeTS::na_ma(Temp$tavg)
  Temp$tmin<-imputeTS::na_ma(Temp$tmin)
  Temp$tmax<-imputeTS::na_ma(Temp$tmax)
  
  write_csv(Temp, paste0(filenames[k], "_na_imputed.csv"))
  print(paste0(filenames[k], " Finished!"))
```
