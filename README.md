### Quick Bites vs. Slow Feasts
**An exploration of Food.com recipe data.**  
Authored by: Kyle Duong

### Introduction

This project uses a large open dataset of recipes and ratings derived from Food.com, a website where home cooks share recipes and review each other's dishes that are shared online. The data comes in from two different CSV files, one with recipe details and the other that contains user interactions. The recipe file contains 83,782 rows (one row per recipe) and interaction file contains 731,927 rows (one row per review).

When people cook at time, they're always trading off between **time** and everything else such as taste, effort, etc. With busy schedules, a natural question comes up:
    **If I choose a "quick bite" (≤ 30 minutes) instead of a "slow feast" (> 30 minutes), am I actually giving up rating?** That's the question we want to explore!
    In other words, **Do “quick” recipes (≤ 30 minutes) get different average ratings than “long” (> 30 minutes) recipes?** 

This is an important question to investigate given it reflects a decision many people make everyday. Some days, there is not just enough time to make a hearty meal that takes multiple hours. They either decide to make something fast or spend the day in the kitchen, making a nice meal. By looking at real recipe data and ratings from thousands of real people, we can get a better sense of whether quick recipes are actually "worse," or whether you can save time without give up too much in terms of how much people enjoy the dish.

To further explore this question, we merge the recipe and rating datasets using a left join merge on Recipe ID and also calcualted the average rating per recipe, which was added to the original recipes DataFrame. The official DataFrame will contain 83,782 recipes.

We want to focus on the following columns, which are most relevant to our main question:
- **'minutes'**: The total preparation time for the recipe, formatted in minutes. This will be used to define **quick** recipes and **slow** recipes.
- **'avg_rating'**: The average rating (from 1 to 5) that a recipe received across all of the recipe's reviews.
- **'calories'**: The number of calories per serving.
- **'n_steps'**: The number of steps in the recipe instructions.
- **n_ingredients**: The number of distinct ingredients used in the recipe.
- **tags**: A set of user-supplied tags