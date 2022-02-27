# MARS

```python
import numpy as np
import pandas as pd
import os
os.chdir('./data')
import scipy
import matplotlib.pyplot as plt
from sklearn.model_selection import cross_val_score

df = pd.read_csv('./lee.csv')
# defining quadratic and interaction terms
df['year.sq'] = df.year**2




# between metoerology variables

df['tavg.1tavg_below_5'] = df['tavg.1']*df.tavg_below_5 
df['prcp.0tavg_below_5'] = df['prcp.0']*df.tavg_below_5
df['tavg.0tmax.2'] = df['tavg.0']*df['tmax.2'] 
df['prcp.-2prcp.2'] = df['prcp.-2']*df['prcp.2']
df['prcp.-2prcp.0'] = df['prcp.-2']*df['prcp.0']
df['tavg.0prcp.2'] = df['tavg.0']*df['prcp.2']
df['Date_doy_tmaxDate_doy_tmin'] = df['Date_doy_tmax']*df['Date_doy_tmin']
df['tmin_movingtmin.1'] = df['tmin_moving']*df['tmin.1']
df['tavg_below_5prcp.2'] = df['tavg_below_5']*df['prcp.2']
df['tavg.0prcp.2'] = df['tavg.0']*df['prcp.2']
df['prcp.-1tmax.3'] = df['prcp.-1']*df['tmax.3']



# between met and geography

df['tavg_above_10lat'] = df['lat']*df['tavg_above_10']
df['prcp.1alt'] = df['prcp.1']*df['alt']
df['tmin.-1long.y'] = df['tmin.-1']*df['long.y']
df['alttavg_moving'] = df['alt']*df['tavg_moving']


# between geography

df['latlong.y'] = df['lat']*df['long.y']
df['long.ylong.x'] = df['long.y']*df['long.x']






# # selecting covariates to be used in the model



X_full = df[['lat', 'alt', 'long.y', 'long.x', 'year', 'year.sq',
       'Date_doy_tavg', 'Date_doy_tmax', 'Date_doy_tmin', 
        'tavg_below_5','tavg_above_10', 
       'tavg_moving', 'tmin_moving', 'tmax_moving', 
        'prcp.-2', 'prcp.-1', 'prcp.0', 'prcp.1', 'prcp.2', 
       'tavg.-2', 'tavg.-1', 'tavg.0', 'tavg.1', 'tavg.2', 
        'tmax.-2', 'tmax.-1', 'tmax.0', 'tmax.1', 'tmax.2',
       'tmin.-2', 'tmin.-1', 'tmin.0', 'tmin.1', 'tmin.2', 
       'slope', 'dg2_coef', 'intc', 
        'tavg.3', 'tmin.3', 'tmax.3', 
       'tavg.1tavg_below_5', 'tavg.0tmax.2', 'prcp.-2prcp.2', 'prcp.-2prcp.0',
       'tavg.0prcp.2', 'Date_doy_tmaxDate_doy_tmin', 'tmin_movingtmin.1',
       'tavg_below_5prcp.2', 'prcp.-1tmax.3', 'tavg_above_10lat', 'prcp.1alt',
       'tmin.-1long.y', 'alttavg_moving', 'latlong.y', 'long.ylong.x']]

y = df.bloom_doy

```

We first select the variables to use in the model. The details of the selection process are explained in the paper.

We considered the MARS because we have seen that location is a very strong predictor. In other words, we would observe different patterns or fit for each of the locations, which are described by latitude, longitude and altitude information in our dataset. In addition, we hope to find differences in year or meteorology data would also split the patterns in association between the `bloom_doy` and other covariates.


```python
from pyearth import Earth
```

We performed the grid search to find the optimal max_degree parameter.


```python
from sklearn.metrics import make_scorer
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import ShuffleSplit
mse = make_scorer(mean_squared_error, greater_is_better = False )
# We will create 20 different cross-validations with the train-test-split-ratio at 7 to 3.
cv = ShuffleSplit(n_splits=20, test_size=0.3)
from sklearn.model_selection import GridSearchCV
parameters = {"max_degree" : [1,2]}
```


```python
# suppress 'FutureWarning' messages
import warnings
warnings.filterwarnings('ignore')
```


```python
regressor = Earth()
clf = GridSearchCV(regressor, parameters, cv = cv, scoring = mse)
clf.fit(X_full, y)
```




    GridSearchCV(cv=ShuffleSplit(n_splits=20, random_state=None, test_size=0.3, train_size=None),
                 estimator=Earth(), param_grid={'max_degree': [1, 2]},
                 scoring=make_scorer(mean_squared_error, greater_is_better=False))




```python
clf.cv_results_['rank_test_score']
```




    array([1, 2], dtype=int32)



We found that the max_degree parameter is better to be 1.


```python
from sklearn.model_selection import cross_val_score
regressor = Earth(smooth = True, penalty = 3)
MARS_score = cross_val_score(regressor, X_full, y, cv= cv, scoring = mse)
```


```python
# mean of 20 MSE and its variance
np.sqrt(abs(MARS_score)).mean(), np.sqrt(abs(MARS_score)).var()
```




    (6.276490591327596, 0.12299216460444888)




```python
# Fir the final 
mars_final = regressor.fit(X_full, y)
```


```python
print(mars_final.summary())
```

    Earth Model
    ----------------------------------------------------------------------------
    Basis Function                                        Pruned  Coefficient   
    ----------------------------------------------------------------------------
    (Intercept)                                           No      115.416       
    C(tmax.2|s=+1,-4.57143,56.8571,115.725)               No      -0.0535534    
    C(tmax.2|s=-1,-4.57143,56.8571,115.725)               No      0.130376      
    C(tmax.3|s=+1,42.2472,126.364,165.591)                No      -0.128535     
    C(tmax.3|s=-1,42.2472,126.364,165.591)                Yes     None          
    C(long.ylong.x|s=+1,-0.176084,0.147692,0.300939)      Yes     None          
    C(long.ylong.x|s=-1,-0.176084,0.147692,0.300939)      No      -63.5343      
    C(tmax.-2|s=+1,148.819,220.992,250.448)               No      0.1196        
    C(tmax.-2|s=-1,148.819,220.992,250.448)               Yes     None          
    tavg_below_5                                          No      0.468017      
    C(tavg.0|s=+1,-12.3861,46.1682,85.6647)               No      0.158863      
    C(tavg.0|s=-1,-12.3861,46.1682,85.6647)               No      -0.180521     
    long.x                                                No      -21.8943      
    C(tmax.1|s=+1,20.4135,49.3548,103.061)                No      0.241433      
    C(tmax.1|s=-1,20.4135,49.3548,103.061)                Yes     None          
    C(tmax.-1|s=+1,48.6167,82.4667,154.159)               No      0.0781458     
    C(tmax.-1|s=-1,48.6167,82.4667,154.159)               No      -0.170574     
    C(tmin_movingtmin.1|s=+1,142.561,1862.82,17424.7)     No      -0.00020908   
    C(tmin_movingtmin.1|s=-1,142.561,1862.82,17424.7)     No      -0.0016647    
    C(tavg.1tavg_below_5|s=+1,-3238.32,-2976.65,1180.71)  No      -0.00275584   
    C(tavg.1tavg_below_5|s=-1,-3238.32,-2976.65,1180.71)  No      0.00431184    
    C(prcp.-2prcp.2|s=+1,216,432,546)                     No      -0.12285      
    C(prcp.-2prcp.2|s=-1,216,432,546)                     No      0.00390473    
    C(tavg.1|s=+1,-33.8871,38.6774,75.0645)               No      0.0853577     
    C(tavg.1|s=-1,-33.8871,38.6774,75.0645)               No      -0.502295     
    C(tavg.1tavg_below_5|s=+1,-8993.45,-3500,-3238.32)    Yes     None          
    C(tavg.1tavg_below_5|s=-1,-8993.45,-3500,-3238.32)    No      -0.000837816  
    C(tavg.0prcp.2|s=+1,-1046.68,-248.903,911.661)        Yes     None          
    C(tavg.0prcp.2|s=-1,-1046.68,-248.903,911.661)        No      0.0156378     
    C(Date_doy_tmaxDate_doy_tmin|s=+1,-527,476,2863.5)    Yes     None          
    C(Date_doy_tmaxDate_doy_tmin|s=-1,-527,476,2863.5)    No      0.00345862    
    C(tmax.1|s=+1,-38.3607,-8.52792,20.4135)              No      -0.176283     
    C(tmax.1|s=-1,-38.3607,-8.52792,20.4135)              Yes     None          
    ----------------------------------------------------------------------------
    MSE: 31.2031, GCV: 37.2471, RSQ: 0.9014, GRSQ: 0.8826



```python
# we also use the boosting method
from sklearn.ensemble import AdaBoostRegressor

boosted_model = AdaBoostRegressor(base_estimator=regressor, n_estimators=25,
                                  learning_rate=0.1, loss="exponential")
boost_score = cross_val_score(boosted_model, X_full, y, cv= cv, scoring = mse)
```


```python
# mean of 20 MSE and its variance
np.sqrt(abs(boost_score)).mean(), np.sqrt(abs(boost_score)).var()
```




    (5.994070880449536, 0.18176450444313832)


