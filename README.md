# tutorial
The package include three scripts files: - run_explore, - run_trainmodel, - run_forecast. As the name indicates, they each provide data analysis, data modelling and forecast and evaluation. In the following, the stop/travel times are referred as dependant variables.


1. run_explore

the information gain were first calculated for the independent variables for each dependant variable. It was found that, the Start time is relevant and so day trend may exist for dependent variables with regard to the Start time. This observation was confirmed by average trend extraction.
The step is time consuming. The relevant codes are commented out. A pre-generated file info_gain.txt is included in the output folder.
The bus dispatch intervals are first investigated. This is just to confirm that it is not meaningful to resample the records and make the series evenly distributed over time. Instead, the dependent variables are treated as sequential series only with respect to their temporal order. The time stamps are later used in trend look up, but not considered in residual modelling.
In average trend extraction, for each dependant variable, all the historical samples are aligned according to their Start time without considering the date. The align the series is cleaned, smoothed and resampled to form a trend template for every dependant variable. Two way-EWMA filtering is used here to get the smoothed day trend without time-shifts.
Smooth ratio. <ALPHA> is used here to control the smoothness.
Then the crosscorrelation between detrending residuals are investigated. It’s found that residuals mostly are not cross-correlated. Only one-pair have low correlation. We can assume there is no linear relationship between different dependant variables after detrending.
Then, the autocorrelation and stationarity analysis is performed on both the original series and the detrained 
residuals to find the tentative p, d, q order for ARIMA model.


2. run_trainmodel

ARIMA model were used to describe both the original series and detrended residual.
It is assumed the process can be describe by a static model. Thus the classical ARIMA model updating procedure (predict-include new data-reestimate ) is not used. Instead, we
1. Select ARIMA models using training dataset, fix the model and then
2. Use the fitted model to get predictions conditional on the observations in the test dataset
Step2 process is actually a filtering process. Step1 is filter optimization.
2/3 data were used for model training. The fitted models are pickled in output/pkl folder.

TODO: the p, d, q were selected roughly by simple confidence level checking. They should be
manually adjusted by checking the plots and selecting among multiple difference orders. But I didn’t do that yet. Some ARIMA models didn’t converge, or some models have close-to-unit root. That’s why the ARIMA models performed really bad on some dependent variables in the forecasting step. The ARIMA modelling and forecasting code is just for illustrating the process.


3. run_forecast.

The trained models are read and used for forecasting the remaining 1/3 data. A couple of methods are used for forecasting:
1. ARIMA on original series
2. ARIMA model + trend template
3. Average trend template
4. Trend template + exponentially weight moving average of the day history (with ALPHA2) 5. Dummy mean
The mean absolute error (MAE) reveals that Trend+EWMA performs the best. The MAE reduced about 10%. The 75% quantile is also the smallest among the the methods.
It is no surprise, as Trend+EWMA model integrates the trend information in all the training data and the day series’ dynamic information. Two parameters ALPHA and ALPHA2 controls the performance. TODO: They should be tuned to improve the performance.
ARIMA model + trend template performs well in some cases. If the problem in ARIMA model fitting is fixed and approximate ARIMA model is estimated for all the residuals, this method may also generate good results.

NOTE:
1. The generated figures and results are saved in folder ./output/....
2. Include the folder “sampen" in the working directory for calculating sample entropy.
TODO: The run-time SettingWithCopyWarning warnings are due to the use of pd.Datafram.loc[index][field], instead of pd.Datafram.loc[index,field]. In this project, the warning doesn’t influenced the results, but need to be fixed.
