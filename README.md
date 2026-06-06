# Do Hawaiian Hotels That Respond to Reviews Have Higher Review/Rates

This is a final project for dsc80
By Gian Carlo Robles

## Introduction

Hawaii's tourism industry is one of the largest in the United States, with hotels and accommodations playing a central role in shaping visitor experiences. Online reviews — particularly on Google Maps — are now one of the most important tools potential guests use when choosing where to stay. Hotel operators are constantly told to engage with these reviews by responding to feedback, but this engagement has real costs: staffing, training, and brand-consistent response policies. The question this project investigates is whether the data actually supports the advice:

> **Among hotels in Hawaii, does active engagement with customer reviews — measured by response rate, response delay, and response text — correspond to higher customer ratings?**

Answering this matters for prospective hotel owners and operators trying to decide how to allocate their limited time and resources. If review engagement is meaningfully linked to ratings, the investment is justified; if not, owners can focus elsewhere.

The dataset comes from Google Maps reviews of businesses in Hawaii, originally scraped and released by the authors of [this paper](https://aclanthology.org/2022.acl-long.426.pdf). After merging the review-level data with business-level metadata and filtering to lodging-type categories (Hotel, Resort hotel, Motel, Inn, Hostel, Bed & Breakfast, etc.), the working dataset contains **68,996 reviews** across **329 hotels**.

**Relevant columns used in the analysis:**

| Column            | Description                                                       |
| ----------------- | ----------------------------------------------------------------- |
| `gmap_id`         | Unique business identifier (used to join review and meta data)    |
| `Review_Rating`   | The star rating given by the reviewer (1–5)                       |
| `avg_rating`      | The hotel's overall average rating on Google Maps                 |
| `Review_Time`     | Timestamp of the customer's review                                |
| `Resp_Time`       | Timestamp of the business's response (if any)                     |
| `Respond Comment` | Text of the business's response                                   |
| `category`        | Business categories, used to filter to hotels                     |
| `num_of_reviews`  | Total number of reviews the hotel has on Google                   |