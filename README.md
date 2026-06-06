# Do Hawaiian Hotels That Respond to Reviews Have Higher Review/Rates

This is a final project for dsc80
By Gian Carlo Robles

## Introduction

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


## Data Cleaning and Exploratory Data Analysis

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



