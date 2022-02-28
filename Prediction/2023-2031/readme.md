Prediction for 2023 - 2031
=====


We used the simple linear regression (`bloom_doy ~ year + I(year^2)`) to predict the bloom_doy for Washington DC, Kyoto, and Liestal.
For Vancouver, we fit the GAM with the data from all other cities available with the features: year, latitude, longitude, and altitude. 
