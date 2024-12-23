# recipe-ratings-analysis

## Introduction

This project analyzes two datasets: RAW_interactions.csv and RAW_recipes.csv. 
Together, they provide insights into user interactions with recipes, including 
ratings and reviews, and details about the recipes themselves.

The first dataset, RAW_interactions, contains 731,927 rows and 5 columns. It 
records user interactions with recipes, including the following columns:

- `user_id:` A unique identifier for each user.
- `recipe_id`: A unique identifier for each recipe.
- `date`: The date when the interaction occurred.
- `rating`: A numerical rating provided by the user for a recipe.
- `review`: A text review left by the user.

The second dataset, RAW_recipes, contains 83,782 rows and 12 columns. It 
provides detailed information about recipes, including:

- `name`: The name of the recipe.
- `id`: A unique identifier for the recipe.
- `minutes`: The time required to prepare the recipe.
- `contributor_id`: The unique identifier of the contributor who submitted 
the recipe.
- `submitted`: The date the recipe was submitted.
- `tags`: Tags describing the recipe's attributes (e.g., vegan, dessert).
- `nutrition`: Nutritional information for the recipe.
- `n_steps`: The number of steps required to complete the recipe.
- `steps`: A detailed list of instructions for the recipe.
- `description`: A brief description of the recipe.
- `ingredients`: A list of ingredients used in the recipe.
- `n_ingredients`: The number of ingredients used.

**Question of Interest:** How do user ratings and reviews correlate with recipe 
attributes such as preparation time, number of ingredients, and recipe tags? 
By exploring this, we can identify patterns and factors that contribute to 
highly-rated recipes, providing insights for both recipe creators and users.

**Why This Question Matters:** Understanding the connection between user 
preferences and recipe attributes can help improve recipe 
recommendations, enhance user experience, and guide recipe contributors in 
crafting popular recipes. This analysis is particularly valuable for culinary 
enthusiasts, content creators, and platforms hosting user-generated recipes.

## Data Cleaning and Exploratory Data analysis

I started by performing a left merge between the two datasets, `recipes` 
(RAW_recipes.csv) and `interactions` (RAW_interactions.csv) together on the 
`id` and `recipe_id` column. I then created an `avg_rating` dataframe that 
displays the means of each recipe, and then merged that with my original 
`recipes` DataFrame and used the `average_rating` column to serve as the 
ratings data for each recipe. Next, I cleaned the `nutrition` column to make the column values
actual lists rather than strings, and then proceeded to parse the cleaned
`nutrition` column to extract protein values. I used these values to define
"high-protein" and "low-protein" groups. The .head() of the DataFrame with 
pertinent columns is shown below.

| name                                 |   minutes | nutrition                                     |   n_steps |   n_ingredients |   average_rating |
|:-------------------------------------|----------:|:----------------------------------------------|----------:|----------------:|-----------------:|
| 1 brownies in the world    best ever |        40 | [138.4, 10.0, 50.0, 3.0, 3.0, 19.0, 6.0]      |        10 |               9 |                4 |
| 1 in canada chocolate chip cookies   |        45 | [595.1, 46.0, 211.0, 22.0, 13.0, 51.0, 26.0]  |        12 |              11 |                5 |
| 412 broccoli casserole               |        40 | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0]     |         6 |               9 |                5 |
| millionaire pound cake               |       120 | [878.3, 63.0, 326.0, 13.0, 20.0, 123.0, 39.0] |         7 |               7 |                5 |
| 2000 meatloaf                        |        90 | [267.0, 30.0, 12.0, 12.0, 29.0, 48.0, 2.0]    |        17 |              13 |                5 |


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
for each recipe. It's the outcome I want to predict. Features and the target 
are defined using the code below. 

```py
nutrition_cols = ['calories', 'fat', 'sugar', 'sodium', 'protein', 'saturated_fat', 'carbs']
features = recipes[['minutes', 'n_steps', 'n_ingredients'] + nutrition_cols]
target = recipes['average_rating']

```

The metric I am going to use for my model is Root Mean Squared Error (RMSE). I am using RMSE because it 
measures the average prediction error in the same units as the target variable
(`average_rating`). making it interpretable. RMSE also penalizes large errors 
more than small ones, which is important when predicting user ratings because 
significant prediction errors (e.g., predicting 2.0 instead of 5.0) can lead to 
poor recommendations.

## Baseline Model 

The baseline model uses a linear regression algorithm with one quantitative 
feature (`minutes`) and one nominal feature (`tags`), which was encoded using 
one-hot encoding. While the model serves as a simple starting point, its 
performance is poor, with an RMSE of 0.6509 and an R² of -0.0480. This suggests 
that the current features and linear approach do not adequately explain the 
variance in `average_rating`. Future models will incorporate additional features 
and more complex algorithms to improve performance.

## Final Model

#### Features added:
- **Log-transformed Minutes** (`log_minutes`): This transformation helps 
normalize the distribution of the `minutes` feature, which is often skewed. 
It likely reduces the influence of extreme outliers and captures diminishing 
returns in cooking time on recipe ratings.
- **Protein-to-Calorie Ratio** (`protein_calorie_ratio'): This feature provides 
insight into the nutritional density of a recipe. It reflects the balance of 
protein relative to calories, which could influence how recipes are perceived 
as healthy or satisfying.

These features were selected based on their potential relationship to user 
preferences and recipe quality:
1. `log_minutes` caputres the realistic diminishing impact of preparation time 
on user satisfaction.
2. `protein_calorie_ratio` highlights the importance of nutrition in predicting 
recipe ratings, as health-conscious users may prefer recipes with higher 
protein density.

#### Modeling Algorithm and Hyperparamters

- **Model Algorithm:** Lasso regression was chosen because it performs feature 
selection by shrinking coefficients of less important features to zero. This is 
especially useful given the large number of features in the dataset.
- **Hyperparamter Tuning:**
  - **Best Hyperparamter** (`alpha`): he optimal value was determined to be 
  `0.1` using a grid search with cross-validation.
  - **Tuning Method:** A grid search with five-fold cross-validation was 
  conducted to identify the best value for the `alpha` parameter, balancing 
  model complexity and performance.

The comparison to the Baseline Model is shown below:

| Metric     |   Baseline |  Final |
|:-----------|------------|-------:|
| RMSE   |         0.6509 | 0.6355 |   
| R^2    |         -0.0480| 0.0010 |

The final model slightly improved the RMSE, indicating a better fit to the 
data. The value R^2, while still low, improved to a positive value, suggesting 
a marginal gain in the model's explanatory power.

I visualization (scatter plot) omparing the predicted vs. actual ratings for \
the test set is shown below.

![alt text](images/final-model-visualization.png)

## Fairness Analysis

#### Group and metric definition

My **Group X** is recipes with fewer than the median number of ingredients, and 
my **Group Y** is recipes with the median number or more ingredients. My 
evaluation metric is **RMSE**, which measures the model's prediction error.

#### Hypotheses

- **Null Hypothesis** (H~0): The model performs equally well for both groups, 
and any observed differences in RMSE are due to random chance.
- **Alternate Hypothesis** (H~1): The model performs worse for Group X 
(higher RMSE) compared to Group Y.

The test statistic is the **difference in RMSE** between Group X and Group Y. 
The test was performed at the α = 0.05 significance level.

The results were

| Metric              |   Value |
|:--------------------|--------:|
| RMSE~X              |  0.6422 |
| RMSE~Y              |  0.6297 |
| Observed Difference |  0.0124 |
| P-value             |  0.1064 |

#### Conclusion

At the significance level of 0.05, the p-value of 0.1064 indicates that the 
observed difference in RMSE between Group X and Group Y is not statistically 
significant. Thus, we fail to reject the null hypothesis and conclude that 
there is no strong evidence of unfairness in the model's performance for these 
two groups.

Visualization for the permutation test:

![alt text](images/fairness-visualization.png)