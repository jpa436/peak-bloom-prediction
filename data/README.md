# Cleaned data sets : clean_data.csv

**Note that in the ipynb files, the name of this file is lee.csv**

Table 1. List of weather variables created for possible inclusion in the predictive model.   




Latitude of the location 

Longitude of the location 

Altitude of the location 

Year of observation 

Day of year when the 14 days moving average of average temperature is at the minimum 

Day of year when the 14 days moving average of maximum temperature is at the minimum 

Day of year when the 14 days moving average of minimum temperature is at the minimum 

The daily average temperature of the day 14-days-moving average reached the minimum 

The daily minimum temperature of the day 14-days-moving average reached the minimum 

The daily maximum temperature of the day 14-days-moving average reached the minimum 

Number of days in October, November, December, January, February with precipitation, separately 

Average temperature of October, November, December, January, February, March, separately 

The maximum temperature of October, November, December, January, February, March, separately 

The minimum temperature of October, November, December, January, February, March, separately 

Square of cosine of longitude   

The slope coefficient of the linear fit between day of year and average temperature in a period between the coldest period and March 10. It captures the speed of temperature change 

The squared term coefficient of the linear fit between day of year and the squared average temperature in a period between the coldest period and March 10. It captures the acceleration of temperature change 

The intercept term coefficient of the linear fit between day of year and the squared average temperature in a period between the coldest period and March 10.  



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
* *Date_doy_tavg* doy at which 14 days moving average of `tavg` is at its minimum (`integer`). January 1st is `1` and December 30th and 31st are `-1` and `0` respectively.
* *Date_doy_tmax* doy at which 14 days moving average of `tmax` is at its minimum (`integer`). January 1st is `1` and December 30th and 31st are `-1` and `0` respectively.
* *Date_doy_tmin* doy at which 14 days moving average of `tmin` is at its minimum (`integer`). January 1st is `1` and December 30th and 31st are `-1` and `0` respectively.
* *tavg_below_5* the number of days when the tavg is below 5 Celsius degree between October 1st of the previous year and March 10
* *tavg_above_10* the number of days when the tavg is above 10 Celsius degree between October 1st of the previous year and March 10
* *tavg_moving* the 14 days moving average of tavg at *Date_doy_tavg*
* *tmin_moving* the 14 days moving average of tavg at *Date_doy_tmin*
* *tmax_moving* the 14 days moving average of tavg at *Date_doy_tmax*
* *prcp.n* the number of days when precipitation is above 0 for month n {October: -2, November : -1, December : 0, January : 1, February : 2, March (1st - 10th) : 3}
* *tavg.n* the average `tavg` for month n {October: -2, November : -1, December : 0, January : 1, February : 2, March (1st - 10th) : 3}
* *tmin.n* the average `tmin` for month n {October: -2, November : -1, December : 0, January : 1, February : 2, March (1st - 10th): 3}
* *tmax.n* the average `tmax` for month n {October: -2, November : -1, December : 0, January : 1, February : 2, March (1st - 10th): 3}



## Data sources

George Mason’s Department of Statistics cherry blossom peak bloom prediction competitio github repo: https://github.com/GMU-CherryBlossomCompetition/peak-bloom-prediction

`rnoaa` R-package: https://github.com/ropensci/rnoaa
