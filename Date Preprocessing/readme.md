# Data Preprocessing

We first obtained the weather data using `rnoaa` R package. Then using the imputation method described in the paper, we filled in the weather data. Lastly, we extracted monthly weather data as well as other variables indicating weather trend throught the winter and spring. Lastly, we also included how the `rFSA` package was used to find interaction terms. We organized subdirectories according to the topics.

## Steps for data preprocessing
1. Obtaining meteorology data from rnoaa package
2. Imputing NA values for weather variables
3. Defining Training Dataset
4. Finding Interaction Terms using rFSA

### CSV files in the codes:
* *korea_complete.csv*: It is the output dataset after obtaining weather data from `rnoaa` package.
* *longdata.csv* : longdata.csv contains weather information for all the cities and the weather information is already imputed. Therefore, longdata.csv is used to extract monthly weather data as well as general weather trend throughout the winter and spring.
* *lee.csv*: dataset to be used to fit the models.
