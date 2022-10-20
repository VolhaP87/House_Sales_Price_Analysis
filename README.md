![](Images/intro-1625185676.webp)

# House Sales Price Analysis
**Author:** Volha Puzikava
***

## Overview
This project analyzes house sales in a northwestern county in order to develop a pricing algorithm that can help real estate agencies and homeowners to sell houses. 
***

## Business Problem
The XYZ Realty asked to analyze the King County House Sales dataset and provide advice about how the number of floors in homes might increase or decrease the estimated value of those homes, and by what amount. The main purpose of the analysis was to create an algorithm that would predict the best price for selling homes based on the known characteristics of those houses.
***

## Data Understanding
The data for the analysis was taken from the King County House Sales dataset. The data provided various information about house sales: unique identifiers for houses, dates homes were sold, sale prices, number of bedrooms and bathrooms the houses had, square footage of living space in the houses, square footage of the lots, etc. The dataset contained 21,596 houses sold from 2014 to 2015 in the King County, Washington.
***

## Data Preparation
The dataframe was checked for the presence of any NaNs. In the "waterfront" and "view" columns all null values were replaced with the string "Unknown" to indicate real categories. The "yr_renovated" column had NaNs replaced with its median.

The "sqft_basement" column contained "?" values. After examining the description of the column names and data in those columns, it was found out that square footage of the living space in the home ("sqft_living") was equal to the sum of square footage of that house apart from basement ("sqft_above") and square footage of the basement in the house ("sqft_basement"). Thus, "?" values in the "sqft_basement" column were replaced with the appropriate measurements. Besides, the format of the "sqft_basement" and "zipcode" columns were converted to float and string types respectively.

In order to be used in a model, categorical variables had to be transformed. Because the values of the categorical variables were ordinal, i.e. they represented some kind of intensity, a map that reflected each value to a number was created.
***

## Data Exploration
The histograms of all the numeric variables in the dataset were plotted. Most of the resulted distributions appeared to be skewed. While linear regression does not assume that each of the individual predictors should be normally distributed, it does assume a linear relationship between the predictors and the target variable ("price"). In order to investigate if this assumption held true, single variable regression plots of each feature were plotted against the target variable using seaborn. Based on the resulted plots, the linear relation was not seen between the target and the following predictors: "waterfront", "condition", "yr_built", "yr_renovated", "lat", and "long".

It was also important to see if the predictive features would result in multicollinearity in the final model. With that in mind, pearson correlation coefficients of the predictive features were generated and visualized. According to the heatmap, the highest correlation belonged to "sqft_living" and "sqft_above" (0.88). Also, "sqft_living" was most strognly correlated with the target variable (0.7). Strong correlation was also present between "sqft_living" and "sqft_living15" (0.76), "sqft_living" and "grade" (0.76), "sqft_living" and "bathrooms" (0.76), "sqft_lot" and "sqft_lot15" (0.72).
***

## Data Modeling
In order to build a regression model, a train-test split should be performed. The prediction target for this analysis was the price of the houses, so the data was separated into a train set and test set accordingly. The "id" column was dropped since it represented a unique identifier, not an actual numeric feature.

The heatmap was generated to check for multicollinearity in the train set. According to the heatmap, the most correlated variable with the target was "sqft_living". That feature was used to build a linear regression model, that was used as the baseline model. The baseline model was then evaluated using cross_validate and ShuffleSplit, the main idea of which was to perform 3 separate train-test splits within the X_train and y_train, and find the average scores for each. Also, the high correlation was again seen on the heatmap between "sqft_living" and "sqft_above" (0.88), "sqft_living" and "sqft_living15" (0.76), "sqft_living" and "grade" (0.76), "sqft_living" and "bathrooms" (0.76), "sqft_lot" and "sqft_lot15" (0.71).

Because the p-value of the baseline model looked good, but the R-square was low enough (0.49), the second model was created. As the predictors with overly high pairwise-correlation (over 0.65) are almost certain to produce multicollinearity in the final models, the columns "sqft_above", "sqft_living15", "sqft_lot15", "grade", and "bathrooms" were dropped. The columns "waterfront", "condition", "yr_built", "yr_renovated", "lat", and "long" were not included either because of the absence of the linearity assumption with the target variable. The train and validation scores got better compared to the scores of the baseline model. However, the model was rerun in StatsModels to check the p-values of the predictors.

According to the summary, "sqft_basement" was not statistically significant, since its p-value was higher than 0.05. Thus, the third model was created, in which the column with square footage of the basements was dropped. The third model was then rerun in StatsModels to check the p-values again and evaluated using cross_validate and ShuffleSplit.

The removal of the feature with high p-value didn't influence much on the resulting scores. The RFECV was used to select the best features. "RFE" ("recursive feature elimination") repeatedly scored the model, found and removed the features with the lowest "importance" until the minimum was reached. The resulted algorithm stated that the third model was the best one. A different strategy was applied to see if a better model could be found. The code went over multiple different permutations of the columns to see if we could find something better than p-values approach or the RFECV approach.

According to the results of the Brute Force Approach, the best model happened to be our third model. A list of best_features, which contained the names of the best model features based on the findings, was created, and the data was prepared for modeling. LinearRegression model called final_model was instantiated, then fitted on the training data and scored on the test data.

But before assuming the resulted coefficients gave the inferential insight into the pricing algorithm, the assumptions of linear regression were investigated, in order to understand how much the final model violated those assumptions.

Three of the four assumptions were violated. Based on the plots, it seemed as though outliers were having a substantial impact on the model. The target variable was checked for outliers.

Because the distribution of the target was skewed, Inter-Quartile Range (IQR) proximity rule was used for outliers detection. The rule states that the data points which fall below Q1 – 1.5 IQR or above Q3 + 1.5 IQR are outliers, where Q1 and Q3 are the 25th and 75th percentile of the dataset respectively, and IQR represents the inter-quartile range, given by (Q3–Q1). Thus, the boundary values were found and outliers were excluded from the analysis. As a result, 1,158 rows were not included into the analysis.

Once the outliers were removed, the new DataFrame was split into train and test sets. LinearRegression model called final_model was instantiated using the best_features, then fitted on the training data and scored on the test data. The linear regression assumptions were then checked on the resulted model without outliers.

Two of the four assumptions were violated. The best solution was to remove highly correlated predictors from the model. All the predictors with high vifs were dropped and a list, which contained only the features with vif less than 5, was created. The linear regression assumptions were then checked on the resulted model without outliers and high vifs.

Only the rule of homoscedasticity was still violated.

According to the resulted model, the base price for a house in the King County is about 148,475.59. Then for every square footage of living space in the home, the price goes up by 163.06, and for every square footage of the lot, the price goes down by 0.06. Then finally, the quality of view from the house increases its price by 35,332.79 per numerical value of the feature.
***

## Model Evaluation
Based on the findings, we can conclude, that the resulted model is not strong enough, but with only three features, 39.4% of the model performance was recovered. We should not solely rely on the resulted coefficients, since they violate homoscedasticity assumption of linear regression. The coefficients should be used only for predictive purposes.
***

## Conclusions
The analysis of the King County House Sales dataset resulted in the development of a pricing algorithm that can help real estate agencies and homeowners to sell houses. Although, the XYZ Realty asked for an advice how the number of floors in homes increases or decreases the estimated values of those homes, the variable "floors" was not included in the final model, meaning that it is not a statistically significant feature. According to the analysis, the predictors of the target variable ("price") are square footage of living space in a house, square footage of the lot, and the quality of view from the house. Thus, the model illustrates that the base price for a house in the King County is about 148,475.59. Then for every square footage of living space in the home, the price goes up by 163.06, and for every square footage of the lot, the price goes down by 0.06. Then finally, the quality of view from the house increases its price by 35,332.79 per numerical value of the feature. 

The resulted model happened to be not strong enough and may be used only for predictive purposes. The R-square of the final test set (0.39) happened to be lower than the validation score of the baseline model (0.49). Also, the rule of homoscedasticity was violated in the final trial. All this can be explained by the following:
 -  the model can't perfectly fit all the datapoints in a linear regression model in real life, as in real models there are always outliers and residuals;
 -  the task is to find the relationship or "the best fit line" (not prefectly fit line) between the target and predictors;
 -  low R-square is not always a problem. R-square measures the scatter of the data around the regression lines. Higher variability around the regression line produces a lower R-square value;
 -  transformed categorical variable ("view") was used in the final model, however, sometimes the effect of categorical variable on the output can be misleading as the assigning of categorical variables is more subjective.
 
Further analysis of the dataset may be necessary, such as using only numerical features for the development of the pricing algorithm, without any transformed categorical variables in them.
***