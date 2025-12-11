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
- **'minutes'**: The total preparation time for the recipe, formatted in minutes. This will be used to define **quick** recipes and **slow** recipes later.
- **'avg_rating'**: The average rating (from 1 to 5) that a recipe received across all of the recipe's reviews.
- **'calories'**: The number of calories per serving.
- **'n_steps'**: The number of steps in the recipe instructions.
- **n_ingredients**: The number of distinct ingredients used in the recipe.
- **tags**: A set of user-supplied tags.
- **time_category**: A self-created column that groups the column **minutes** into two groups. If the recipe is less than or equal to 30 minutes, it will be grouped as quick otherwise it'll be grouped as long.

### Data Cleaning and Exploratory Data Analysis

To ensure that our analysis reflects how the Food.com data were generated, we performed several data cleaning steps before doing any visualizations. Below, we describe each major step, and why it makes sense given how the data were affected, as well as how it will affect our project moving forward.

**1. Merging recipes and interactions & fixing invalid ratings**

Food.com stores recipe information and user interactions in two separate tables. In the real world, a recipe is created first, and then zero or more users may later visit the recipe, try it, then come back and leave ratings. To respect this strcuture, we merged the two datasets using a left join on the recipe ID. This keeps every recipe from the recipe table even it never received any reviews from any users. This gives us a long table where each recipe could appear multiple times, once per interaction. When we inspected the rating column, I noticed that some of the rows had a rating of 0, even though Food.com only allows ratings
from one to five. A rating of 0 corresponds to a review where the user submitted a comment, but actually didn't rate the recipe at all. This what we refer to as "missing data," rather than a recipe being rated "zero." To avoid pulling down the average rating for these recipes, I replaced all rating values of 0 with NaN.
After that, we grouped by recipe and computed the mean of the (now cleaned) rating values, resulting in a new column called **avg_rating**, which we stored back in our DataFrame! 

This step is important to us because it created a recipe-level dataset that matches the real process, recipes are sent to the website first than ratings come after it. This also ensures that average rating are not lowered by the "zeros". It also gave us the avg_rating column, which is central to our main question and an
important column that we will be using for the majority of this analysis.

---

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

---

**3. Trimming extreme preparation times (in minutes)**

This one is an important one. The minutes column records the preparation time for each recipe. This is user-generated content, some entries are unrealistically large or represent very rare and extremely long recipes. This project is mainly heavily focused on preparation time, so I decided out of consensus to trim these extreme values, so they are not a factor later in the analysis. These extreme values heavily skew the distribution and can dominate summary statistics and visualizations in a way that doesn't reflect typical home cooking. For example, 

| index  | name                      | id     | minutes | ... | contributor_id | sodium | protein | saturated_fat | carbohydrates |
|--------|---------------------------|--------|---------|-----|----------------|--------|---------|---------------|----------------|
| 109931 | how to preserve a husband | 447963 | 1051200 | ... | 576273         | 1.0    | 7.0     | 115.0         | 5.0            |

Sourced: https://www.food.com/recipe/how-to-preserve-a-husband-447963

These types of recipes aren't even recipes. This recipe takes over one million minutes to make, which is approximately two years. To address this, I computed the 99th percentile of minutes and defined cleaned_recipe_reviews, a new DataFrame where it keeps recipes with mintues less than or equal to the cutoff of the 99th percentile. In other words, to address this situation, I removed the top 1% longest recipes in terms of reported preparation time. The bulk of the project is focused on everyday quick and slow recipes, not the longest dishes that we've ever seen. Removing the top 1% of of mintues helped us focus on patterns that are most relevant, as well as being able to keep a very large number of recipes.

---

**4. Creating a new column: time_category**

Our main question is about quick bites versus slow feasts, so I decided to create a new categorical column called time_category that labels each recipe as either quick or long. Based on the project description, we defined quick recipes as those with minutes ≤ 30 and long recipes as those with minutes > 30. This was done by applying a simple condition on the cleaned minutes column and assigning the labels "quick" and "long" for the amount of minutes it was.

This step is important because it turned raw preparation time into a clear group label that we will use throughout the rest of the project. The time_category column will allow us to directly compare the distribution of ratings, calories, and complexity between quick and long recipes. It will be used later for further analysis on the question that we are asking.

---

**Final Cleaned Data Preview**

Shown below is a preview of the cleaned dataset (cleaned_recipe_reviews.head()), which includes all the cleaning that we had done:

| name                               | id     | minutes | avg_rating | ... | n_steps | n_ingredients | tags                                                           | time_category |
|------------------------------------|--------|---------|-----------:|-----|--------:|--------------:|----------------------------------------------------------------|---------------|
| 1 brownies in the world best ever  | 333281 | 40      | 4.0        | ... | 10      | 9             | ['60-minutes-or-less', 'time-to-make', 'course', ...]         | long          |
| 1 in canada chocolate chip cookies | 453467 | 45      | 5.0        | ... | 12      | 11            | ['60-minutes-or-less', 'time-to-make', 'cuisine', ...]        | long          |
| 412 broccoli casserole             | 306168 | 40      | 5.0        | ... | 6       | 9             | ['60-minutes-or-less', 'time-to-make', 'course', ...]         | long          |
| 412 broccoli casserole             | 306168 | 40      | 5.0        | ... | 6       | 9             | ['60-minutes-or-less', 'time-to-make', 'course', ...]         | long          |
| 412 broccoli casserole             | 306168 | 40      | 5.0        | ... | 6       | 9             | ['60-minutes-or-less', 'time-to-make', 'course', ...]         | long          |

---

**Univariate Analysis: Number of Steps**

To further analyze our question, I decided to examine the number of steps in a recipe. In this graph, the distribution is right-skewed, where most recipes have roughly five to fifteen steps, with the count dropping off as the number of steps increases in the recipe. There are a small number of recipes that have 30+ steps, which represent dishes that take a long time. This graph is useful because it showcases the complexity of the recipes, as the more steps it has, the more complex it is as well as the more time it takes to make that certain dish as well. 

<iframe
  src="assets/steps_hist.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

---

**Univariate Analysis: Distribution of Average Recipe Ratings**

Another useful plot is to show how average recipe ratings are being distributed across all recipes in the dataset. As we can see, most recipes usually have very high ratings ranging from 4.0 to 5.0 with a big boom nears 5 stars. There are relatively very few recipes that are rated below three stars. This tells us that users tend to rate recipes positively, so any difference between "quick" and "long" recipe average ratings will likely be very small, rather than us noticng a big contrast between good and bad recipes. 

<iframe
  src="assets/avg_rec_ratings.html"
  width="600"
  height="450"
  frameborder="0"
></iframe>

--- 

**Bivariate Analysis: Average Ratings by Time Category**

Another useful plot is to compare the average ratings between the "quick" category and the "long" category. To do this, we plotted a boxplot because it will compare the distribution of average ratings between two groups in a compact way. This boxplot will summarize key distributional differences cleanly. This will tell us more information about the question we are up against. As we can see, both categories are rated highly overall and it looks to be that their medians and spreads are almost identical together. As suggested in the Univariate Analysis: **Distribution of Average Recipe Ratings**, it seems that recipe do not tend to be reated differently if they are quick or long, it might be a very subtle difference if we do see any kind of it. We will learn more when we do our hypothesis testing!

<iframe
  src="assets/boxplt_avg_ratings_quick_long.html"
  width="600"
  height="450"
  frameborder="0"
></iframe>

---

**Bivariate Analysis: Preparation Time vs. Average Rating**

This scatterplot shows the relationship between preparation time and average rating, with polors colored by whether the recipe is quick or long. As we can see, quick recipes cluster at shorter preparation times while long recipes span a much wider time range. However, as we can see, both groups are still mostly rated between four and five stars, which backs up what we had seen in our previous graph. There is no strong trend of ratings increasing or decreasing with cook time, which shows that long recipes are not neccessarily rather much higher or lower than quick ones. As what we been seeing, the difference may be subtle if there is anything at all.

<iframe
  src="assets/preptime_vs_avgrating.html"
  width="600"
  height="450"
  frameborder="0"
></iframe>

---

Interesting Aggregates: Rating Summary for Quick vs. Long Recipes

To further explore how cooking time relates to user satisfication, we aggregated our data by the **time_category** column and created a pivot table that summarizes for each group of recipes:

- **Mean Rating**: The average **avg_rating** across the long and quick categories.
- **Median Rating**: The middle **avg_rating**, which is less sensitive to extreme values.
- **Number of Recipes**: How many recipes fall into each time category.

The aggregate table compares ratings for quick and long recipes. Both categories have very high ratings overall, with mean ratings of 4.66 (long) and 4.69 (quick), while having median ratings of 4.86 and 4,88, so the difference in tendency is very small. There is also a similar number of recipes in each group, which means that our later hypothesis test comparing quick vs long is very balanced because there isn’t that much of a difference between the two group sizes. This table suggests that if there is a difference in ratings between quick and long recipes that it is likely to be very subtle.

| Time Category   |   Mean Rating |   Median Rating |   Number of Recipes |
|:----------------|--------------:|----------------:|--------------------:|
| long            |       4.66022 |         4.85714 |              122797 |
| quick           |       4.69495 |         4.88    |              106587 |

---

Interesting Aggregates: Rating by Number of Steps

To explore the relationship between recipe complexity and the ratings, we aggregated our data by the n_steps column and created a pivot table that summarizes for each distinct number of steps.

- **Mean Rating**: The average **avg_rating** for recipes with that many steps.
- **Median Rating**: The middle **avg_rating**.
- **Number of Recipes**: How many recipes have exactly that number of steps.

This pivot table summarizes how recipe ratings vary with the number of steps, which is showcases how long a recipe is to make. Most recipes fall in the mid-range of steps, and across almost all steps, the mean ratings stay around 4.6 and 4.7 with a median of 5. In other words, recipes with only a few steps and many steps are both rated very highly. Recipes with more steps are generally more time-consuming, so this pattern supports our main findings that any differences between quick and long recipes likely be very subtle.

| Number of Steps | Mean Rating | Median Rating | Number of Recipes |
|----------------:|------------:|--------------:|------------------:|
|               1 |     4.71742 |        5.0000 |              2653 |
|               2 |     4.72083 |        4.9231 |              7536 |
|               3 |     4.70468 |        4.9000 |             11551 |
|               4 |     4.69776 |        4.8750 |             14663 |
|               5 |     4.65825 |        4.8333 |             17448 |
|               6 |     4.66666 |        4.8333 |             18538 |
|               7 |     4.67944 |        4.8571 |             19741 |
|               8 |     4.66903 |        4.85246|             19062 |
|               9 |     4.66500 |        4.84615|             18098 |
|              10 |     4.65200 |        4.8333 |             15947 |
|              11 |     4.66880 |        4.86667|             13402 |
|              12 |     4.66565 |        4.8333 |             11868 |
|              13 |     4.68586 |        4.8750 |              9712 |
|              14 |     4.66882 |        4.8750 |              8276 |
|              15 |     4.67000 |        4.88776|              6839 |
| ...            |      ...     |        ...    |               ... |
|              87 |     5.00000 |        5.0000 |                10 |
|              88 |     3.00000 |        3.0000 |                 1 |
|              93 |     5.00000 |        5.0000 |                 4 |

---

### Assessment of Missingness

**NMAR Analysis**:

Using the flowchart for missingness, we start with Missing by Design (MD). For the **review** column, the missing values are not MD. This is because Food.com doesn't automatically require or generate review text based on other columns and nothing about minutes, avg_rating, or n_steps lets us reconstruct exactly what this msising review would have said. Uers are simply given the option to leave a review or not, so we move onto NMAR. We ask whether the missingness of **review** is based on NMAR. Here, I think the answer is yes! In practice, people are more likely to type out a review only when they have a strong reaction to a recipe, either that they loved it or disliked it, had an amazing time with it, or felt that it was important to write something about it. If there is experience was mediorce, or they just bake and never leave reviews, or in a rush, they may just click a star rating and skip out writing the review. Those underlying reasons are not recorded in our dataset, but they directly affect whether **review** is missing or not. This is exactly the definition of NMAR because the probability that a value is missing depends on unobserved information, in this case, what the people would have written or how strongly they felt. To make this missingness closer to MCAR, we need extra observed variables that capture these hidden factors. For example, Food.com could log a simple yes/no response such as "Did you enjoy this recipe?" or a short reason for not reviewing option. If we had columns such as overall enjoyment or review frequency, then we found that missing reviews were largely explained by those variables, then we could say review missingness is MAR with respect to them. 

---

**Missingness Dependency**:

Note this analysis for this part, I will include all information that was included in the DataFrame. Earlier, we removed outliers because our questions were mostly focused on preparation timee. However, since we are testing for missingness, we will use all originally observed rows (including the outliers) because it is important when studying for missingness. 

To study missingness more, we focused on the column **rating** and created a boolean indicator **rating_missing** that is True when a recipe has no average rating and **False** otherwise. Following the missingness flowchart, once we ruled out "MD" and "NMAR" for **rating**, we used permutation tests whether its missingness was MCAR and MAR with respect to other columns.

Permutation tests will be conducted whether the missingness of the column **rating** is dependent on other variables.

Let's start with if the column of **rating** is dependent on **minutes**.

Before running our permutation test, we have to first set it up!

* Null Hypothesis: The distribution of **minutes** is the same for recipes with missing ratings and recipes with non-missing ratings.
* Alternative Hypothesis: The distribution of **minutes** is different for recipes with missing ratings compared to those with non-missing ratings.
* Test Statistic: The differnece in mean preparation time between recipes with missing ratings and recipes with non-missing ratings.

In other words:

*T* = (mean minutes for rows with missing ratings) - (mean minutes for rows with non-missing ratings)

Significance: (α = 0.05)
* If the p-value < 0.05: I will reject the null hypothesis and conclude that rating missingness is related to the preparation time.
* If the p-value ≥ 0.05: I will fail to reject hypothesis and conclude that any observed difference in minutes is consistent with random chance.

Let's graph it before we start our permutation test.

This KDE plot is comparing the distribution of prep time in minutes for two groups:
- True (Blue): Rows where rating_missing == True
- False (Orange): Rows where rating_missing == False

Both curves are strongly right-skewed. The two lines mostly overlap, which means the distributon of preparation times for recipes with missing ratings looks very similar to the distribution for recipes that have recorded ratings. Visually, there isn't a huge shift in the center or the shape between the groups, so from this plot alone we don't have strong evidence that longer (or shorter) recipes are much more likely to be missing ratings.

<iframe
  src="assets/distofmins.html"
  width="600"
  height="450"
  frameborder="0"
></iframe>

To test whether missing ratings depend on preparation time, we compare the average minutes for recipes with missing ratings to those that aren't missing. The test statistic is the difference in mean minutes (missing - not missing). Under the null hypothesis that the rating missingness is independent of preparation time, shufflign the missing/not missing labels across recipes shouldn't systemically change this difference. By repeatedly permuting the labels and recomputing the difference in means, the function builds an empirical null distribuition and uses it to compute a p-value, telling us how unsuual our observed difference is under the null.

After running our permutation test, let's build an empirical distribution to get sense of our data:

<iframe
  src="assets/empdismeandiffinmin.html"
  width="600"
  height="450"
  frameborder="0"
></iframe>

This empirical distribution shows the null distribution of our test statistic, which is the difference in mean minutes (missing - present). The blue bars represent how likely different difference in means are if, in reality, there is no true difference between the two groups. The red vertical line marks our observed difference of about 51.45 minutes from the original and unshuffled data. To find the p-value, we look at the proportion of the blue bars at or to the right of this red line, which was 0.13, which means 13% of simualted differences were as larger as or larger than what we had observed, so our result is not extremely unusual under the null. In our permutation test, the observed difference in mean preparation time (51.45 minutes) produced a p-value of roughly 0.13, which is larger than our chosen significance level of 0.05. This means our data do not provide strong statistical evidence against the null hypothesis. This means that we **fail to reject the null** hypothesis. In other words, while recipes with missing ratings appear to take longer on average, the difference is not statistically significant at the 5% level. This means that we can not confidently claim that rating missingness is associated with preparation time. With respect to **minutes**, the missingness of **rating** is consistent with MCAR, as any observed difference in mean prep time could reasonably be explained by random chance rather than a systemtic dependence on minutes. 

--- 

Now, let's try see if the column of **rating** is dependent on **calories**!

As before, we have to set it up!

* Null Hypothesis: The distribution of calories is the same for recipes with missing ratings and recipes with non-missing ratings.
* Alternative Hypothesis: The distribution of calories is different for recipes with missing ratings compared to those with non missing ratings.
* Test Statistic: The difference in mean calories between recipes with missing ratings and recipes with non-missing ratings.

In other words:

*T* = (mean calories for rows with missing ratings) - (mean calories for rows with non-missing ratings)

Significance: (α = 0.05)
* If the p-value < 0.05: I will reject the null hypothesis and conclude that our data provides evidence that rating missingness is associated with calories.
* If the p-value ≥ 0.05: I will fail to reject the null hypothesis and conclude that our data does not provide strong evidence that rating missingness depends on calories.

Again, let's visualize it first!

This KDE plot is comparing the distribution of calories for two groups:

* True (Blue): Rows where rating_missing == True.
* False (Orange): Rows where rating_missing == False

This graph shows the KDE plot of calories for recipes with and without missing ratings, after filtering with fewer than 10,000 calories to get a closer look. The orange curve represents recipes where rating is not missing, and the blue curve represents recipes with missing ratings. Both curves are extremely right-skewed, as most recipes have relatively low calories, and a long tail that stretches out to high values. The two lines lie almost on top of each other across the whole range.

<iframe
  src="assets/distofcalories.html"
  width="600"
  height="450"
  frameborder="0"
></iframe>

We ran a permutation test to compare the mean of a numeric column between rows where a value is missing or not missing. It first drops rows where the numeric column is NaN, then computes the observed difference in means between the True and False groups of missing_col. To see what differences could occur by random chance, it shuffles the missing/not-missing labels, recomputes the difference in means for each shuffle, and then stores these values. The p-value is then calculated as the proportion of shuffled difference that are at least extreme.

Again, let's visualize it using a empirical distribution:

<iframe
  src="assets/empdistofcalories.html"
  width="600"
  height="450"
  frameborder="0"
></iframe>


This empirical distribution shows the permutation (null) distribution of the difference in mean calories between missing and non-missing ratings. The blue bars are the differences we get just by randomly shuffling the missing/non-missing labels, as they are mostly near 0. The red line is at about 69.01 calories, which is our observed difference, far out in the tail, meaning a difference this large is extremely unlikely under the null and gives strong evidence that missingness is related to calories. In our permutation test, the observed difference in mean calories between recipes with missing and non-missing ratings was about 69.01 (missing - not missing), and the p-value was essentially 0. This p-value is well below our signifcance level of 0.05, we **reject** the null hypothesis and conclude that our data provide strong evidence that rating missingness is associated with calories. In this sample, recipes with missing ratings tend to have higher average calories than those with non-missing ratings, suggesting that missingness is not completely random with respect to calories, although we can't fully determine the exact missingness mechanism from this test alone. 

---

### Hypothesis Testing 

To remind you about our question:

**Do “quick” recipes (≤ 30 minutes) get different average ratings than “long” (> 30 minutes) recipes?**  

As in Step 3, we worked with a DataFrame that included all of the rows, because we needed all the rows to study missingness. For this hypothesis test, however, return to our original dataset, where we have already removed unrealistic time outliers and added the columns **avg_rating** and **time_category**. This cleaned verisionbebtter reflects typical recipes and is more appropriate for comparing the average ratings of quick vs long recipes. It removes these fake recipes that take over one million minutes that may skew the ratings to one group, hurting the overall analysis becasue of it.

* Null Hypothesis: Quick recipes (≤ 30 minutes) and long recipes (> 30 minutes) have the same average rating.
* Alternative Hypothesis: Quick recipes and long recipes have different average ratings.
* Test Statistic: The difference in average rating between the two groups.

In other words:

*T* = (average rating for long recipes) - (average rating for quick recipes)

Significance: (α = 0.05)
* If the p-value < 0.05: I will reject the null hypothesis and conclude that quick and long recipes have different average ratings.
* If the p-value ≥ 0.05: I will fail to reject the null hypothesis and conclude that any observed difference in average rating could reasonably be due to chance. 

Results:
* Observed Statistic: -0.035
* p-value: 0.00

Let's visualize it using an empirical distribution!

This empirical distribution shows the null distribution of the difference in average mean recipe rating between long and quick recipes, built from the permutation test. The blue bars are simulated differences in mean rating (long - quick) you get when the permutation test shuffled the "quick/long" labels many times. They are centered near 0, which is what we'd expect if the two groups truly had the smae average rating. The red line at about -0.035 is our observed difference, meaning long recipes are rated about 0.035 stars lower on average than quick recipes. The red line lies far in the left tail of the null distribution and the p-value is zero, this difference is unlikely to occur by chance alone, so our data provides strong evidence that quick and long recipes do not have the same average rating.

<iframe
  src="assets/empdistofhypothesis.html"
  width="600"
  height="450"
  frameborder="0"
></iframe>

We used a two-sided alternative because our question was is a two-sided alternative because our question was "do they differ?" rather than assuming in advance that one group must be higher. To assess significance, we run a permutation test meaning that we shuffled the time_category labels (quick and long) among recipes, recomputed the difference in mean ratings each time. This is a good choice for our setting because ratings are discrete and skewed and a permutation test does not rely on normality assumptions, it directly reflects the kind of variation we'd expect just from random group labels. We used a significance level of 0.05. The resulting p-value was effectively 0. This p-value far below 0.05, meaning we **reject** the null hypothesis sand conclude that our data provide evidence that quick and long recipes do not have exactly the same average rating. However, the estimated difference in means is very small (about 0.035 stars), so while the result is statistically significant given our large sample size, the practical difference in suer ratings between "quick bites" and "slow feasts" appear to be quite subtle rather than very dramatic. 

### Framing a Prediction Problem

Our overall project has focused on whether “quick” recipes get different ratings than “long” recipes. To extend this theme, we now turn to a prediction problem! Let’s predict a recipe’s cooking time (in minutes). This type of problem is a regression problem since the quanity we want to predict is a continuous numeric value rather than predicting a category for which it belongs to. For our baseline model, we'll predict minutes using four features from our DataFrame! 

Let's start to make our baseline model!

- Our target variable (minutes): This is what we're trying to predict. 

Here are some features we will use at the time of the prediction:

We imagine our model being used when a recipe page already exists on Food.com. At that point, we know:
- n_steps and n_ingredients: This comes directly from recipe instructions and ingredient list
- calories: This is found within the nutrition information that is stored on that recipe
- tags: These are user-supplied labels describing the recipe
- avg_rating: This summarizes the reviews the reecipe has already received.

All of these are available before predicting a cooking time estimate. None of these are information from the future. 

Our evaluation metric: We will evaluate our models primarily on using the **root mean squared error** in minutes, and we will also report R² as a secondary summary. RMSE is a good choice because it is defined for continuous outcomes and penalizes large mistakes more heavily than small ones. It is measured in the same units as the target, so we can interpret it directly. For example, if there was an RMSE of 60, this would mean our predictions are off by about 60 minutes on average. RMSE is preferred over metrics like accuracy or F-1 score because they are suited for classification problems, while we are trying to predict the cooking time.  

--- 

### Baseline Model

For my baseline model, I used a **multiple linear regression** model to predict a recipe's preparation time from a small set of numeric features.

The model uses the following features:

- **n_steps** (Quantitative): number of steps in the recipe
- **n_ingredients** (Quantitative): number of distinct ingredients
- **avg_rating** (Quantitative): average user rating (1-5)
- **calories** (Quantitative): calories per serving

All four of these are quantitative features. I also did not include any ordinal or nominal features in the baseline model, so there was no need for any one-hot encoding or other categorical encodings. Before fitting the model, I also standardized all four numeric features using **StandardScaler** inside a ColumnTransformer, so that each feature has a mean of 0 and variance 1. We split the data into an 80% training set and a 20% test set in order to assess how well the model is able to generalize to unseen recipes. This helps the linear regression treat all features on a comparable scale but doesn't change the fundamental predictions. 

Here were the results for our multiple linear regression model:

| Split | R_squared | RMSE_minutes |
|-------|-----------|--------------|
| Train | 0.0448    | 80.27        |
| Test  | 0.0452    | 79.29        |

On the test set, we set using an 80/20 split, where the baseline linear regression model performs very poorly. Both the training and test R² values are close to 0. The RMSE on both splits is roughly 80 minutes. An RMSE of about 80 means, on average, the model predictions are off by more than 80 minutes in total, which is not very helpful for someone trying to estimate cooking time. The very low R² also shows how the model explains almost none of the variation in minutes beyond just predicting something close to the overall mean. Because of the combination of high error and low predictive power, I do not consider the baseline to be "good" but rather for it to be very "weak". This baseline was mainly serve as a starting point, as I look to improve the model.
 
--- 

### Final Model 

For my final model, the same prediction target was used as in the baseline model, which was minutes. This is because minutes is the overall item that we are trying to predict! However, there are new features that I haev added and also switched to a more flexible modeling algorithm as well!

The features that will be used in the final model:
- On top of the original numeric features (n_steps, n_ingredients, avg_rating, calories), I also engineered three additional features that will help predict preparation time.
- calories_per_ingredient: total calories divided by the number of ingredients
This feature is important because it gives a sense of how rich or dense each ingredient is on average. A dish with many calories packed into relatively few ingredients (butter, cream, cheese) might invovle different preparation steps (melting, baking, longer cooking) than a low-calorie salad. This connects to the data generating process because more complex and heavier recipes may correlate to more time.
- log_calories: a log transform of calories using log1p
Cooking times can grow with recipe "size," but the relationship is unlikely to be perfectly linear. Taking the log compresses very large calorie values and helps the model pick up a smoother and more realistic pattern without leeting a few huge-calorie recipes dominate the model.
- meal_type: a category inferred from the tags column, where there are four options: "breakfast", "lunch", "main-dish" (otherwise known as dinner), and "other".
Different meal types often have different typical prep times. For example, breakfast dishes have to be quick because people either have work, school, etc. while mai dishes are more likely to be slower. Using meal_type gives the model high-level context about when and how the recipe is intended to be served, which is related to how long people are willing to spend cooking it.

All numeric features are treated as quantitative, and meal_type is a nominal categorical that I one-hot encoded in the pipeline. Together, these features try to capture structural complexity (steps, ingredients) and recipe style (nutrition, meal type) which are all reasonable drivers of cooking time!

Modeling algorithm and hyperparameter selection:

For the final model, I pivoted towards a Random Forest Regressor, wrapped in a pipeline that will do the following threee steps:

1. Come feature engineering with FunctionTransformer to add calories_per_ingredient, log_calories, and meal_type.
2. Use a ColumnTransformer to: 
    - Standardize all numeric features with StandardScaler
    - One-hot encode meal_type
3. Fit a RandomForestRegressor on transformed measures 

Random forests is my chosen model because they can handle these types of relationships and interactions without us having to hand specify all those combinations as features. I, then, tuned the model's hyperparameters using GridSearchCV with 3-fold cross-validation on the training set, using negative RMSE as the scoring metric. This is because RMSE being lower means that it is better. The grid searched over:

- n_estimators: [100, 200]
- max_depth: [None, 10, 20]
- min_samples_leaf: [1, 5]

The best-performing combination found by GridSearchCV was:

- n_estimators: 200
- max_depth: None
- min_samples_leaf = 1

I then used this best model to fit on the full training data and evaluated it on the held-out test data, which is the same 80/20 split as in the baseline. Now, we can see the performance of our baseline model.

| Split | R_squared | RMSE_minutes |
|-------|-----------|--------------|
| Train | 0.966     | 15.06        |
| Test  | 0.758     | 39.89        |

For comparison, here was the earlier baseline model:

| Split | R_squared | RMSE_minutes |
|-------|-----------|--------------|
| Train | 0.0448    | 80.27        |
| Test  | 0.0452    | 79.29        |

Compared to the baseline:
- Test R² improved dramatically about 0.05 to 0.78, meaning the final model explains a much larger share of the variation in cooking time.
- Test RMSE improved from about 79 minutes to 40 minutes, so typical prediction errors were roughly cut in half.

There is a noticeable gap between training and test performance. By this, this means that there is a high train R² and a low test R², suggesting some degree of overfitting, which is common with flexible models like random forests. However, even with the gap, the final model generalizes far better than the baseline. This can be told because the errors are smaller and it captures more structure in the data. Overall, the engineered features and the Random Forest Regressor yield a more useful predictor of recipe preparation time than the original multiple linear regression baseline model. 

---

### Fairness Analysis 

For our fairness check, we asked whether our final model performs worse for "simple" recipes than for "complex" recipes. In other words, does the model make larger errors for one type of recipe?

To define our groups:
- Group X (Simple recipes): recipes with n_steps ≤ 7
- Group Y (Complex recipes): recipes with n_steps > 7

We chose n_steps because it's a natural measure of recipe complexity! They also measure the complexity of time because the more steps, the more time it takes. A fair model should not be much less accurate for one of these groups than the other. 

Let's define our permutation test now:

Hypotheses & Test Statistic:
* Null Hypothesis ($H_0$): There is no difference in RMSE performance between simple recipes and complex recipes, any observed difference is due to chance.
* Alternative Hypothesis ($H_1$): The model performs worse on complex recipes, meaning its RMSE for complex recipes is higher than its RMSE for simple recipes.
* Test Statistic: (The root mean squared error of complexity) - (the root mean squared error of simple)

In other words:

*T* = (RMSE of complex) - (RMSE of simple)

- Note: We use RMSE on the test set as our evaluation metric because RMSE directly measures how far our predicted cooking times are, in minutes. 

Significance: (α = 0.05)
* If the p-value < 0.05: There is evidence that the model performs worse on complex recipe.
* If the p-value ≥ 0.05: We do not have enough evidence to claim unfairness in that direction.

From the test set, we found: 
- RMSE for simple recipes to be 41.93 minutes
- RMSE for complex recipes to be 38.45 minutes

This gives us our observed test statistic of -3.48 minutes. After running our permutation test on 10000 iterations, we found the p-value to be 0.9923. Let's visualize this using a empirical distribution:

This shows the null distribution of our test statistic. Under the null hypothesis, the model is equally accurate for simple and complex recipes; these permuted differences cluster around 0, so most of the blue bars are near 0 on the x-axis. The red vertical line marks our observed difference of about -3.48 minutes, meaning that in the real data, the RMSE for complex recipes is about 3.5 minutes lower than for simple recipes. Almost the entire null distribution lies to the right of the red line; the one-sided p-value is very large. This indicates that our analysis does not provide evidence that the model performs worse on complex recipes. 

<iframe
  src="assets/empdistofrmsediff.html"
  width="600"
  height="450"
  frameborder="0"
></iframe>

The resulting one-sided p-value was about 0.9923. This is much larger than 0.05, we **fail to reject** the null hypothesis. In other words, our data does not provide evidence that the model performs worse on compelx recipes than on simple ones. In fact, the model may slightly be more accurate on complex recipes, though we would be catious on over-interpreting thos.

Overall, within this fairness setup, we do not fidn evidence that the final model is unfair against complex recipes.

--- 
 
## Fin 👨‍🍳