# recipe-ratings-analysis

## Introduction

This will be sonme form of introduction to the project

## Data Cleaning and Exploratory Data analysis

I started by performing a left merge between the two datasets, `recipes` 
(RAW_recipes.csv) and `interactions` (RAW_interactions.csv) together on the 
`id` and `recipe_id` column. I then created an `avg_rating` dataframe that 
displays the means of each recipe, and then merged that with my original 
`recipes` DataFrame and used the `average_rating` column to serve as the 
ratings data for each recipe. Next, I cleaned the `nutrition` column to make the column values
actual lists rather than strings, and then proceeded to parse the cleaned
`nutrition` column to extract protein values. I used these values to define
"high-protein" and "low-protein" groups. The .head()  of the DataFrame is shown
below.

![alt text](images/recipes_head().png)

Then created a couple of distributions of some of the relevant columns in the 
`recipes` dataframe. An example of one such distribution is shown below.

<iframe
  src="assets/log-minutes-plotly.html"
  width="500"
  height="375"
  frameborder="0"
></iframe>

Then proceeded to analyze pairs of columns to check for positive associations.
An example of a pair of columns that showed a weaker association is shown below.

<iframe
  src="assets/minutes-ratings.html"
  width="500"
  height="375"
  frameborder="0"
></iframe>

I then created a pivot table. I used the `minutes` column and pivotted by the
`n_ingredients' column. I filtered out data entries that listed over 10000 
minutes to make as these were not "real" recipes and were posted in the data as
a sort of joke, and entries that had over 15 ingredients since a vast majority 
of the recipes required 15 or less ingredients. The resulting pivot table is 
shown below.

![alt text](images/pivot-table.png)

## Assessment of Missingness

I believe that the `description` column is NMAR (Not Missing At Random). After
running several permutation tests on the data in the `recipes` DataFrame, there
were a couple columns, `n_ingredients` and `average_rating`, that the
missingness of descriptions depended on. I parsed through the dataset and ran 
permutation tests on each column using this function

```py
def permutation_test_fast(col_missing, col_other, n_permutations=500):
    col_other_array = col_other.to_numpy()
    col_missing_array = col_missing.to_numpy()
    observed_stat = col_other_array[col_missing_array].mean() - col_other_array[~col_missing_array].mean()
    perm_stats = []
    for _ in range(n_permutations):
        shuffled = np.random.permutation(col_other_array)
        perm_stat = shuffled[col_missing_array].mean() - shuffled[~col_missing_array].mean()
        perm_stats.append(perm_stat)
    p_value = np.mean(np.abs(perm_stats) >= np.abs(observed_stat))
    return observed_stat, p_value, perm_stats
```

A `plotly` plot that demonstrates such missingness exploration is shown below

<iframe
  src="assets/ingredients-missingness.html"
  width="500"
  height="375"
  frameborder="0"
></iframe>

## Hypothesis Testing

#### Question: Do recipes with more protein have higher average ratings?

- Null Hypothesis: There is no difference in the average ratings of recipes with 
higher protein content compared to those with lower protein count

- Alternate Hypothesis: Recipes with higher protein count tend to have higher average ratings

The test was performed using the "difference in means" test-statistic at the 
0.05 significance level. The code for the hypothesis test is:

```py
from scipy.stats import ttest_ind

#Perform a two-sample t-test
t_stat, p_val = ttest_ind(high_protein, low_protein, nan_policy='omit')

# Output results
print(f"T-statistic: {t_stat}, P-value: {p_val}")

alpha = 0.05  # Significance level
if p_val < alpha:
    print("Reject the null hypothesis: Recipes with higher protein content have higher average ratings.")
else:
    print("Fail to reject the null hypothesis: No significant difference in ratings.")
```

The p-value was 0.79, which means we failed to reject the null hypothesis. In 
other words, the test indicates that there was no statistically significant
difference in the average ratings of recipes with higher protein content 
than those with lower protein count.

I visualization pertinent to this hypothesis test is below

<iframe
  src="assets/hypothesis-test-visualization.html"
  width="500"
  height="375"
  frameborder="0"
></iframe>

## Framing a Prediction Problem

The prediction problem I chose was to predict the ratings of recipes, which is
a Regression problem. We are going to target the `average_rating` (continuous)
variable because it directly represents the aggregated user feedback (ratings) 
for each recipe. It's the outcome I want to predict. The metric I am going to
use for my model is Root Mean Squared Error (RMSE). I am using RMSE because it 
measures the average prediction error in the same units as the target variable
(`average_rating`). making it interpretable. RMSE also penalizes large errors 
more than small ones, which is important when predicting user ratings because 
significant prediction errors (e.g., predicting 2.0 instead of 5.0) can lead to 
poor recommendations.

## Baseline Model 

"The baseline model uses a linear regression algorithm with one quantitative 
feature (`minutes`) and one nominal feature (`tags`), which was encoded using 
one-hot encoding. While the model serves as a simple starting point, its 
performance is poor, with an RMSE of 0.6509 and an R² of -0.0480. This suggests 
that the current features and linear approach do not adequately explain the 
variance in `average_rating`. Future models will incorporate additional features 
and more complex algorithms to improve performance."