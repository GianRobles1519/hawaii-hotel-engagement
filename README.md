# Do Hawaiian Hotels That Respond to Reviews Have Higher Review/Rates

This is a final project for dsc80
By Gian Carlo Robles

## 1. Introduction

Hawaii has a huge tourism industry, and hotels play a big role in shaping how visitors feel about their trip. Most people these days look at Google Maps reviews before booking a hotel, so what shows up online really matters. A lot of business advice tells hotel owners they should respond to reviews, but that takes time and money — so I wanted to see if it actually pays off in the data. The question I'm trying to answer is:

> **Among hotels in Hawaii, does responding to customer reviews actually correspond to higher ratings?**

This is something I think real hotel owners would want to know, especially smaller ones who can't afford a dedicated team just to reply to every review. If responding really does help, then it's worth the effort. If not, they can spend that time somewhere else.

I'm using a dataset of Google Maps reviews for businesses in Hawaii from [this paper](https://aclanthology.org/2022.acl-long.426.pdf). After merging the review file with the business file, the full dataset has **1,505,011 rows** covering every kind of business in the state — restaurants, gas stations, beaches, everything. Since my question is specifically about hotels, I filtered down to only lodging-type places (Hotel, Resort hotel, Motel, Inn, Hostel, Bed & Breakfast, etc.). That gave me a working dataset of **68,996 reviews** across **329 hotels**.

**Columns I'm using:**

| Column            | What it is                                                  |
| ----------------- | ----------------------------------------------------------- |
| `gmap_id`         | Unique ID for each business — I used this to merge the two files |
| `Review_Rating`   | The star rating from the customer (1–5)                     |
| `avg_rating`      | The hotel's overall average rating on Google               |
| `Review_Time`     | When the customer left the review                           |
| `Resp_Time`       | When the business responded (if they did)                   |
| `Respond Comment` | The actual text of the business's response                  |
| `category`        | What kind of business it is — I used this to filter to hotels |
| `num_of_reviews`  | How many total reviews the hotel has                        |


## 2. Data Cleaning and Exploratory Data Analysis

The first thing I did was load both JSON files (the reviews and the business metadata) and merge them on `gmap_id`. I used an inner merge so I'd only end up with reviews that actually had business info, and vice versa — no orphan rows.

After the merge there were duplicate column names like `name_x` and `name_y` from both DataFrames, so I renamed them to `Business_Name`, `Reviewer_Name`, `Review_Time`, and `Review_Rating` so it'd be easier to keep track of what was what.

Next I filtered down to only hotels. The `category` column is a list of tags per business (a hotel might be tagged as `["Resort hotel", "Lodging", "Wedding venue"]`), so I exploded the column to get all the unique tags, picked the ones that looked like accommodations (Hotel, Resort hotel, Motel, Inn, Extended stay hotel, Hostel, Bed & breakfast, Lodge, Lodging, Boarding house, Travellers lodge), and kept any business whose category list contained at least one of those. This filter alone dropped the dataset from about 1.5 million rows down to 68,996 — Hawaii has way more restaurants and other small businesses than hotels.

Then I cleaned up the time columns. `Review_Time` came as a Unix timestamp in milliseconds, so I converted it to a real datetime. The `resp` column was a nested dictionary with a time and a text field for the business's response (or `NaN` if there was no response). I pulled both pieces out into two new columns: `Resp_Time` and `Respond Comment`. I made sure to leave the `NaN` values alone since whether a business responded or not is literally the thing I'm trying to study — replacing them with 0 or anything else would destroy that signal.

Finally I dropped a bunch of columns I wasn't going to use (`pics`, `url`, `relative_results`, `description`, `hours`, `MISC`, `resp`) just to keep the DataFrame readable.

Here's the head of the cleaned DataFrame (showing only the columns relevant to my analysis):

| Business_Name  | Review_Time         | Review_Rating | avg_rating | num_of_reviews | Resp_Time | Respond Comment |
| -------------- | ------------------- | ------------- | ---------- | -------------- | --------- | --------------- |
| Hickam Lodging | 2021-01-26 20:34:00 | 5             | 3.8        | 48             | NaT       | NaN             |
| Hickam Lodging | 2021-03-22 02:08:42 | 5             | 3.8        | 48             | NaT       | NaN             |
| Hickam Lodging | 2020-02-15 21:57:11 | 4             | 3.8        | 48             | NaT       | NaN             |
| Hickam Lodging | 2019-09-07 20:20:29 | 4             | 3.8        | 48             | NaT       | NaN             |
| Hickam Lodging | 2019-10-15 06:02:18 | 4             | 3.8        | 48             | NaT       | NaN             |

<iframe src="response_rate_bar.html" width="800" height="600" frameborder="0"></iframe>

The chart above shows how often hotels respond to reviews. About 26% of reviews get a response — meaningful, but most reviews go unanswered, which is exactly the engagement gap I'm interested in.

<iframe src="response_delay.html" width="800" height="600" frameborder="0"></iframe>

This one shows how quickly businesses respond when they do. Most responses come within a few weeks, with a long tail of much later responses.

<iframe src="response_vs_rating_scatter.html" width="800" height="600" frameborder="0"></iframe>

Each point is a hotel — x-axis is its response rate, y-axis is its average rating. There's a slight upward trend, suggesting hotels that respond more tend to have higher ratings.

<iframe src="rating_by_responded_box.html" width="800" height="600" frameborder="0"></iframe>

This box plot compares the rating distribution of reviews that got a response versus reviews that didn't. Reviews with a response tend to skew slightly lower — likely because businesses prioritize replying to negative feedback to manage their reputation.

| primary_category     | mean_rating | response_rate | review_count |
| -------------------- | ----------- | ------------- | ------------ |
| Hotel                | 4.35        | 0.23          | 45,640       |
| Resort hotel         | 4.46        | 0.32          | 19,000       |
| Lodge                | 4.23        | 0.30          | 833          |
| Condominium complex  | 4.56        | 0.11          | 784          |
| Golf course          | 4.42        | 0.55          | 645          |
| Hostel               | 4.13        | 0.14          | 621          |
| Bed & breakfast      | 4.46        | 0.44          | 376          |
| Indoor lodging       | 4.36        | 0.09          | 321          |
| Lodging              | 4.32        | 0.00          | 168          |
| Inn                  | 4.38        | 0.40          | 144          |

Looking at this, most of the reviews are for regular Hotels and Resort hotels. Resort hotels actually have a higher mean rating (4.46) and higher response rate (32%) than regular hotels (4.35 / 23%), so the bigger more expensive places tend to engage more with their reviews — which sort of makes sense since they have the staff and brand reasons to do it. Bed & breakfasts also stand out with one of the highest response rates (44%) since they're usually smaller and more personal places.


## 3.Assessment of Missingness

I think the `text` column (the actual written review left by the customer) in this dataset is likely **NMAR**. Whether or not someone writes text along with their star rating depends on how strongly they feel about their stay — and that's exactly the information the missing text would have contained. People who feel meh about a hotel might just leave a 3 or 4 star rating and skip writing anything, while people who felt really strongly (good or bad) are more likely to actually type out a review. So the missingness depends on what the text would have said if it were there — that's the definition of NMAR. If I had additional data like how long the reviewer spent on the review page, how many reviews they've left in the past in general, or whether they were prompted by Google to write text, I could probably explain the missingness through observable behavior and reclassify it as MAR.

For my permutation tests I looked at the missingness of the `resp` column (whether the business responded), since that's the column central to my whole question — and about 74% of reviews don't have a response.

**Test 1: Does `resp` missingness depend on `Review_Rating`?**

- **Null:** The distribution of review ratings is the same whether the business responded or not.
- **Alternative:** The distributions are different.
- **Test statistic:** Difference in mean rating between reviews with a response missing vs. not missing.
- **Method:** Permutation test (1,000 iterations).
- **Observed difference:** ~0.036
- **p-value:** 0.000

At a significance level of 0.05, I reject the null hypothesis. Missingness of `resp` does depend on the review's star rating. Interestingly, reviews *without* a response actually have slightly *higher* ratings on average, which means businesses are more likely to respond to lower-rated reviews — probably because they want to manage their reputation when something bad gets posted. This makes the `resp` missingness **MAR** with respect to `Review_Rating`.

<iframe src="missingness_rating.html" width="800" height="600" frameborder="0"></iframe>

The plot above shows the difference clearly — reviews with no response skew slightly higher in rating compared to reviews where the business did respond.

**Test 2: Does `resp` missingness depend on the day of the week the review was posted?**

- **Null:** The day-of-week distribution is the same whether the response is missing or not.
- **Alternative:** The distributions are different.
- **Test statistic:** Total Variation Distance (TVD) between the two day-of-week distributions.
- **Method:** Permutation test (1,000 iterations).
- **Observed TVD:** ~0.012
- **p-value:** 0.114

At a significance level of 0.05, I fail to reject the null hypothesis. There's no significant relationship between the day a review was posted and whether the business responded to it. That makes sense — there's no real reason a hotel would systematically skip responding to reviews posted on, say, Wednesdays vs. Saturdays.




## 4. Hypothesis Testing

To follow up on the missingness analysis, I wanted to actually test whether hotels that engage with reviews tend to have higher ratings than hotels that don't. To do this I aggregated my data up to the hotel level — one row per `gmap_id` — with each hotel's `response_rate` (the proportion of its reviews it replied to) and its `mean_rating`. Then I labeled each hotel as a "responder" if its response rate was at or above 50%, and a "non-responder" otherwise. That gave me 56 responder hotels and 273 non-responder hotels.

- **Null Hypothesis:** Among Hawaiian hotels, the average rating of hotels that respond to customer reviews is the same as the average rating of hotels that don't. Any observed difference is due to random chance.
- **Alternative Hypothesis:** Hawaiian hotels that respond to customer reviews have a higher average rating than hotels that don't.
- **Test Statistic:** Difference in mean rating between responder hotels and non-responder hotels (responders − non-responders).
- **Significance Level:** 0.05
- **Method:** Permutation test (1,000 iterations).
- **Observed Difference:** ~0.086
- **p-value:** 0.04

At a significance level of 0.05, I reject the null hypothesis. The p-value of 0.04 suggests that the observed difference in mean ratings (about 0.086 stars) is unlikely to have happened by chance alone. This gives evidence that Hawaiian hotels that actually respond to customer reviews tend to have higher average ratings than hotels that don't.

That said, this is a correlational result — not a causal one. It doesn't *prove* that responding to reviews makes ratings go up. It's possible that hotels which respond a lot are also just better-managed in general, so they'd already have higher ratings regardless of whether they engaged with reviews or not.



## 5. Framing a Prediction Problem

For my prediction problem I'm predicting a hotel's average rating (`avg_rating`), which makes this a regression problem since the target is a continuous value between 1.0 and 5.0. All of my features are aggregated to the hotel level and describe how the hotel behaves — not its outcomes — so I avoid data leakage. The features I'm using are the hotel's response rate (how often it replies to reviews), its average response delay (how long it takes to reply), the average length of its responses, the total number of reviews it has, and its primary accommodation category (Hotel, Resort hotel, Motel, etc.). All of this information would be known at the time of prediction since it's based on how the hotel operates, not on the ratings themselves.

For my evaluation metric I'm using **RMSE** (Root Mean Squared Error). I chose it because my target is continuous and on a small scale (1.0 to 5.0), so I wanted a metric that penalizes large prediction errors more than small ones and stays in the same units as the original ratings. RMSE is intuitive to interpret — an RMSE of 0.34 means my model is off by about 0.34 stars on average. I considered using R² but it doesn't have units and felt less directly meaningful for someone trying to interpret how good a rating prediction actually is.



## 6. Baseline Model

My baseline model is a **Linear Regression** that predicts a hotel's `avg_rating` using two features:

- `response_rate` (quantitative): the proportion of a hotel's reviews that it responded to.
- `total_reviews` (quantitative): the total number of reviews the hotel has on Google.

Both features are numerical, so no encoding was needed. I implemented everything in a single sklearn `Pipeline` and used an 80/20 train/test split with `random_state=42` so I could compare future models on the exact same data.

Originally I planned to use `price` (the price tier `$`, `$$`, etc.) as one of my baseline features instead of `total_reviews`. But when I actually checked the column, every single one of my 68,996 hotel rows had `None` for price. Hotels in this dataset just don't have a price tier populated — Google Maps tracks price tiers way more consistently for restaurants and bars than for accommodations. So I swapped it out for `total_reviews`, which is something every hotel in the dataset actually has.

My baseline model's performance:

| Metric     | Value   |
| ---------- | ------- |
| Train RMSE | 0.3702  |
| Test RMSE  | 0.3440  |

A test RMSE of about 0.34 means the model is off by roughly a third of a star on average for predicting `avg_rating`. For a baseline that only uses two features, this is a reasonable starting point — the model isn't overfitting (test RMSE is actually slightly *lower* than train RMSE, which usually means good generalization), but there's still plenty of room to improve. I wouldn't say this baseline is "good" in any final sense — it's missing a lot of the engagement-specific features (response delay, response text length, accommodation type) that I think will actually matter for predicting ratings. That's what I'll address in the final model.


