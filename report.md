# Assignment 1: Housing Price Prediction — Written Report

IE500618 — Group 13: Magnus Grande, Ida Soldal

## Part A - Answers to Q1-Q7

***Q1. What were the most important things you learned from exploring the dataset? Refer to relevant statistics, missing values, distributions, unusual observations and visualizations.***

**Answer:**

SalePrice is right-skewed with a skewness of `1.74`, mean `$180,796` vs median `$160,000`, and a long tail of very expensive houses, meaning that the mean is not a good measure of central tendency.

27 columns had missing values, mostly meaning "no such feature" (e.g. no alley, no pool, no fence).

We also identified three unfinished partial sales above 4,000 sq ft sold far below what their size suggests, so we removed them from the training data to avoid skewing the models.

See the relevant sections in part 1 of the notebook for further details and reasoning.

The most important things we learned came in regards to missing and unusual values. The "rest" of the work felt fairly textbook, like we were walking the intended path, but learning how to identify and handle issues with the data is generally more unique to each dataset so learning how to approach them without a known solution will be of great use to us in future work with machine learning.

***Q2. Which features appeared to be useful for predicting house prices? What evidence led you to this conclusion? Refer to relevant visualizations, descriptive statistics, correlations or your understanding of the variables.***

**Answer:**

The features that appeared to be most useful for predicting house prices were `Gr Liv Area` (above-ground living area), `Overall Qual` (overall material and finish quality), and `Garage Cars` (size of garage in car capacity) due to their strong correlations with SalePrice. The correlation coefficients for these features were 0.72, 0.79, and 0.64 respectively, indicating a strong positive relationship with SalePrice.

Additionally, `Neighborhood` was also found to be a useful feature, as it captures the location of the house, which is often a significant factor in determining its value.

This does read as a logical conclusion, as these features are all related to the size and quality of the house, which are generally expected to be strong predictors of price.

***Q3. How did you create the training and test sets? Why should the test set be kept separate during model development, and how does this help provide an unbiased evaluation on unseen data?***

**Answer:**

The training set was split from the original dataset using a random 80/20 split. The test set was kept separate to ensure that the model's performance could be evaluated on unseen data, without which we wouldn't be able to assess how well the model generalizes to new data. This helps provide a more accurate measure of the model's predictive capabilities.

(Another reason to keep them split was that we were explicitly told to do so by the assignment :)

***Q4. How did you handle missing values and categorical variables? Explain your main preprocessing decisions, including scaling or transformations if used. Why should preprocessing be learned from the training data and then applied to the test data?***

**Answer:**

Missing values: we handled each column according to its description and used related columns to tell the two types of NaN-values apart. Columns where missing means "no such feature" (`Alley`, `Fireplace Qu`, `Pool QC`, `Fence`, `Misc Feature`) got a `None` category. For basement, garage and veneer columns, a missing value became `None` when the related size column (`Total Bsmt SF`, `Garage Area`, `Mas Vnr Area`) was 0, and `Unknown` otherwise, since the feature exists but was not recorded. Unrecorded numbers (mainly `Lot Frontage`) got the training median, and the one missing `Electrical` value became `Unknown`. We also dropped `Garage Yr Blt` (redundant with `Year Built`, and meaningless without a garage) and removed three unfinished "Partial" sales above 4,000 sq ft from the training data, as they sold far below what their size suggests.

Categorical variables were one-hot encoded, including `MS SubClass`, whose numbers are house-type codes rather than quantities. Numeric columns were standardised, which mainly matters for linear regression. We did not transform `SalePrice`, so RMSE stays in dollars which seemed to be suggested by the assignment.

The medians, encoder categories, means and standard deviations are all learned from the training data only and then applied unchanged to the test data. Otherwise information from the test set leaks into the preprocessing and the test score becomes too optimistic, since the test set should behave like data we have never seen.

***Q5. Which features did you finally use for prediction? Did you remove any features or create new ones? Explain the reasoning behind your main feature-selection or feature-engineering decisions.***

**Answer:**

We ended with 212 features. Of the 81 original columns we removed `Order` and `PID` (identifiers) and `Garage Yr Blt` (redundant with `Year Built`, correlation 0.83). After encoding, we also removed 126 features whose correlation with `SalePrice` in the training data was below 0.05 in absolute value: 119 one-hot categories and 7 numeric columns such as `Yr Sold` and `Mo Sold`. `Street` and `Utilities` lost all their categories, which fits Step 1, where they explained almost none of the price variation.

We created two features. `Total SF` (`1st Flr SF` + `2nd Flr SF` + `Total Bsmt SF`) has a correlation of 0.83 with price, stronger than `Gr Liv Area` (0.72) or `Total Bsmt SF` (0.65) alone. `House Age` (`Yr Sold` - `Year Built`) is how old the house was when it sold (correlation -0.56). The rest were kept, including the strongest predictors from Step 1: quality, living area, garage and neighbourhood.

***Q6. What RMSE and R² values did you obtain for each of the three models on the test data? Present your results clearly in a table. Which model performed best on the test data?***

**Answer:**

Results on the 20% test set (one random split):

| Model | Test RMSE | Test R² |
| --- | --- | --- |
| Linear Regression | `$26,817` | 0.891 |
| Decision Tree Regression | `$38,383` | 0.776 |
| Random Forest Regression | `$22,634` | 0.922 |

Random Forest performed best on both measures. Its RMSE is about 14% of the median house price, and its squared errors are about 8% of those from always predicting the training mean (RMSE `$81,148`, R² about 0). Linear Regression was fairly close. The Decision Tree was clearly worst: it fits the training data perfectly (R² 1.000) but only reaches 0.776 on the test data, so it overfits.

***Q7. Based on what you learned from the analysis, what could you change in the preprocessing or features to potentially improve the prediction results without changing to a different model?***

**Answer:**

Based on what we saw, two changes look most promising.

First, log-transform `SalePrice`. It is right-skewed (skewness 1.74, about 0 after the log), and the most expensive 10% of test houses account for 22% to 37% of each model's squared error. The log compresses that tail and turns the model's effects into percentage changes instead of dollar amounts, which suits linear regression better.

Second, remove redundant features. `Total SF` is the sum of `1st Flr SF`, `2nd Flr SF` and `Total Bsmt SF`, which are all still in the data, and `Garage Cars` and `Garage Area` overlap heavily (correlation 0.89). Dropping one of each overlapping group would stop the linear model from getting the same information twice.

## Part B - Group-work reflection

***Q1. How was the work divided among group members, and what did each member contribute?***

**Answer:**

We sat down and worked out a simple solution to the assignment together. We then implemented this using AI tools to quickly generate the notebook and rough skeleton.

Following this, we took some time to think about our approach, and later sat down together to work through our final solution together, editing and improving the existing implementation. Additionally, we had a short discussion in-person before the feedback discussion with the TA to ensure we both understood the full solution.

***Q2. Which important decisions were made together as a group?***

**Answer:**

Every major decision was made as a group. Despite using AI to generate the code, we used the code review process we're familiar with as software engineers to ensure that we both understood the code that was being implemented *before* it was added to the notebook.

The remaining individual decisions mostly dealt with the presentation of the results, and did not affect the final solution.

***Q3. How did you ensure that everyone understood the complete solution, not only their own part?***

**Answer:**

By coworking rather than splitting the project into parts.

***Q4. Was the work distributed fairly? Explain briefly.***

**Answer:**

Yes.

We're both happy with the distribution, given we both actively collaborated for the majority of the process.
