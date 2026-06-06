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


| Business_Name  | Review_Time         | Review_Rating | avg_rating | num_of_reviews | Resp_Time | Respond Comment |
| -------------- | ------------------- | ------------- | ---------- | -------------- | --------- | --------------- |
| Hickam Lodging | 2021-01-26 20:34:00 | 5             | 3.8        | 48             | NaT       | NaN             |
| Hickam Lodging | 2021-03-22 02:08:42 | 5             | 3.8        | 48             | NaT       | NaN             |
| Hickam Lodging | 2020-02-15 21:57:11 | 4             | 3.8        | 48             | NaT       | NaN             |
| Hickam Lodging | 2019-09-07 20:20:29 | 4             | 3.8        | 48             | NaT       | NaN             |
| Hickam Lodging | 2019-10-15 06:02:18 | 4             | 3.8        | 48             | NaT       | NaN             |








