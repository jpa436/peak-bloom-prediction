```python
import numpy as np
import pandas as pd
import os
os.chdir('./data')
import scipy
import matplotlib.pyplot as plt



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


```python
from sklearn.metrics import make_scorer
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import ShuffleSplit
from sklearn.model_selection import GridSearchCV
from sklearn.model_selection import cross_val_score

mse = make_scorer(mean_squared_error, greater_is_better = False )
cv = ShuffleSplit(n_splits=20, test_size=0.3)

```


```python
alphas = np.arange(0.001, 10, .25 )

```


```python
import warnings
warnings.filterwarnings('ignore')
```


```python
from sklearn.linear_model import Lasso
from sklearn.linear_model import Ridge
alphas = np.arange(0.001, 10, .25 )
parameters = {'alpha':np.arange(0.001, 10, .25 )}
lasso = Lasso()
lasso_grid = GridSearchCV(lasso, parameters, cv = cv, scoring = mse)
ridge = Ridge()
ridge_grid = GridSearchCV(ridge, parameters, cv = cv, scoring = mse)
lasso_grid.fit(X_full, y)
ridge_grid.fit(X_full, y)
```




    GridSearchCV(cv=ShuffleSplit(n_splits=20, random_state=None, test_size=0.3, train_size=None),
                 estimator=Ridge(),
                 param_grid={'alpha': array([1.000e-03, 2.510e-01, 5.010e-01, 7.510e-01, 1.001e+00, 1.251e+00,
           1.501e+00, 1.751e+00, 2.001e+00, 2.251e+00, 2.501e+00, 2.751e+00,
           3.001e+00, 3.251e+00, 3.501e+00, 3.751e+00, 4.001e+00, 4.251e+00,
           4.501e+00, 4.751e+00, 5.001e+00, 5.251e+00, 5.501e+00, 5.751e+00,
           6.001e+00, 6.251e+00, 6.501e+00, 6.751e+00, 7.001e+00, 7.251e+00,
           7.501e+00, 7.751e+00, 8.001e+00, 8.251e+00, 8.501e+00, 8.751e+00,
           9.001e+00, 9.251e+00, 9.501e+00, 9.751e+00])},
                 scoring=make_scorer(mean_squared_error, greater_is_better=False))




```python
index = np.where(lasso_grid.cv_results_['rank_test_score']==1)[0][0]
alpha_lasso = alphas[index]
alpha_lasso
```




    0.001




```python
index = np.where(ridge_grid.cv_results_['rank_test_score']==1)[0][0]
alpha_ridge = alphas[index]
alpha_ridge
```




    0.001




```python
lasso = Lasso(alpha = alpha_lasso)
ridge = Ridge(alpha = alpha_ridge)
scoreLasso = cross_val_score(lasso, X_full, y, cv= cv, scoring = mse)
scoreRidge = cross_val_score(ridge, X_full, y, cv = cv, scoring = mse)
```


```python
# mean squred error
np.sqrt(abs(scoreLasso)).mean(), np.sqrt(abs(scoreRidge)).mean()
```




    (5.648485932342682, 5.88353408014353)




```python
# variance
np.sqrt(abs(scoreLasso)).var(), np.sqrt(abs(scoreRidge)).var()
```




    (0.18903172536195578, 0.2672479140140163)



## PCA method


```python
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import make_pipeline, Pipeline
from sklearn.linear_model import LinearRegression
scaler = StandardScaler()
pca = PCA()
linear = LinearRegression()
pipe = Pipeline(steps=[("scaler", scaler), ("pca", pca), ("linear", linear)])
param_grid = {
    "pca__n_components": [5, 10, 15, 20, 25, 30, 35, 40],
}
search = GridSearchCV(pipe, param_grid, n_jobs=2)
search.fit(X_full, y)



```




    GridSearchCV(estimator=Pipeline(steps=[('scaler', StandardScaler()),
                                           ('pca', PCA()),
                                           ('linear', LinearRegression())]),
                 n_jobs=2,
                 param_grid={'pca__n_components': [5, 10, 15, 20, 25, 30, 35, 40]})




```python
n_comp_index = np.where(search.cv_results_['rank_test_score']==1)[0][0]
n_comp_index
```




    7




```python
n = [5, 10, 15, 20, 25, 30, 35, 40][n_comp_index]
```


```python
scaler = StandardScaler()
pca = PCA(n_components = 40)
linear = LinearRegression()
pipe = Pipeline(steps=[("scaler", scaler), ("pca", pca), ("linear", linear)])
```


```python
scorePCA = cross_val_score(pipe, X_full, y, cv= cv, scoring = mse)

```


```python
np.sqrt(abs(scorePCA)).mean(), np.sqrt(abs(scorePCA)).var()
```




    (6.081169021549245, 0.14591921902119961)



However, the purpose of PCA is to reduce the dimension but if we have 40 components, I don't see the benefit of using PCA.


```python
scaler = StandardScaler()
pca = PCA(n_components = 5)
linear = LinearRegression()
pipe = Pipeline(steps=[("scaler", scaler), ("pca", pca), ("linear", linear)])
scorePCA = cross_val_score(pipe, X_full, y, cv= cv, scoring = mse)

```


```python
np.sqrt(abs(scorePCA)).mean(), np.sqrt(abs(scorePCA)).var()
```




    (9.271980860354876, 0.3092042571605515)




```python

```
