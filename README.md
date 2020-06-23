# Using Autoregressive Models and Principal Component Analysis to Predict Harmful Algal Blooms in Lake Erie

Harmful Algal Blooms (HABs) are an ecological phenomenon where, due to nutrient loading or other pollution, certain varieties of algae exhibit explosive growth and may generate toxins that can impact human drinking water supplies. Most notably, in 2017, the drinking water supply for the city of Toledo, Ohio and surrounding areas was impacted by a particularly severe HAB.

https://www.npr.org/sections/health-shots/2017/11/09/563073022/algae-contaminates-drinking-water

HABs, like other biological processes, are extremely complex and difficult to model, especially on the scale of an entire Great Lake. Herein I limit the scope to predicting the levels of particulate microcystin, the toxin that has the potential for human harm, at monitoring site WE2 from the following study:

Burtner, Ashley; Palladino, Danna; Kitchens, Christine; Fyffe, Deanna; Johengen, Tom; Stuart, Dack; Cooperative Institute for Great Lakes Research, University of Michigan; Fanslow, David; Gossiaux, Duane; National Oceanic and Atmospheric Administration; Great Lakes Environmental Research Laboratory (2019). Physical, chemical, and biological water quality data collected from a small boat in western Lake Erie, Great Lakes from 2012-05-15 to 2018-10-09 (NCEI Accession 0187718). NOAA National Centers for Environmental Information. Dataset. https://accession.nodc.noaa.gov/0187718. Accessed 06-14-2020.

![Original Time Series](https://github.com/marbakes/pca-arima-fun/blob/master/images/TimeSeries.jpg?raw=true)

## Autoregressive Models

An autoregressive model is when a value from a time series is regressed on previous values from that same time series. (https://online.stat.psu.edu/stat501/lesson/14/14.1). Autoregressive (AR) models can be used to generate predictions for a wide variety of time series, however they are constrained by a property called stationarity. Basically, stationarity requires that the mean value of a time series does not vary over time. The Augmented Dickey-Fuller test is able to assert stationarity, and lucky for us, the time series of particulate microcystin achieves a p-value of 2.494817e-10, which is small enough to be able to assert stationarity. That's a very long-winded way of saying that we can use an AR model without differencing our data!

The hyperparameter that we can set for our AR model is the order, or the number of lags to use as an input variable. These can be selected using the autocorrelation function (ACF) and partial autocorrelation function (PACF) plots. These plots seem to suggest that the lags of 1, 2, 3, 49, and 52 are significant lags. Note that the original data was roughly on the frequency of 2 weeks and I resampled to apply a 1 week frequency for ease of computation and display. 

![Autocorrelation Plot](https://github.com/marbakes/pca-arima-fun/blob/master/images/ACF.jpg?raw=true)
![Partial Autocorrelation Plot](https://github.com/marbakes/pca-arima-fun/blob/master/images/PACF.jpg?raw=true)

Using only these lags of the previous values of particulate microcystin, I am able to construct a simple linear AR model that achieves a root mean squared error (RMSE) of 0.785 µg/L for the last 52 weeks of the data. As evidenced by the plot below, the AR model correctly assigns a magnitude and time period to the 2018 season peak, however the predicted peak is both shorter and later than the actual. Whether this is close enough to be able to anticipate and reduce the effects of the HAB warrants greater examiniation and testing, however this functions as an illustration of the ultility of AR models for predicting highly seasonal stationary time series. 

![AR Predictions](https://github.com/marbakes/pca-arima-fun/blob/master/images/TimeSeriesAR.jpg?raw=true)

## Principal Component Analysis

Along with the particulate microcystin, 36 other variables were measured in the data. Among these are date and time characteristics, and details regarding the site and personnel making measurements, along with 17 measured chemical and physical properties of the water sample. For the purposes of this analysis, I use these properties to try to predict the particulate microcystin, with the motivation that these can be measured continuously or periodically from water samples and then input into a model to anticipate a particulate microcystin peak. However, with less than 200 values in the model training set, 17 exogenous variables may be excessive and will likely lead to overfitting on the training data. We need a way to capture most of the variation of these exogenous variables while reducing the dimension space to reduce the potential for overfitting, and this is where principal component analysis (PCA) comes in. PCA is a type of matrix decomposition that transforms the initial variable space into a set of new dimentions that contain the most variation in descending order. For the purposes of this model I will select the first 5 principal components to greatly reduce the dimensionality while maintaining about 80% of the variation in the exogenous variables. For each of these principal components (PCs) I fit an AR model using the same lags as before to see if the PCs exibit the same time series behavior as the target variable. For all except the 5th PC the AR model achieves an R-squared over 0.4. This seems to suggest that these PCs can be used as predictors for the target variable.

![Principal Components](https://github.com/marbakes/pca-arima-fun/blob/master/images/PCs.jpg?raw=true)

## Combining Models

The goal is to combine the PCA and the AR model to use the principal components of the water properties to predict the future particulate microcystin levels. Herein I construct a simple linear regresssion where the particular microcystin is regressed against the 1st 5 PCs on the trainind data period. For the future values, each PC is predicted using the AR model described above, and then the particular microcystin is predicted from these values. This combination model achieves a RMSE of 1.02 µg/L, which is on the same order as the AR function, with somewhat higher error. Additionally, this model predicts an even shorter and later peak than the AR model, and I would not recommend this model for prediction of future microcystin peaks, however this serves as an example of the functionality of PCA and AR models in predicting time series.

![AR + PCA Predictions](https://github.com/marbakes/pca-arima-fun/blob/master/images/TimeSeriesARPCA.jpg?raw=true)
