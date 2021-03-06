# How DeepAR Works<a name="deepar_how-it-works"></a>

During training, DeepAR accepts a training dataset and an optional test dataset, which is used to evaluate the final trained model\. In general, these two datasets do not have to contain the same set of time series\. A model trained on a given training set can be used to generate forecasts not only for the future of the time series in the training set, but also for other time series\. Both the training and the test datasets consist of \(preferably more than one\) *target time series*, optionally associated with a vector of *feature time series* and a vector of *categorical features*\. See [Input/Output Interface](deepar.md#deepar-inputoutput) for details\. As an illustration, consider the following example for an element of a training set indexed by *i* which consists of a target time series *zi,t* and two associated feature time series *zi,1,t* and *zi,2,t*\.

![\[Figure 1: Target time series and associated feature time series\]](http://docs.aws.amazon.com/sagemaker/latest/dg/images/ts-full-159.base.png)

The target time series may contain missing values, represented by line breaks in the time series\. DeepAR only supports feature time series that are known in the future\. This allows one to run counterfactual "what\-if" scenarios\. What happens, for example, if I change the price of a product in some way? Each target time series can also be associated with a number of categorical features\. These can be used to encode that a time series belongs to certain groupings\. Categorical features allow the model to learn typical behavior for such groups, which can be used to increase the overall model accuracy\. DeepAR implements this by learning an embedding vector for each group that captures the common properties of all time series in the group\. 

## Under the Hood<a name="deepar_under-the-hood"></a>

In order to facilitate the learning of time\-dependent patterns such as spikes during weekends, DeepAR automatically creates feature time series based on the frequency of the target time series\. For example, DeepAR creates two feature time series \(day of the month and day of the year\) for a weekly time series frequency\. These derived feature time series are used along with the custom feature time series provided by the user during training and inference\. The following figure shows two of these derived time series features: *ui,1,t* represents the hour of the day, and *ui,2,t* the day of the week\.

![\[Figure 2: Derived time series\]](http://docs.aws.amazon.com/sagemaker/latest/dg/images/ts-full-159.derived.png)

These feature time series are automatically generated and don't have to be provided by the user\. Here is the list of derived features for the supported basic time frequencies\.


| Frequency of the Time Series | Derived Features | 
| --- | --- | 
| Minute |  minute\-of\-hour, hour\-of\-day, day\-of\-week, day\-of\-month, day\-of\-year  | 
| Hour |  hour\-of\-day, day\-of\-week, day\-of\-month, day\-of\-year  | 
| Day |  day\-of\-week, day\-of\-month, day\-of\-year  | 
| Week |  day\-of\-month, week\-of\-year  | 
| Month |  month\-of\-year  | 

The DeepAR model is trained by randomly sampling several training examples from each of the time series in the training dataset\. Each such training example consists of a pair of adjacent context and prediction windows with fixed predefined lengths\. The `context_length` hyperparameter controls how far in the past the network can see, and `prediction_length` controls how far in the future predictions can be made\. Training set elements containing time series that are shorter than a specified prediction length are ignored during training\. The following figure represents five samples with context lengths of 12 hours and prediction lengths of 6 hours drawn from element *i*\. The feature time series *xi,1,t* and *ui,2,t* have been omitted for brevity\.

![\[Figure 3: Sampled time series\]](http://docs.aws.amazon.com/sagemaker/latest/dg/images/ts-full-159.sampled.png)

To capture seasonality patterns, DeepAR also automatically feeds lagged values from the target time series\. In our running example with hourly frequency, for each time index *t = T*, the model exposes the *zi,t* values which occurred approximately one, two, and three days in the past\.

![\[Figure 4: Lagged time series\]](http://docs.aws.amazon.com/sagemaker/latest/dg/images/ts-full-159.lags.png)

For inference, the trained model takes as input target time series, which might or might not have been used during training, and forecasts a probability distribution for the next `prediction_length` values\. Because DeepAR is trained on the entire dataset, the forecast takes into account patterns learned from for similar time series\.

For information on the mathematics behind DeepAR, see [DeepAR: Probabilistic Forecasting with Autoregressive Recurrent Networks](https://arxiv.org/abs/1704.04110)\. 