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
  width="800"
  height="600"
  frameborder="0"
></iframe>

Then proceeded to analyze pairs of columns to check for positive associations.
An example of a pair of columns that showed a weaker association is shown below.

<iframe
  src="assets/log-minutes-plotly.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

I then created a pivot table. I used the `minutes` column and pivotted by the
`n_ingredients' column. I filtered out data entries that listed over 10000 
minutes to make as these were not "real" recipes and were posted in the data as
a sort of joke, and entries that had over 15 ingredients since a vast majority 
of the recipes required 15 or less ingredients. The resulting pivot table is 
shown below.

![alt text](images/pivot-table.png)