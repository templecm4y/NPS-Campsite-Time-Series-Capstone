# Forecasting Campground Pressure using Time-Series Analysis
## Using aggregated National Park Service data to project campground usage from 5 years of historical data

- Temple Moore

## Problem Statement
The Recreation Information Database (RIDB) was created by the National Park Service (NPS) to provide information about federal recreation areas, facilities, campsites, tours, and permits. RIDB (https://ridb.recreation.gov/ ) is a single point of access to information about recreational opportunities nationwide, its API can be used to dynamically retrieve the availability of various sites managed by federal agencies. Historical reservation data is available to the public, and contains millions of records. I downloaded 5 years of data and 15,187,728‬ individual reservations, 2,784,270‬ of which were campground reservations made through the NPS.

From this data, I want to observe and forecast trends in campground reservations for five of the most popular NPS campgrounds. The NPS does not provide data for the individual campsites or daily site data. All data used in this project will have to be aggregated from the reservations taken from the RIDB.

2016 was a record popular year for  Park Service, coupled with the fact that it was the centennial for the agency. 331 Million people visited the park service in 2016, and Congress and American citizens alike are concerned about the implications of this increased pressure. In response, the current administration has suggested to raise entry fees for some of the most popular parks to as much as $75.  While this policy has been stepped by to more moderate increases, it reflects a growing concern in the agency that our parks are becoming too congested. (https://crsreports.congress.gov/product/pdf/IF/IF10151) My goal is to use the observations to make conclusions about the NPS's best practices for park entry and campground reservation fees.

## Methodology
The following is how I aggregated the data and used SARIMA modeling to forecast campground usage.

**NOTE** At the moment, there is no data in this repository. However, it has been uploaded to an S3 and bucket and AWS functionality will be added to the modeling notebook at a later date.

- **Step 1: Data Acquisition**: I used the historical reservation data from RIDB from the years 2014 to 2018. From these 15,187,728‬ individual reservations, 2,784,270‬ of which are campground reservations, the data needed to be aggregated by date in order to allow for proper time series analysis. This was the biggest challenge in undertaking this project, as the reservations only report the date the reservation is made, and the date range the site will be reserved. Each individual camp site has a unique facility ID per the RIDB API, and I was able to use downloaded API data to create uniform records for each campsite.

- **Step 2: Aggregation**: This was the most time consuming aspect of the project. On average, my python code took 20 minutes to group and aggregate the data by date and facility ID. I used much of the built in Functionality of the Pandas module to accomplish this, but had to be creative with separating the reservations by date. This required a filter test to see if a certain date was within the range of each reservation, and from there the reservations were aggregated appropriately to create the features. In addition to the sum of individual reservations, number of people at the site, the total revenue from fees paid, I created features for the average length of stay for reservations for a given day, the average booking horizon for those reservations, and the average fee paid. My target variable was created by getting a percentage of the campsites booked for a single day. I had to use the RIDB API to extract the total campsites for each site, and divide the daily value to get this percentage.

Combining the aggregated data, I was left with 198,042 entries representing daily data for each site. Note that each site has its own off-season where you cannot make reservations, so this was not a continuous time-series. From here, I was able to observe the top sites for each variable, and selected five campsites for further analysis. These campsites were Big Meadows in Shenandoah National Park, Mather Campground in Grand Canyon National Park, Moraine Park in Rocky Mountain National Park, Elkmont Campground in Great Smoky Mountain National Park, and Upper Pines Campground in Yosemite National Park. I separated the data for each of these campgrounds to model individually, as they each display unique seasonality and characteristics. In addition, some camp sites exhibited a daily booking percentage of over 100%, which had to be truncated to 100% for modeling purposes.

- **Step 3: Pre-Processing and Modeling**: While my original goal was to combine several different models and weight the average for each based on performance, a SARIMA forecast was efficient and capable of effective predictions. I was able to train a Recurrent Neural Net on the Moraine Campground, however, the source for this is not yet available in this repository, and will be added at a later date when completed.

To prepare for the time series analysis, I ensured each site's data had a data-time index, grouped the data by week appropriately for easier seasonality evaluation, and filled in missing days with zeroes. The missing reservation data for each site represents its own offseason, and other values should not be imputed.

For each site's data, I evaluated its stationarity with an Augmented Dickey Fuller Test (ADF) and  Kwiatkowski–Phillips–Schmidt–Shin (KPSS) Test. Additionally, the Osborn, Chui, Smith, and Birchenhall (OCSB) Test Canova and Hansen (CH) Test were used to determine if differencing was necessary.

Each SARIMA model was trained using the pmdarima (pronounced "Pyramid") module, which is an emulation of R's Auto Arima functionality. It functions similarly to a 'grid search', and returns the best performing model according to its Bayesian and Akaike Information Criterions, BIC and AIC respectively. Additonally, as the limit to the data was between 0 and 1, representing percentage, I used a logarithmic transformation (https://otexts.com/fpp2/limits.html) to constrain the predictions in all but one of the models.

Each SARIMA model will be used to predict a 60 week forecast. Then, using half of the 60 week testing data, will update and predict a 30 week forecast. Some models were used to predict 104 week periods on unseen data as well to observe trends.

Each SARIMA model was evaluated using Mean Absolute Error (MAE), Root Mean Squared Error(RMSE), and a Forecast Bias Test.

  - **Moraine**: Moraine park exhibits a somewhat stationary 52-week seasonality, but shows more volatility than Moraine during its season. The ADF test boasts a p-value of 0.01, indicating that the data is stationary, but KPSS p-value of 0.1 indicates that the data has a unit-root present. However, both the OCSB and CH Test indicated no need for differencing.

    - The SARIMA(1,1,1)(0,1,0)52 model on the Moraine data was by far the best performing out of all the forecasts. It reported a RMSE of 4.20%  and a 2.34% MAE on a 60 week forecast. On the 30 week updated forecast, the model scored a RMSE of 6.19% and a MAE of 2.34%.

    - The Moraine forecast indicates that this site maintains a relatively stable booking rate during its season from May to October. The two year predictions shows that this seasonality is extremely predictable and very stable.

  - **Mather**: Mather exhibits rather stationary 52-week seasonality. The ADF test boasts a p-value of 0.0305, indicating that the data is stationary, but KPSS p-value of 0.1 indicates that the data has a unit-root present. However, both the OCSB and CH Test indicated no need for differencing.

    - The SARIMA(3,1,2)(0,1,0)52 model on the Mather data was adequate in capturing its seasonality, but still saw significant error. It reported a RMSE of 24.50%  and a 12.69% MAE on a 60 week forecast. On the 30 week updated forecast, the model scored a RMSE of 22.32% and a MAE of 17.90%.

    - Mather was an interesting case as each October sees a significant spike in booking percentage, often reaching 100% or higher. Mather's forecast was more bullish on its trend, which saw a significant increase in the 60 week forecast. However, on updating the model with 30 weeks of testing data, the model's projects dropped to indicating a decreasing trend.

  - **Big Meadows**: Big Meadows was unique among the other sites as it's average booking percentage was lower than the others during its season, averaging 56.22% between May and October. The ADF test boasts a p-value of 0.064, indicating that the data may not be stationary, and KPSS p-value of 0.1 indicates that the data has a unit-root present. However, both the OCSB and CH Test indicated no need for differencing.

    - The SARIMA(1,1,1)(1,1,0)52 model on the Big Meadow data captured it seasonality, but predicted a large uptick in booking. It reported a RMSE of 24.38%  and a 17.42% MAE on a 60 week forecast. On the 30 week updated forecast, the model scored a RMSE of 21.86% and a MAE of 17.90%.

    - Big Meadows, being a the smallest and most Eastern of the campgrounds selected, is an interesting case in observing rising trends in park usage. Its seasonality looks stable, but sees sharp upticks in the late season, likely due to park visitors coming to see the fall foliage. The two year prediction especially looks volatile, but observes a trend of increased rate of reservations.

  - **Upper Pines**: Upper Pines is opposed to Big Meadows in that it is by far the most popular campground in this analysis, boasting an average booking percentage of 98.02% during its peak season. The ADF test boasts a p-value of 0.014, indicating the data is stationary, and KPSS p-value of 0.1 indicates that the data has a unit-root present. However, both the OCSB and CH Test indicated no need for differencing. However, upon observation of the Upper Pines series, it becomes apparent that the campsite began to allow year round reservations in 2016, which required a different approach to modeling this data. A logarithmic transformation threw off the results dramatically, but incorporating exogenous variables into the model improved performance. I selected a lagged exogenous variable of the Average Stay Length, Average Booking Horizon, and the Average Fee for the SARIMAX model.

    - The SARIMAX(1,1,1)(2,1,0)52 model was very accurate and captured the unique seasonality well. It reported a RMSE of 13.39%  and a 10.77% MAE on a 60 week forecast. On the 30 week updated forecast, the model scored a RMSE of 9.46% and a MAE of 7.84%.

    - Upper Pines was a very unique case, but adapting my modeling approach to use lagged exogenous gave my forecast more accuracy an power. However, the two year forecast was not as convincing, showing a sharp decrease in booking and even significant negative values. This is a disadvantage of not using the logarithmic transformation, and indicates the exogenous factor could have been overfit to the model.

  - **Elkmont**: Elkmont was more traditional in its seasonality compared to Big Meadows and Upper Pines. The ADF test boasts a p-value of 0.033, indicating the data is stationary, and KPSS p-value of 0.1 indicates that the data has a unit-root present. However, both the OCSB and CH Test indicated no need for differencing. I again used logarithmic transformation on the Elkmont endogenous variable to limit predictions.

    - The SARIMA(2,1,1)(0,1,0)52 model was accurate and captured some unique trends. It reported a RMSE of 10.77%  and a 5.9% MAE on a 60 week forecast. On the 30 week updated forecast, the model scored a RMSE of 11.35% and a MAE of 7.9%.

    - Elkmont is a very popular campground that, like Big Meadows, sees a sharp uptick in the late season likely due to changing foliage and milder weather. The two year forecast was able to effectively capture this trend.

## Results and conclusions
In the campsites I did feel comfortable running predictions for, we see relatively stable seasonal trends that can differ from site to site. With the exception of Big Meadows, I was able to accurately predict a 60 week forecast for each campground. In each instance, I did not observe any drastic change in occupancy during the season, inferring that this trend can and will continue under the services provided by the NPS.

I do believe a higher park entry fee would slow down at-capacity or near capacity bookings for these campsites, and I do understand the limitations the park service in under in expanding and maintaining these campgrounds. Its amazing that so many Americans are utilizing these facilities to experience parts of what makes this country so beautiful, but Congress and the Administration need to consider alternatives to slow down or limit the pressure put on these parks during their peak seasons.
