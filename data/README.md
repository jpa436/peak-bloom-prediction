# Cleaned data sets : cleaned_data.csv

The structure of the cleaned data files is as follows:

* _location_ location identifier (`string`)
* _lat_ (approximate) latitude of the observation (`double`).
* _long_ (approximate) longitude of the observation (`double`).
* _alt_ (approximate) altitude of the observation (`double`).
* _year_ year of the observation (`integer`).
* *bloom_date* date of peak bloom of the cherry trees (ISO 8601 date `string`). The "peak bloom date" may be defined differently for different locations.
* *bloom_doy* days since January 1st of the year until peak bloom (`integer`). January 1st is `1`.

## Data sources

George Mason’s Department of Statistics cherry blossom peak bloom prediction competitio github repo: https://github.com/GMU-CherryBlossomCompetition/peak-bloom-prediction

`rnoaa` R-package: https://github.com/ropensci/rnoaa
