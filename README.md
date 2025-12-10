### Quick Bites vs. Slow Feasts
**An exploration of Food.com recipe data.**  
Authored by: Kyle Duong

### Introduction

This project uses a large open dataset of recipes and ratings derived from Food.com, a website where home cooks share recipes and review each other's dishes that are shared online. The data comes in from two different CSV files, one with recipe details and the other that contains user interactions. The recipe file contains 83,782 rows (one row per recipe) and interaction file contains 731,927 rows (one row per review).

When people cook at time, they're always trading off between **time** and everything else such as taste, effort, etc. With busy schedules, a natural question comes up:
    **If I choose to make "quick bite" (≤ 30 minutes) instead of a "slow feast" (> 30 minutes), am I actually giving up the quality?** That's the question we want to explore!
    In other words: **Do “quick” recipes (≤ 30 minutes) get different average ratings than “long” (> 30 minutes) recipes?** 

This is an important question to investigate given it reflects a decision many people make everyday. Some days, there is not just enough time to make a hearty meal that takes multiple hours. They either decide to make something fast or spend the day in the kitchen, making a nice meal. By looking at real recipe data and ratings from thousands of real people, we can get a better sense of whether quick recipes are actually "worse," or whether you can save time without give up too much in terms of how much people enjoy the dish.

To further explore this question, we merge the recipe and rating datasets using a left join merge on Recipe ID and also calcualted the average rating per recipe, which was added to the original recipes DataFrame. After the merge, the official DataFrame will contain 83,782 rows that contain the recipe as well as the average rating of it.

We want to focus on the following columns, which are most relevant to our main question:
- **'minutes'**: The total preparation time for the recipe, formatted in minutes. This will be used to define **quick** recipes and **slow** recipes.
- **'avg_rating'**: The average rating (from 1 to 5) that a recipe received across all of the recipe's reviews.
- **'calories'**: The number of calories per serving.
- **'n_steps'**: The number of steps in the recipe instructions.
- **n_ingredients**: The number of distinct ingredients used in the recipe.
- **tags**: A set of user-supplied tags.

### Data Cleaning and Exploratory Data Analysis

To ensure that our analysis reflects how the Food.com data were generated, we performed several data cleaning steps before doing any visualizations. Below, we describe each major step, and why it makes sense given how the data were affected, as well as how it will affect our project moving forward.

**1. Merging recipes and interactions & fixing invalid ratings**

Food.com stores recipe information and user interactions in two separate tables. In the real world, a recipe is created first, and then zero or more users may later visit the recipe, try it, then come back and leave ratings. To respect this strcuture, we merged the two datasets using a left join on the recipe ID. This keeps every recipe from the recipe table even it never received any reviews from any users. This gives us a long table where each recipe could appear multiple times, once per interaction. When we inspected the rating column, I noticed that some of the rows had a rating of 0, even though Food.com only allows ratings
from one to five. A rating of 0 corresponds to a review where the user submitted a comment, but actually didn't rate the recipe at all. This what we refer to as "missing data," rather than a recipe being rated "zero." To avoid pulling down the average rating for these recipes, I replaced all rating values of 0 with NaN.
After that, we grouped by recipe and computed the mean of the (now cleaned) rating values, resulting in a new column called **avg_rating**, which we stored back in our DataFrame! 

This step is important to us because it created a recipe-level dataset that matches the real process, recipes are sent to the website first than ratings come after it. This also ensures that average rating are not lowered by the "zeros". It also gave us the avg_rating column, which is central to our main question and an
important column that we will be using for the majority of this analysis.

**2. Splitting the nutrition column into separate numeric features**

In the raw recipes table, Food.com stores nutritional information as a single string in a column called nutrition. This means that it has values formatted as "[calories, total fat, sugar, etc.]" Each of these values is a separate numerical quanity. To make these usable, we stripped the square brackets off the nutrition string, split each string on commas into seven parts, cleaned extra whitespace, and converted all seven entries into numeric numbers. We then assigned them into meaningful column names.

The columns are the following:

- `calories`
- `total_fat`
- `sugar`
- `sodium`
- `protein`
- `saturated_fat`
- `carbohydrates`

These following columns were added to our DataFrame, while we dropped the original nutrition column because it was not needed anymore. By turning nutrition from a single string into separate numeric columns, we are able to use these columns directly if needed. In particular, the calories column will come in handy later on, as we further analyze this data!

**3. Trimming extreme preparation times (in minutes)**
