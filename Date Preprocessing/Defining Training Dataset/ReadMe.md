```python
import numpy as np
import pandas as pd
import os
os.chdir('./data')
import scipy
import matplotlib.pyplot as plt
```

In this jupyter notebook file, we show step-by-step process of creating variables to be used. First of all, we obtained the meteorology data from `ronaa` package and selected 5 cities from the four countries; the US, Switzerland, Japan, and Korea. The concatenated data is saved as `longdata.csv`.


```python
df = pd.read_csv('./longdata.csv')
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>location</th>
      <th>year</th>
      <th>stationid</th>
      <th>lat</th>
      <th>long</th>
      <th>alt</th>
      <th>bloom_date</th>
      <th>bloom_doy</th>
      <th>date</th>
      <th>tmax</th>
      <th>tmin</th>
      <th>prcp</th>
      <th>tavg</th>
      <th>month</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>29890</td>
      <td>2019</td>
      <td>CA001108910</td>
      <td>48.91993</td>
      <td>-122.640564</td>
      <td>10.0</td>
      <td>5/11/2019</td>
      <td>131</td>
      <td>10/1/2018</td>
      <td>178.0</td>
      <td>125.0</td>
      <td>50.0</td>
      <td>151.0</td>
      <td>-2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>29890</td>
      <td>2019</td>
      <td>CA001108910</td>
      <td>48.91993</td>
      <td>-122.640564</td>
      <td>10.0</td>
      <td>5/11/2019</td>
      <td>131</td>
      <td>10/2/2018</td>
      <td>153.0</td>
      <td>69.0</td>
      <td>6.0</td>
      <td>111.0</td>
      <td>-2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>29890</td>
      <td>2019</td>
      <td>CA001108910</td>
      <td>48.91993</td>
      <td>-122.640564</td>
      <td>10.0</td>
      <td>5/11/2019</td>
      <td>131</td>
      <td>10/3/2018</td>
      <td>122.0</td>
      <td>43.0</td>
      <td>0.0</td>
      <td>83.0</td>
      <td>-2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>29890</td>
      <td>2019</td>
      <td>CA001108910</td>
      <td>48.91993</td>
      <td>-122.640564</td>
      <td>10.0</td>
      <td>5/11/2019</td>
      <td>131</td>
      <td>10/4/2018</td>
      <td>135.0</td>
      <td>21.0</td>
      <td>0.0</td>
      <td>78.0</td>
      <td>-2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>29890</td>
      <td>2019</td>
      <td>CA001108910</td>
      <td>48.91993</td>
      <td>-122.640564</td>
      <td>10.0</td>
      <td>5/11/2019</td>
      <td>131</td>
      <td>10/5/2018</td>
      <td>151.0</td>
      <td>45.0</td>
      <td>2.0</td>
      <td>98.0</td>
      <td>-2</td>
    </tr>
  </tbody>
</table>
</div>



We first define `Date_doy` which is the day of year. For October, November, and December of the previous year, their doy would be subtracted by 365 so that they are number of days from the beginning of the following year.


```python
def define_doy(char):
    doy = pd.Period(char, freq='D').day_of_year
    return(doy)

df['Date_doy'] = df.date.apply(define_doy)
```


```python
df.Date_doy = [x if x < 250 else x-365 for x in df.Date_doy]
```

Then we define the four new variables `tavg_moving`, `tmin_moving`, `tmax_moving`, and `tavg_below_5`.
The first three variables are the 14 days moving averages of the temperature data, and tavg_below_5 finds number of accumulate days when its `tavg` is below 5


```python
# 14 days rolling moving average of temperature data and total number of days having tavg below 5
df['tavg_moving'] = df.groupby(['location','year'])['tavg'].transform(lambda x: x.rolling(14, 14).mean()).round(4)
df['tmin_moving'] = df.groupby(['location','year'])['tmin'].transform(lambda x: x.rolling(14, 14).mean()).round(4)
df['tmax_moving'] = df.groupby(['location','year'])['tmax'].transform(lambda x: x.rolling(14, 14).mean()).round(4)
df['tavg_below_5'] = df.groupby(['location','year'])['tavg'].transform(lambda x: x.le(5).sum())
```

We also define `tavg_above_10`, which is the number of days when `tavg` is above 10 between December and March 10.


```python
df['tavg_above_10'] = df.tavg.ge(10)
new_above_5 = df.loc[df.Date_doy.lt(70) & df.Date_doy.gt(-30),:].groupby(['location','year'])['tavg_above_10'].agg(sum).reset_index()
df = df.drop(['tavg_above_10'], axis = 1)
df = df.merge(new_above_5, on = ['location','year'], how = 'left')
```

For `tavg_moving`, `tmin_moving`, and `tmax_moving`, our interest is to find the minimum of it and the doy of the last day of the 14 days period. To extract the information, we group by location and year, and found the minimum


```python
minimums = df.groupby(['location','year'])[['location','year','tavg_moving','tmin_moving','tmax_moving']].transform(lambda x: x.min()).drop_duplicates().reset_index(drop = True)
minimums
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>location</th>
      <th>year</th>
      <th>tavg_moving</th>
      <th>tmin_moving</th>
      <th>tmax_moving</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>29890</td>
      <td>2019</td>
      <td>-8.5714</td>
      <td>-41.9286</td>
      <td>24.8571</td>
    </tr>
    <tr>
      <th>1</th>
      <td>29890</td>
      <td>2020</td>
      <td>10.5714</td>
      <td>-19.9286</td>
      <td>41.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>29890</td>
      <td>2021</td>
      <td>-27.9626</td>
      <td>-50.7796</td>
      <td>-5.5571</td>
    </tr>
    <tr>
      <th>3</th>
      <td>32789</td>
      <td>2019</td>
      <td>-24.7483</td>
      <td>-71.4286</td>
      <td>12.5714</td>
    </tr>
    <tr>
      <th>4</th>
      <td>32789</td>
      <td>2020</td>
      <td>17.6321</td>
      <td>-15.0</td>
      <td>51.2857</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>754</th>
      <td>washingtondc</td>
      <td>2017</td>
      <td>23.6429</td>
      <td>-12.5</td>
      <td>58.9286</td>
    </tr>
    <tr>
      <th>755</th>
      <td>washingtondc</td>
      <td>2018</td>
      <td>-52.7413</td>
      <td>-86.4286</td>
      <td>-17.1429</td>
    </tr>
    <tr>
      <th>756</th>
      <td>washingtondc</td>
      <td>2019</td>
      <td>-2.5</td>
      <td>-50.6429</td>
      <td>41.7857</td>
    </tr>
    <tr>
      <th>757</th>
      <td>washingtondc</td>
      <td>2020</td>
      <td>30.7857</td>
      <td>-6.2143</td>
      <td>71.7857</td>
    </tr>
    <tr>
      <th>758</th>
      <td>washingtondc</td>
      <td>2021</td>
      <td>7.7857</td>
      <td>-15.2143</td>
      <td>36.9286</td>
    </tr>
  </tbody>
</table>
<p>759 rows × 5 columns</p>
</div>



`mindoy_tavg`, `mindoy_tmin`, and `mindoy_tmax` indicate whether the doy of the 14 days period when its average `tavg`, `tmin`, and `tmax` are at minimum respectively. we first set those values at 0 and replace it with 1 when their moving average is equal to the minimum.


```python
df['mindoy_tavg'] = 0
df['mindoy_tmin'] = 0
df['mindoy_tmax'] = 0
```


```python
for i, l in enumerate(minimums.location.unique()):
    for y in minimums.loc[minimums.location == l,'year']:
        df.loc[df.loc[(df.location == l) & 
               (df.year == y) & 
               (df.tavg_moving == minimums.loc[(minimums.location == l) & 
                                               (minimums.year == y),'tavg_moving'].values[0]),
               'mindoy_tavg'].index[-1],'mindoy_tavg'] = 1
        
        df.loc[df.loc[(df.location == l) & 
               (df.year == y) & 
               (df.tmax_moving == minimums.loc[(minimums.location == l) & 
                                               (minimums.year == y),'tmax_moving'].values[0]),
               'mindoy_tmax'].index[-1],'mindoy_tmax'] = 1
        
        df.loc[df.loc[(df.location == l) & 
               (df.year == y) & 
               (df.tmin_moving == minimums.loc[(minimums.location == l) & 
                                               (minimums.year == y),'tmin_moving'].values[0]),
               'mindoy_tmin'].index[-1],'mindoy_tmin'] = 1
```

We pause here and copy the dataset.


```python
copy = df.copy()
```

Now, we create a long-table which gives minimum of `tavg_moving`,`tmin_moving` and `tmax_moving` as well as their DOY.


```python
i = 0
for var1, var2 in zip(['mindoy_tavg','mindoy_tmin','mindoy_tmax'],
                                   ['tavg_moving','tmin_moving','tmax_moving']):
    subset = df.loc[df[var1].eq(1),:]
    subset = subset[['location','year','Date_doy',var2]]
    melt = subset.melt(id_vars=['location','year','Date_doy'], value_vars=[var2])
    if i == 0:
        output = melt
    else:
        output = pd.concat([output, melt])
    i += 1
```


```python
output
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>location</th>
      <th>year</th>
      <th>Date_doy</th>
      <th>variable</th>
      <th>value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>29890</td>
      <td>2019</td>
      <td>47</td>
      <td>tavg_moving</td>
      <td>-8.5714</td>
    </tr>
    <tr>
      <th>1</th>
      <td>29890</td>
      <td>2020</td>
      <td>19</td>
      <td>tavg_moving</td>
      <td>10.5714</td>
    </tr>
    <tr>
      <th>2</th>
      <td>29890</td>
      <td>2021</td>
      <td>44</td>
      <td>tavg_moving</td>
      <td>-27.9626</td>
    </tr>
    <tr>
      <th>3</th>
      <td>32789</td>
      <td>2019</td>
      <td>33</td>
      <td>tavg_moving</td>
      <td>-24.7483</td>
    </tr>
    <tr>
      <th>4</th>
      <td>32789</td>
      <td>2020</td>
      <td>30</td>
      <td>tavg_moving</td>
      <td>17.6321</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>754</th>
      <td>washingtondc</td>
      <td>2017</td>
      <td>10</td>
      <td>tmax_moving</td>
      <td>58.9286</td>
    </tr>
    <tr>
      <th>755</th>
      <td>washingtondc</td>
      <td>2018</td>
      <td>8</td>
      <td>tmax_moving</td>
      <td>-17.1429</td>
    </tr>
    <tr>
      <th>756</th>
      <td>washingtondc</td>
      <td>2019</td>
      <td>23</td>
      <td>tmax_moving</td>
      <td>41.7857</td>
    </tr>
    <tr>
      <th>757</th>
      <td>washingtondc</td>
      <td>2020</td>
      <td>30</td>
      <td>tmax_moving</td>
      <td>71.7857</td>
    </tr>
    <tr>
      <th>758</th>
      <td>washingtondc</td>
      <td>2021</td>
      <td>52</td>
      <td>tmax_moving</td>
      <td>36.9286</td>
    </tr>
  </tbody>
</table>
<p>2277 rows × 5 columns</p>
</div>



By pivoting the table above, we can obtain a table which gives `Date_doy_tavg`, `Date_doy_tmax`, and `Date_doy_tmin`, which are identical to `mindoy_tavg`,`mindoy_tmax`,and `mindoy_tmin` respectively.


```python
tdoy = pd.pivot_table(output, values='Date_doy', index=['location', 'year'],
                    columns=['variable']).reset_index().rename(columns = {'tavg_moving':'Date_doy_tavg',
                                                                         'tmax_moving':'Date_doy_tmax',
                                                                         'tmin_moving':'Date_doy_tmin'})
```


```python
tdoy
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>variable</th>
      <th>location</th>
      <th>year</th>
      <th>Date_doy_tavg</th>
      <th>Date_doy_tmax</th>
      <th>Date_doy_tmin</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>29890</td>
      <td>2019</td>
      <td>47</td>
      <td>47</td>
      <td>47</td>
    </tr>
    <tr>
      <th>1</th>
      <td>29890</td>
      <td>2020</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
    </tr>
    <tr>
      <th>2</th>
      <td>29890</td>
      <td>2021</td>
      <td>44</td>
      <td>44</td>
      <td>44</td>
    </tr>
    <tr>
      <th>3</th>
      <td>32789</td>
      <td>2019</td>
      <td>33</td>
      <td>22</td>
      <td>33</td>
    </tr>
    <tr>
      <th>4</th>
      <td>32789</td>
      <td>2020</td>
      <td>30</td>
      <td>30</td>
      <td>30</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>754</th>
      <td>washingtondc</td>
      <td>2017</td>
      <td>11</td>
      <td>10</td>
      <td>11</td>
    </tr>
    <tr>
      <th>755</th>
      <td>washingtondc</td>
      <td>2018</td>
      <td>8</td>
      <td>8</td>
      <td>9</td>
    </tr>
    <tr>
      <th>756</th>
      <td>washingtondc</td>
      <td>2019</td>
      <td>34</td>
      <td>23</td>
      <td>33</td>
    </tr>
    <tr>
      <th>757</th>
      <td>washingtondc</td>
      <td>2020</td>
      <td>-6</td>
      <td>30</td>
      <td>30</td>
    </tr>
    <tr>
      <th>758</th>
      <td>washingtondc</td>
      <td>2021</td>
      <td>52</td>
      <td>52</td>
      <td>52</td>
    </tr>
  </tbody>
</table>
<p>759 rows × 5 columns</p>
</div>



The following steps will finally generate a table that each year and location contains a single unique entry.


```python
df = df.merge(tdoy, on = ['location','year'], how = 'left')
```


```python
# tavg_moving, tmin_moving, and tmax_moving are 0 or NA if they are not at its minimum
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>location</th>
      <th>year</th>
      <th>stationid</th>
      <th>lat</th>
      <th>long</th>
      <th>alt</th>
      <th>bloom_date</th>
      <th>bloom_doy</th>
      <th>date</th>
      <th>tmax</th>
      <th>...</th>
      <th>tmin_moving</th>
      <th>tmax_moving</th>
      <th>tavg_below_5</th>
      <th>tavg_above_10</th>
      <th>mindoy_tavg</th>
      <th>mindoy_tmin</th>
      <th>mindoy_tmax</th>
      <th>Date_doy_tavg</th>
      <th>Date_doy_tmax</th>
      <th>Date_doy_tmin</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>29890</td>
      <td>2019</td>
      <td>CA001108910</td>
      <td>48.91993</td>
      <td>-122.640564</td>
      <td>10.0</td>
      <td>5/11/2019</td>
      <td>131</td>
      <td>10/1/2018</td>
      <td>178.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>10</td>
      <td>86.0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>47</td>
      <td>47</td>
      <td>47</td>
    </tr>
    <tr>
      <th>1</th>
      <td>29890</td>
      <td>2019</td>
      <td>CA001108910</td>
      <td>48.91993</td>
      <td>-122.640564</td>
      <td>10.0</td>
      <td>5/11/2019</td>
      <td>131</td>
      <td>10/2/2018</td>
      <td>153.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>10</td>
      <td>86.0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>47</td>
      <td>47</td>
      <td>47</td>
    </tr>
    <tr>
      <th>2</th>
      <td>29890</td>
      <td>2019</td>
      <td>CA001108910</td>
      <td>48.91993</td>
      <td>-122.640564</td>
      <td>10.0</td>
      <td>5/11/2019</td>
      <td>131</td>
      <td>10/3/2018</td>
      <td>122.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>10</td>
      <td>86.0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>47</td>
      <td>47</td>
      <td>47</td>
    </tr>
    <tr>
      <th>3</th>
      <td>29890</td>
      <td>2019</td>
      <td>CA001108910</td>
      <td>48.91993</td>
      <td>-122.640564</td>
      <td>10.0</td>
      <td>5/11/2019</td>
      <td>131</td>
      <td>10/4/2018</td>
      <td>135.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>10</td>
      <td>86.0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>47</td>
      <td>47</td>
      <td>47</td>
    </tr>
    <tr>
      <th>4</th>
      <td>29890</td>
      <td>2019</td>
      <td>CA001108910</td>
      <td>48.91993</td>
      <td>-122.640564</td>
      <td>10.0</td>
      <td>5/11/2019</td>
      <td>131</td>
      <td>10/5/2018</td>
      <td>151.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>10</td>
      <td>86.0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>47</td>
      <td>47</td>
      <td>47</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 26 columns</p>
</div>




```python
'''
We make sure that tavg_moving is not 0 if its mindoy_tavg is equal to 1, 
because we will drop the cases where the product of the two variables is 1
'''
df.loc[(df.tavg_moving.eq(0) & df.mindoy_tavg.eq(1)), 'tavg_moving'] = .001
```


```python
# tavg_moving, tmin_moving, and tmax_moving will be 0 if they are not at its minimum
df['tavg_moving'] = df.tavg_moving*df.mindoy_tavg
df['tmin_moving'] = df.tmin_moving*df.mindoy_tmin
df['tmax_moving'] = df.tmax_moving*df.mindoy_tmax
```


```python
df = df.loc[(df.tavg_moving.ne(0) | df.tmin_moving.ne(0) | df.tmax_moving.ne(0))&
       df.tavg_moving.notnull() & df.tmin_moving.notnull() & df.tmax_moving.notnull(),:]
```


```python
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>location</th>
      <th>year</th>
      <th>stationid</th>
      <th>lat</th>
      <th>long</th>
      <th>alt</th>
      <th>bloom_date</th>
      <th>bloom_doy</th>
      <th>date</th>
      <th>tmax</th>
      <th>...</th>
      <th>tmin_moving</th>
      <th>tmax_moving</th>
      <th>tavg_below_5</th>
      <th>tavg_above_10</th>
      <th>mindoy_tavg</th>
      <th>mindoy_tmin</th>
      <th>mindoy_tmax</th>
      <th>Date_doy_tavg</th>
      <th>Date_doy_tmax</th>
      <th>Date_doy_tmin</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>138</th>
      <td>29890</td>
      <td>2019</td>
      <td>CA001108910</td>
      <td>48.91993</td>
      <td>-122.640564</td>
      <td>10.0</td>
      <td>5/11/2019</td>
      <td>131</td>
      <td>2/16/2019</td>
      <td>72.0</td>
      <td>...</td>
      <td>-41.9286</td>
      <td>24.8571</td>
      <td>10</td>
      <td>86.0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>47</td>
      <td>47</td>
      <td>47</td>
    </tr>
    <tr>
      <th>322</th>
      <td>29890</td>
      <td>2020</td>
      <td>CA001108910</td>
      <td>48.91993</td>
      <td>-122.640564</td>
      <td>10.0</td>
      <td>5/9/2020</td>
      <td>130</td>
      <td>1/19/2020</td>
      <td>73.0</td>
      <td>...</td>
      <td>-19.9286</td>
      <td>41.0000</td>
      <td>6</td>
      <td>93.0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
    </tr>
    <tr>
      <th>498</th>
      <td>29890</td>
      <td>2021</td>
      <td>CA001108910</td>
      <td>48.91993</td>
      <td>-122.640564</td>
      <td>10.0</td>
      <td>4/30/2021</td>
      <td>120</td>
      <td>2/13/2021</td>
      <td>5.0</td>
      <td>...</td>
      <td>-50.7796</td>
      <td>-5.5571</td>
      <td>16</td>
      <td>24.0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>44</td>
      <td>44</td>
      <td>44</td>
    </tr>
    <tr>
      <th>900</th>
      <td>32789</td>
      <td>2019</td>
      <td>USW00094728</td>
      <td>40.73082</td>
      <td>-73.997330</td>
      <td>6.0</td>
      <td>4/16/2019</td>
      <td>106</td>
      <td>1/22/2019</td>
      <td>-5.0</td>
      <td>...</td>
      <td>-0.0000</td>
      <td>12.5714</td>
      <td>72</td>
      <td>124.0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>33</td>
      <td>22</td>
      <td>33</td>
    </tr>
    <tr>
      <th>911</th>
      <td>32789</td>
      <td>2019</td>
      <td>USW00094728</td>
      <td>40.73082</td>
      <td>-73.997330</td>
      <td>6.0</td>
      <td>4/16/2019</td>
      <td>106</td>
      <td>2/2/2019</td>
      <td>11.0</td>
      <td>...</td>
      <td>-71.4286</td>
      <td>0.0000</td>
      <td>72</td>
      <td>124.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>33</td>
      <td>22</td>
      <td>33</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 26 columns</p>
</div>




```python
# We don't need the variables in the drop method
df = df.drop(['stationid','date','tmax','tmin','prcp','tavg','month','Date_doy'], axis = 1)
```


```python
# group by the variables and re-define the following variables: tavg_moving, tmin_moving, and tmax_moving
df = df.groupby(['location','lat','long','alt','year',
            'bloom_date','bloom_doy','Date_doy_tavg',
            'Date_doy_tmax','Date_doy_tmin','tavg_below_5', 'tavg_above_10']).aggregate({'tavg_moving':min,
                                                        'tmin_moving':min,
                                                        'tmax_moving':min}).reset_index()
```

---

Now we would like to define average of `tmin`, `tmax`, and `tavg` of each month and count of the days when `prcp` is above 0 in a given month. Since the table `df` no longer has daily temperature and precipitation data, we use the `copy` table. 


```python
copy2 = copy.loc[copy.month.le(2),['location','year','month','tavg','tmin','tmax','prcp']]
```


```python
copy2 = copy2.groupby(['location','year','month']).aggregate({'prcp': lambda x: (x>0).sum(),
                                                     'tmin': np.mean,
                                                     'tmax': np.mean,
                                                     'tavg': np.mean}).reset_index()
```


```python
copy2.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>location</th>
      <th>year</th>
      <th>month</th>
      <th>prcp</th>
      <th>tmin</th>
      <th>tmax</th>
      <th>tavg</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>29890</td>
      <td>2019</td>
      <td>-2</td>
      <td>17.0</td>
      <td>63.677419</td>
      <td>148.322581</td>
      <td>105.967742</td>
    </tr>
    <tr>
      <th>1</th>
      <td>29890</td>
      <td>2019</td>
      <td>-1</td>
      <td>19.0</td>
      <td>52.233333</td>
      <td>117.866667</td>
      <td>84.966667</td>
    </tr>
    <tr>
      <th>2</th>
      <td>29890</td>
      <td>2019</td>
      <td>0</td>
      <td>20.0</td>
      <td>31.516129</td>
      <td>89.129032</td>
      <td>60.322581</td>
    </tr>
    <tr>
      <th>3</th>
      <td>29890</td>
      <td>2019</td>
      <td>1</td>
      <td>15.0</td>
      <td>28.290323</td>
      <td>95.032258</td>
      <td>61.548387</td>
    </tr>
    <tr>
      <th>4</th>
      <td>29890</td>
      <td>2019</td>
      <td>2</td>
      <td>4.0</td>
      <td>-17.964286</td>
      <td>48.250000</td>
      <td>15.214286</td>
    </tr>
  </tbody>
</table>
</div>



We pivot the table to have one unique entry for a given year and location

The variable names followed by dot and the month value


```python
copy2 = pd.pivot_table(copy2, values=['tmin','tmax','tavg','prcp'], index=['location', 'year'],
                    columns=['month']).reset_index()
colnames = [x+'.'+y if y != '' else x for x,y in zip(copy2.columns.droplevel(1).astype(str), copy2.columns.droplevel(0).astype(str))]
copy2.columns = colnames
```


```python
copy2.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>location</th>
      <th>year</th>
      <th>prcp.-2</th>
      <th>prcp.-1</th>
      <th>prcp.0</th>
      <th>prcp.1</th>
      <th>prcp.2</th>
      <th>tavg.-2</th>
      <th>tavg.-1</th>
      <th>tavg.0</th>
      <th>...</th>
      <th>tmax.-2</th>
      <th>tmax.-1</th>
      <th>tmax.0</th>
      <th>tmax.1</th>
      <th>tmax.2</th>
      <th>tmin.-2</th>
      <th>tmin.-1</th>
      <th>tmin.0</th>
      <th>tmin.1</th>
      <th>tmin.2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>29890</td>
      <td>2019</td>
      <td>17.0</td>
      <td>19.0</td>
      <td>20.0</td>
      <td>15.0</td>
      <td>4.0</td>
      <td>105.967742</td>
      <td>84.966667</td>
      <td>60.322581</td>
      <td>...</td>
      <td>148.322581</td>
      <td>117.866667</td>
      <td>89.129032</td>
      <td>95.032258</td>
      <td>48.250000</td>
      <td>63.677419</td>
      <td>52.233333</td>
      <td>31.516129</td>
      <td>28.290323</td>
      <td>-17.964286</td>
    </tr>
    <tr>
      <th>1</th>
      <td>29890</td>
      <td>2020</td>
      <td>14.0</td>
      <td>14.0</td>
      <td>23.0</td>
      <td>22.0</td>
      <td>19.0</td>
      <td>92.741935</td>
      <td>72.933333</td>
      <td>58.419355</td>
      <td>...</td>
      <td>130.000000</td>
      <td>108.600000</td>
      <td>79.129032</td>
      <td>75.612903</td>
      <td>87.379310</td>
      <td>55.741935</td>
      <td>37.200000</td>
      <td>37.548387</td>
      <td>20.161290</td>
      <td>24.793103</td>
    </tr>
    <tr>
      <th>2</th>
      <td>29890</td>
      <td>2021</td>
      <td>14.0</td>
      <td>9.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>10.0</td>
      <td>108.483871</td>
      <td>64.348175</td>
      <td>NaN</td>
      <td>...</td>
      <td>137.387097</td>
      <td>90.794048</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>40.176190</td>
      <td>79.516129</td>
      <td>38.142460</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>-11.319558</td>
    </tr>
    <tr>
      <th>3</th>
      <td>32789</td>
      <td>2019</td>
      <td>22.0</td>
      <td>30.0</td>
      <td>26.0</td>
      <td>20.0</td>
      <td>24.0</td>
      <td>140.055252</td>
      <td>67.824363</td>
      <td>44.636134</td>
      <td>...</td>
      <td>174.935484</td>
      <td>98.533333</td>
      <td>74.709677</td>
      <td>39.129032</td>
      <td>59.214286</td>
      <td>110.419355</td>
      <td>39.233333</td>
      <td>15.806452</td>
      <td>-32.677419</td>
      <td>-11.678571</td>
    </tr>
    <tr>
      <th>4</th>
      <td>32789</td>
      <td>2020</td>
      <td>15.0</td>
      <td>9.0</td>
      <td>14.0</td>
      <td>9.0</td>
      <td>9.0</td>
      <td>151.951782</td>
      <td>64.310026</td>
      <td>34.899526</td>
      <td>...</td>
      <td>190.225806</td>
      <td>103.400000</td>
      <td>63.838710</td>
      <td>73.548387</td>
      <td>80.551724</td>
      <td>119.935484</td>
      <td>28.766667</td>
      <td>6.677419</td>
      <td>6.290323</td>
      <td>10.172414</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 22 columns</p>
</div>



Now, we merge the table with the table `df`. We name the merged table `output`


```python
output= df.merge(copy2.dropna(), how = 'right', on = ['location','year'])
```


```python
output.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>location</th>
      <th>lat</th>
      <th>long</th>
      <th>alt</th>
      <th>year</th>
      <th>bloom_date</th>
      <th>bloom_doy</th>
      <th>Date_doy_tavg</th>
      <th>Date_doy_tmax</th>
      <th>Date_doy_tmin</th>
      <th>...</th>
      <th>tmax.-2</th>
      <th>tmax.-1</th>
      <th>tmax.0</th>
      <th>tmax.1</th>
      <th>tmax.2</th>
      <th>tmin.-2</th>
      <th>tmin.-1</th>
      <th>tmin.0</th>
      <th>tmin.1</th>
      <th>tmin.2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>29890</td>
      <td>48.91993</td>
      <td>-122.640564</td>
      <td>10.0</td>
      <td>2019</td>
      <td>5/11/2019</td>
      <td>131</td>
      <td>47</td>
      <td>47</td>
      <td>47</td>
      <td>...</td>
      <td>148.322581</td>
      <td>117.866667</td>
      <td>89.129032</td>
      <td>95.032258</td>
      <td>48.250000</td>
      <td>63.677419</td>
      <td>52.233333</td>
      <td>31.516129</td>
      <td>28.290323</td>
      <td>-17.964286</td>
    </tr>
    <tr>
      <th>1</th>
      <td>29890</td>
      <td>48.91993</td>
      <td>-122.640564</td>
      <td>10.0</td>
      <td>2020</td>
      <td>5/9/2020</td>
      <td>130</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>...</td>
      <td>130.000000</td>
      <td>108.600000</td>
      <td>79.129032</td>
      <td>75.612903</td>
      <td>87.379310</td>
      <td>55.741935</td>
      <td>37.200000</td>
      <td>37.548387</td>
      <td>20.161290</td>
      <td>24.793103</td>
    </tr>
    <tr>
      <th>2</th>
      <td>32789</td>
      <td>40.73082</td>
      <td>-73.997330</td>
      <td>6.0</td>
      <td>2019</td>
      <td>4/16/2019</td>
      <td>106</td>
      <td>33</td>
      <td>22</td>
      <td>33</td>
      <td>...</td>
      <td>174.935484</td>
      <td>98.533333</td>
      <td>74.709677</td>
      <td>39.129032</td>
      <td>59.214286</td>
      <td>110.419355</td>
      <td>39.233333</td>
      <td>15.806452</td>
      <td>-32.677419</td>
      <td>-11.678571</td>
    </tr>
    <tr>
      <th>3</th>
      <td>32789</td>
      <td>40.73082</td>
      <td>-73.997330</td>
      <td>6.0</td>
      <td>2020</td>
      <td>4/27/2020</td>
      <td>118</td>
      <td>30</td>
      <td>30</td>
      <td>30</td>
      <td>...</td>
      <td>190.225806</td>
      <td>103.400000</td>
      <td>63.838710</td>
      <td>73.548387</td>
      <td>80.551724</td>
      <td>119.935484</td>
      <td>28.766667</td>
      <td>6.677419</td>
      <td>6.290323</td>
      <td>10.172414</td>
    </tr>
    <tr>
      <th>4</th>
      <td>32789</td>
      <td>40.73082</td>
      <td>-73.997330</td>
      <td>6.0</td>
      <td>2021</td>
      <td>4/14/2021</td>
      <td>104</td>
      <td>34</td>
      <td>52</td>
      <td>33</td>
      <td>...</td>
      <td>177.258065</td>
      <td>154.600000</td>
      <td>72.709677</td>
      <td>42.709677</td>
      <td>38.678571</td>
      <td>110.322581</td>
      <td>78.200000</td>
      <td>7.838710</td>
      <td>-11.483871</td>
      <td>-13.857143</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 35 columns</p>
</div>



Now we define additional geographical variables.
`long.y` is the sine of longitude and `long.x` is cosine of longitude


```python
output['long.y'] =  output.long.apply(lambda x: np.sin(np.pi/180*x))
output['long.x'] =  output.long.apply(lambda x: np.cos(np.pi/180*x))
```


```python
copy3 = output[['location','year']].merge(copy)
```


```python
output.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>location</th>
      <th>lat</th>
      <th>long</th>
      <th>alt</th>
      <th>year</th>
      <th>bloom_date</th>
      <th>bloom_doy</th>
      <th>Date_doy_tavg</th>
      <th>Date_doy_tmax</th>
      <th>Date_doy_tmin</th>
      <th>...</th>
      <th>tmax.0</th>
      <th>tmax.1</th>
      <th>tmax.2</th>
      <th>tmin.-2</th>
      <th>tmin.-1</th>
      <th>tmin.0</th>
      <th>tmin.1</th>
      <th>tmin.2</th>
      <th>long.y</th>
      <th>long.x</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>29890</td>
      <td>48.91993</td>
      <td>-122.640564</td>
      <td>10.0</td>
      <td>2019</td>
      <td>5/11/2019</td>
      <td>131</td>
      <td>47</td>
      <td>47</td>
      <td>47</td>
      <td>...</td>
      <td>89.129032</td>
      <td>95.032258</td>
      <td>48.250000</td>
      <td>63.677419</td>
      <td>52.233333</td>
      <td>31.516129</td>
      <td>28.290323</td>
      <td>-17.964286</td>
      <td>-0.842071</td>
      <td>-0.539367</td>
    </tr>
    <tr>
      <th>1</th>
      <td>29890</td>
      <td>48.91993</td>
      <td>-122.640564</td>
      <td>10.0</td>
      <td>2020</td>
      <td>5/9/2020</td>
      <td>130</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>...</td>
      <td>79.129032</td>
      <td>75.612903</td>
      <td>87.379310</td>
      <td>55.741935</td>
      <td>37.200000</td>
      <td>37.548387</td>
      <td>20.161290</td>
      <td>24.793103</td>
      <td>-0.842071</td>
      <td>-0.539367</td>
    </tr>
    <tr>
      <th>2</th>
      <td>32789</td>
      <td>40.73082</td>
      <td>-73.997330</td>
      <td>6.0</td>
      <td>2019</td>
      <td>4/16/2019</td>
      <td>106</td>
      <td>33</td>
      <td>22</td>
      <td>33</td>
      <td>...</td>
      <td>74.709677</td>
      <td>39.129032</td>
      <td>59.214286</td>
      <td>110.419355</td>
      <td>39.233333</td>
      <td>15.806452</td>
      <td>-32.677419</td>
      <td>-11.678571</td>
      <td>-0.961249</td>
      <td>0.275682</td>
    </tr>
    <tr>
      <th>3</th>
      <td>32789</td>
      <td>40.73082</td>
      <td>-73.997330</td>
      <td>6.0</td>
      <td>2020</td>
      <td>4/27/2020</td>
      <td>118</td>
      <td>30</td>
      <td>30</td>
      <td>30</td>
      <td>...</td>
      <td>63.838710</td>
      <td>73.548387</td>
      <td>80.551724</td>
      <td>119.935484</td>
      <td>28.766667</td>
      <td>6.677419</td>
      <td>6.290323</td>
      <td>10.172414</td>
      <td>-0.961249</td>
      <td>0.275682</td>
    </tr>
    <tr>
      <th>4</th>
      <td>32789</td>
      <td>40.73082</td>
      <td>-73.997330</td>
      <td>6.0</td>
      <td>2021</td>
      <td>4/14/2021</td>
      <td>104</td>
      <td>34</td>
      <td>52</td>
      <td>33</td>
      <td>...</td>
      <td>72.709677</td>
      <td>42.709677</td>
      <td>38.678571</td>
      <td>110.322581</td>
      <td>78.200000</td>
      <td>7.838710</td>
      <td>-11.483871</td>
      <td>-13.857143</td>
      <td>-0.961249</td>
      <td>0.275682</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 37 columns</p>
</div>



Lastly, we define `slope`,`dg2_coef`,`intc`,`tavg.3`,`tmin.3`,and `tmax.3` variables. `slope` and `intc` is found from the linear fit between `Date_doy` and `tavg` between the `Date_doy_tavg` and March 10th. Similarly, `dg2_coef` is found from the quadratic fit between `Date_doy_tavg` and March 10th. If the `Date_doy_tavg` is after 2/28, we just fit linear and quadratic model with days between 1/31 and 3/10. For `tavg.3`,`tmin.3`, and `tmax.3`, we find the average of `tavg`, `tmin` and `tmax` betwen March 1 and March 10.


```python
slope = []
acc = []
intercept = []
march = []
march1 = []
march2 = []
for l,y in zip(output.location, output.year):
    threshold = output.loc[output.location.eq(l) & output.year.eq(y),'Date_doy_tavg'].values[0]
    if threshold > 60:
        ss = copy3.loc[copy3.location.eq(l) & copy3.year.eq(y) & copy3.Date_doy.ge(30) & 
                       copy3.Date_doy.le(70),['Date_doy','tavg','tmin','tmax']]
        acc.append(np.polyfit(x = ss.Date_doy, y = ss.tavg, deg = 2)[0])
        slope.append(np.polyfit(x = ss.Date_doy, y = ss.tavg, deg = 1)[0])
        intercept.append(np.polyfit(x = ss.Date_doy, y = ss.tavg, deg = 1)[1])
        ss = copy3.loc[copy3.Date_doy.gt(59) & 
               copy3.Date_doy.le(70),['Date_doy','tavg','tmin','tmax']]
        march.append(ss.loc[ss.Date_doy.gt(59),'tavg'].mean())
        march1.append(ss.loc[ss.Date_doy.gt(59),'tmax'].mean())
        march2.append(ss.loc[ss.Date_doy.gt(59),'tmin'].mean())
    else:
        ss = copy3.loc[copy3.location.eq(l) & copy3.year.eq(y) & copy3.Date_doy.ge(threshold) & 
                       copy3.Date_doy.le(70),['Date_doy','tavg','tmin','tmax']]
        acc.append(np.polyfit(x = ss.Date_doy, y = ss.tavg, deg = 2)[0])
        slope.append(np.polyfit(x = ss.Date_doy, y = ss.tavg, deg = 1)[0])
        intercept.append(np.polyfit(x = ss.Date_doy, y = ss.tavg, deg = 1)[1])
        march.append(ss.loc[ss.Date_doy.gt(59),'tavg'].mean())
        march1.append(ss.loc[ss.Date_doy.gt(59),'tmax'].mean())
        march2.append(ss.loc[ss.Date_doy.gt(59),'tmin'].mean())
```


```python
output['slope'] = slope
output['dg2_coef'] = acc
output['intc'] = intercept
output['tavg.3'] = march
output['tmin.3'] = march2
output['tmax.3'] = march1
```


```python
output.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>location</th>
      <th>lat</th>
      <th>long</th>
      <th>alt</th>
      <th>year</th>
      <th>bloom_date</th>
      <th>bloom_doy</th>
      <th>Date_doy_tavg</th>
      <th>Date_doy_tmax</th>
      <th>Date_doy_tmin</th>
      <th>...</th>
      <th>tmin.1</th>
      <th>tmin.2</th>
      <th>long.y</th>
      <th>long.x</th>
      <th>slope</th>
      <th>dg2_coef</th>
      <th>intc</th>
      <th>tavg.3</th>
      <th>tmin.3</th>
      <th>tmax.3</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>29890</td>
      <td>48.91993</td>
      <td>-122.640564</td>
      <td>10.0</td>
      <td>2019</td>
      <td>5/11/2019</td>
      <td>131</td>
      <td>47</td>
      <td>47</td>
      <td>47</td>
      <td>...</td>
      <td>28.290323</td>
      <td>-17.964286</td>
      <td>-0.842071</td>
      <td>-0.539367</td>
      <td>0.129565</td>
      <td>-0.028945</td>
      <td>27.253768</td>
      <td>35.181818</td>
      <td>-10.000000</td>
      <td>80.272727</td>
    </tr>
    <tr>
      <th>1</th>
      <td>29890</td>
      <td>48.91993</td>
      <td>-122.640564</td>
      <td>10.0</td>
      <td>2020</td>
      <td>5/9/2020</td>
      <td>130</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>...</td>
      <td>20.161290</td>
      <td>24.793103</td>
      <td>-0.842071</td>
      <td>-0.539367</td>
      <td>-0.250320</td>
      <td>0.018789</td>
      <td>73.023862</td>
      <td>58.727273</td>
      <td>27.272727</td>
      <td>90.272727</td>
    </tr>
    <tr>
      <th>2</th>
      <td>32789</td>
      <td>40.73082</td>
      <td>-73.997330</td>
      <td>6.0</td>
      <td>2019</td>
      <td>4/16/2019</td>
      <td>106</td>
      <td>33</td>
      <td>22</td>
      <td>33</td>
      <td>...</td>
      <td>-32.677419</td>
      <td>-11.678571</td>
      <td>-0.961249</td>
      <td>0.275682</td>
      <td>-1.047027</td>
      <td>0.064543</td>
      <td>75.590678</td>
      <td>8.663593</td>
      <td>-22.000000</td>
      <td>39.545455</td>
    </tr>
    <tr>
      <th>3</th>
      <td>32789</td>
      <td>40.73082</td>
      <td>-73.997330</td>
      <td>6.0</td>
      <td>2020</td>
      <td>4/27/2020</td>
      <td>118</td>
      <td>30</td>
      <td>30</td>
      <td>30</td>
      <td>...</td>
      <td>6.290323</td>
      <td>10.172414</td>
      <td>-0.961249</td>
      <td>0.275682</td>
      <td>1.164496</td>
      <td>0.122699</td>
      <td>-4.880522</td>
      <td>78.481070</td>
      <td>40.454545</td>
      <td>121.181818</td>
    </tr>
    <tr>
      <th>4</th>
      <td>32789</td>
      <td>40.73082</td>
      <td>-73.997330</td>
      <td>6.0</td>
      <td>2021</td>
      <td>4/14/2021</td>
      <td>104</td>
      <td>34</td>
      <td>52</td>
      <td>33</td>
      <td>...</td>
      <td>-11.483871</td>
      <td>-13.857143</td>
      <td>-0.961249</td>
      <td>0.275682</td>
      <td>1.809549</td>
      <td>0.112638</td>
      <td>-72.127563</td>
      <td>39.654925</td>
      <td>-2.909091</td>
      <td>86.363636</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 43 columns</p>
</div>


