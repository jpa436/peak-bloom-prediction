# Cleaned data sets : clean_data.csv

**Note that in the ipynb files, the name of this file is lee.csv**

Table 1. List of weather variables created for possible inclusion in the predictive model.   


The structure of the cleaned data files is as follows:

* _location_ location identifier (`string`)
* _lat_ (approximate) latitude of the observation (`double`).
* _long_ (approximate) longitude of the observation (`double`).
* _long.x_ cosine of _long_ (`double`).
* _long.y_ sine of _long_ (`double`).
* _alt_ (approximate) altitude of the observation (`double`).
* _year_ year of the observation (`integer`).
* *bloom_date* date of peak bloom of the cherry trees (ISO 8601 date `string`). The "peak bloom date" may be defined differently for different locations.
* *bloom_doy* days since January 1st of the year until peak bloom (`integer`). January 1st is `1`.
* *Date_doy_tavg* Day of year when the 14 days moving average of average temperature is at the minimum 
* *Date_doy_tmax* Day of year when the 14 days moving average of maximum temperature is at the minimum 
* *Date_doy_tmin* Day of year when the 14 days moving average of minimum temperature is at the minimum 
* *tavg_below_5* the number of days when the tavg is below 5 Celsius degree between October 1st of the previous year and March 10
* *tavg_above_10* the number of days when the tavg is above 10 Celsius degree between October 1st of the previous year and March 10
* *tavg_moving* The daily average temperature of the day 14-days-moving average reached the minimum 
* *tmin_moving* The daily minimum temperature of the day 14-days-moving average reached the minimum 
* *tmax_moving* The daily maximum temperature of the day 14-days-moving average reached the minimum 
* *prcp.n* the number of days when precipitation is above 0 for month n {October: -2, November : -1, December : 0, January : 1, February : 2, March (1st - 10th) : 3}
* *tavg.n* Average temperature of October, November, December, January, February, March (through March 10), separately -> the average `tavg` for month n {October: -2, November : -1, December : 0, January : 1, February : 2, March (1st - 10th) : 3}
* *tmin.n* The minimum temperature of October, November, December, January, February, March (through March 10), separately 
 -> the average `tmin` for month n {October: -2, November : -1, December : 0, January : 1, February : 2, March (1st - 10th): 3}
* *tmax.n* The maximum temperature of October, November, December, January, February, March (through March 10), separately -> the average `tmax` for month n {October: -2, November : -1, December : 0, January : 1, February : 2, March (1st - 10th): 3}
* *tmax.n*
* *slope* The slope coefficient of the linear fit between day of year and average temperature in a period between the coldest period and March 10. It captures the speed of temperature change 
* *intc* The intercept term coefficient of the linear fit between day of year and the squared average temperature in a period between the coldest period and March 10.  
* *dg2_coef* The squared term coefficient of the linear fit between day of year and the squared average temperature in a period between the coldest period and March 10. It captures the acceleration of temperature change 


In the analysis, there are some interaction terms considered:

* 

## Data sources

George Mason’s Department of Statistics cherry blossom peak bloom prediction competitio github repo: https://github.com/GMU-CherryBlossomCompetition/peak-bloom-prediction

`rnoaa` R-package: https://github.com/ropensci/rnoaa
