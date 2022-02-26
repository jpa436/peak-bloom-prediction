Obtaining meteorology data from rnoaa package
================

In this tutorial, we would like to show how we obtained the weather data
through rnoaa package. As a demonstration, we would like to show how we
collected data for <tt>korea.csv</tt>. Let’s take a look at the first
several rows of the data.

``` r
knitr::kable(head(korea))
```

| location           |      lat |     long |   alt | year | bloom\_date | bloom\_doy |
|:-------------------|---------:|---------:|------:|-----:|:------------|-----------:|
| South Korea/Sokcho | 38.25085 | 128.5647 | 18.06 | 1980 | 1980-04-21  |        112 |
| South Korea/Sokcho | 38.25085 | 128.5647 | 18.06 | 1981 | 1981-04-19  |        109 |
| South Korea/Sokcho | 38.25085 | 128.5647 | 18.06 | 1982 | 1982-04-12  |        102 |
| South Korea/Sokcho | 38.25085 | 128.5647 | 18.06 | 1983 | 1983-04-18  |        108 |
| South Korea/Sokcho | 38.25085 | 128.5647 | 18.06 | 1984 | 1984-05-07  |        128 |
| South Korea/Sokcho | 38.25085 | 128.5647 | 18.06 | 1985 | 1985-04-22  |        112 |

Given latitude and longitude information, we would like to find weather
stations, which have historical weather data. To find the weather
stations, first we need to get stations information using
<tt>ghcnd\_stations</tt> function from <tt>rnoaa</tt> package. The data
retrieved contains the basic information of each weather stations.

``` r
stations <- ghcnd_stations()
knitr::kable(head(stations))
```

| id          | latitude | longitude | elevation | state | name                  | gsn\_flag | wmo\_id | element | first\_year | last\_year |
|:------------|---------:|----------:|----------:|:------|:----------------------|:----------|:--------|:--------|------------:|-----------:|
| ACW00011604 |  17.1167 |  -61.7833 |      10.1 |       | ST JOHNS COOLIDGE FLD |           |         | TMAX    |        1949 |       1949 |
| ACW00011604 |  17.1167 |  -61.7833 |      10.1 |       | ST JOHNS COOLIDGE FLD |           |         | TMIN    |        1949 |       1949 |
| ACW00011604 |  17.1167 |  -61.7833 |      10.1 |       | ST JOHNS COOLIDGE FLD |           |         | PRCP    |        1949 |       1949 |
| ACW00011604 |  17.1167 |  -61.7833 |      10.1 |       | ST JOHNS COOLIDGE FLD |           |         | SNOW    |        1949 |       1949 |
| ACW00011604 |  17.1167 |  -61.7833 |      10.1 |       | ST JOHNS COOLIDGE FLD |           |         | SNWD    |        1949 |       1949 |
| ACW00011604 |  17.1167 |  -61.7833 |      10.1 |       | ST JOHNS COOLIDGE FLD |           |         | PGTM    |        1949 |       1949 |

There is another function, <tt>meteo\_nearby\_stations</tt>, which
allows us to find nearby station given latitude and longitude
information. However, to use the function, it requires a dataframe with
the certain column names; ‘id’,‘latitude’, and ‘longitude’. Hence, we
will first select the first three columns from korea and name it
<tt>korea\_latlong</tt> and change the column names. Then we select the
unique set of longitude and latitude to avoid searching for the same
location multiple times.

``` r
korea_latlong = korea[,1:3]
colnames(korea_latlong) = c('id','latitude','longitude')
korea_latlong = korea_latlong %>% unique()
knitr::kable(head(korea_latlong))
```

|     | id                        | latitude | longitude |
|:----|:--------------------------|---------:|----------:|
| 1   | South Korea/Sokcho        | 38.25085 |  128.5647 |
| 19  | South Korea/Daegwallyeong | 37.67713 |  128.7183 |
| 26  | South Korea/Chuncheon     | 37.90256 |  127.7357 |
| 46  | South Korea/Gangneung     | 37.75147 |  128.8910 |
| 61  | South Korea/Seoul         | 37.57142 |  126.9658 |
| 86  | South Korea/Incheon       | 37.47772 |  126.6249 |

Now passing the <tt>korea\_latlong</tt> into the
<tt>meteo\_nearby\_stations</tt>, you can find a list of available
stations for each of the latitude and longitude value. Not all cities
have a weather station in 20km radius. Among the cities with available
weather station, we will show you Yeosu as an example.

``` r
result = meteo_nearby_stations(korea_latlong, station_data = stations, radius = 20, year_min = 1990) 
knitr::kable(result['South Korea/Yeosu'])
```

<table class="kable_wrapper">
<tbody>
<tr>
<td>

| id          | name  | latitude | longitude | distance |
|:------------|:------|---------:|----------:|---------:|
| KSM00047168 | YEOSU |   34.733 |    127.75 | 1.104863 |

</td>
</tr>
</tbody>
</table>

The result shows that there is one station 1.1km away from the latitude
and longitude we have for the city of Yeosu. Availability of the nearby
weather station does not assure that the weather station store
information we would like to explore. In our case, we want to obtain
‘tavg’,‘tmin’,‘tmax’, and ‘prcp’. Due to our imputation method, it is
okay to contain two of the temperature variables. Using for-loop, we
will explore each of the weather station and check if they provide at
least three of the four variables. After the for-loop, it seems like the
station near Yeosu has met our criteria.

``` r
for(i in 1:length(result)){
  output = NULL
  if(dim(result[[i]])[1] > 1){
    station_ids = result[[i]]$id
    for (stationid in station_ids){
      list_df = ghcnd_search(stationid = stationid, var = c("tmax",'tmin','prcp','tavg'),
                             date_min = "1951-10-01", date_max = "2022-02-28")
      N = attributes(list_df)$names %>% length()
      N
      if(N>3){
        output = c(output, stationid)
      }
    }
    if(length(output) == 0){
      result[i] = NULL
    }
    else{
      result[[i]] = result[[i]][result[[i]]$id == output[1],]
    }
  }
}
knitr::kable(result['South Korea/Yeosu'])
```

<table class="kable_wrapper">
<tbody>
<tr>
<td>

| id          | name  | latitude | longitude | distance |
|:------------|:------|---------:|----------:|---------:|
| KSM00047168 | YEOSU |   34.733 |    127.75 | 1.104863 |

</td>
</tr>
</tbody>
</table>

Now back to korea\_latlong table, we would like to narrow down to the
list of cities where nearby weather station, which meets our criteria,
exists. To do so, we created ‘exist’ variable which indicates whether
the weather station is available and selected only those rows that are
TRUE for the variable. We also add a variable ‘stationid’ which
indicates the unique weather station id in the ‘result’ dataframe for
each of the corresponding cities. Finally, using the right merge, in the
‘korea’ dataframe, we have only the cities with available weather
station and its weather station id.

``` r
korea_latlong['exist'] = sapply(korea_latlong$id, function(x){dim(result[[as.character(x)]])[1] > 0}) %>% unlist()
korea_latlong = korea_latlong[korea_latlong$exist, ]
korea_latlong['stationid']  = sapply(korea_latlong$id, function(x){result[[as.character(x)]]$id   })
# we need to change the column names of the korea_latlong
colnames(korea_latlong)[1:3] = c('location', 'lat','long')
# merge
korea = left_join(x= korea, y = korea_latlong, by = c('location','lat','long')) %>% na.omit() %>% 
  dplyr::select(c('stationid','location','lat','long','alt','year','bloom_date','bloom_doy'))
knitr::kable(head(korea))
```

|     | stationid   | location                  |      lat |     long |    alt | year | bloom\_date | bloom\_doy |
|:----|:------------|:--------------------------|---------:|---------:|-------:|-----:|:------------|-----------:|
| 19  | KSM00047105 | South Korea/Daegwallyeong | 37.67713 | 128.7183 | 772.57 | 1982 | 1982-05-12  |        132 |
| 20  | KSM00047105 | South Korea/Daegwallyeong | 37.67713 | 128.7183 | 772.57 | 1989 | 1989-06-15  |        166 |
| 21  | KSM00047105 | South Korea/Daegwallyeong | 37.67713 | 128.7183 | 772.57 | 1990 | 1990-05-25  |        145 |
| 22  | KSM00047105 | South Korea/Daegwallyeong | 37.67713 | 128.7183 | 772.57 | 1991 | 1991-05-21  |        141 |
| 23  | KSM00047105 | South Korea/Daegwallyeong | 37.67713 | 128.7183 | 772.57 | 1992 | 1992-05-02  |        123 |
| 24  | KSM00047105 | South Korea/Daegwallyeong | 37.67713 | 128.7183 | 772.57 | 1993 | 1993-05-15  |        135 |

Now, going over each of the city, we will collect weather information
from the corresponding weather stations defined by the ‘stationid’
column. We will create a dataframe, ‘output’, which contain all the
weather information between 1951 and early 2022. Even though we set the
begin date on October 1951, it returns either this or the earliest
avaiable date.

``` r
output = NULL
for( stationid in unique(korea$stationid)){
  list_df = ghcnd_search(stationid = stationid, var = c("tmax",'tmin','prcp','tavg'), 
                         date_min = "1951-10-01", date_max = "2022-02-28")
  
  N = attributes(list_df)$names %>% length()
  df = list_df[[1]]
  df
  N
  if(N > 1){
  for(i in 2:N){
    df = left_join(x = df, y = list_df[[i]], by = c('id','date'))
  }}
  #df = df %>% dplyr::select(c('id','date','tmax','tmin','prcp'))
  
  df = df %>% dplyr::select(c('id','date',attributes(list_df)$names))
  colnames(df)[1] = 'stationid'
  output = plyr::rbind.fill(output, df)
}
knitr::kable(head(output))
```

| stationid   | date       | tmax | tmin | prcp | tavg |
|:------------|:-----------|-----:|-----:|-----:|-----:|
| KSM00047105 | 1973-01-01 |   NA |   NA |   NA |   43 |
| KSM00047105 | 1973-01-02 |   NA |   NA |   NA |  -54 |
| KSM00047105 | 1973-01-03 |   NA |   NA |   NA |  -27 |
| KSM00047105 | 1973-01-04 |   NA |   NA |   NA |    9 |
| KSM00047105 | 1973-01-05 |   90 |    0 |    0 |   27 |
| KSM00047105 | 1973-01-06 |   70 |  -10 |    0 |   27 |

Before merging this table with the ‘korea’ table, we need to mutate the
variables. December is going to be 0 and October and November of the
previous year is going to be -2 and -1 respectively. Also, for October,
November, and December, their year value increase by 1.

``` r
output = output %>% mutate(year = as.integer(format(date, "%Y")),
                           month = as.integer(strftime(date, '%m')) %% 12,
                           month = if_else(month %in% c(10, 11), month -12, month),
                           year = if_else(month <= 0, year + 1L, year))
knitr::kable(head(output))
```

| stationid   | date       | tmax | tmin | prcp | tavg | year | month |
|:------------|:-----------|-----:|-----:|-----:|-----:|-----:|------:|
| KSM00047105 | 1973-01-01 |   NA |   NA |   NA |   43 | 1973 |     1 |
| KSM00047105 | 1973-01-02 |   NA |   NA |   NA |  -54 | 1973 |     1 |
| KSM00047105 | 1973-01-03 |   NA |   NA |   NA |  -27 | 1973 |     1 |
| KSM00047105 | 1973-01-04 |   NA |   NA |   NA |    9 | 1973 |     1 |
| KSM00047105 | 1973-01-05 |   90 |    0 |    0 |   27 | 1973 |     1 |
| KSM00047105 | 1973-01-06 |   70 |  -10 |    0 |   27 | 1973 |     1 |

Finally, using inner\_join, we obtain the final table.

``` r
final = inner_join(x = korea, y = output, by = c('stationid','year'))

knitr::kable(head(final))
```

| stationid   | location                  |      lat |     long |    alt | year | bloom\_date | bloom\_doy | date       | tmax | tmin | prcp | tavg | month |
|:------------|:--------------------------|---------:|---------:|-------:|-----:|:------------|-----------:|:-----------|-----:|-----:|-----:|-----:|------:|
| KSM00047105 | South Korea/Daegwallyeong | 37.67713 | 128.7183 | 772.57 | 1982 | 1982-05-12  |        132 | 1981-10-01 |  240 |  150 |   NA |  188 |    -2 |
| KSM00047105 | South Korea/Daegwallyeong | 37.67713 | 128.7183 | 772.57 | 1982 | 1982-05-12  |        132 | 1981-10-02 |  210 |  140 |   NA |  163 |    -2 |
| KSM00047105 | South Korea/Daegwallyeong | 37.67713 | 128.7183 | 772.57 | 1982 | 1982-05-12  |        132 | 1981-10-03 |  210 |   NA |   NA |  155 |    -2 |
| KSM00047105 | South Korea/Daegwallyeong | 37.67713 | 128.7183 | 772.57 | 1982 | 1982-05-12  |        132 | 1981-10-04 |  200 |   90 |   NA |  162 |    -2 |
| KSM00047105 | South Korea/Daegwallyeong | 37.67713 | 128.7183 | 772.57 | 1982 | 1982-05-12  |        132 | 1981-10-05 |  250 |  140 |   NA |  196 |    -2 |
| KSM00047105 | South Korea/Daegwallyeong | 37.67713 | 128.7183 | 772.57 | 1982 | 1982-05-12  |        132 | 1981-10-06 |  220 |  150 |   NA |  173 |    -2 |
