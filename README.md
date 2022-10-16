![Amazon Vine Logo](./Resources/amazon_vine_logo.png) ![AWS Logo](./Resources/AWS_logo.png) ![PostgreSQL Logo](./Resources/PostgreSQL_logo.png)

# Amazon Vine Analysis
Big Data: Analyze Vine Reviews with AWS S3 Scalable Storage in the Cloud and PostgreSQL AWS RDS Managed Relational Database Service

## Overview

Amazon Vine is a program where Amazon selects what it views as its "best" reviewers and invites them to be Vine Voices. A Vine Voice is not paid, but she is expected to order selected products free of change and is required to submit an insightful, honest, and helpful review, whether the review be negative, neutral, or positive. A Vine Voice review is tagged with a special badge, and other potential buyers may give more consideration to a Vine Voice tagged review.

Companies, such as manufacturers and publishers, who participate in Amazon Vine pay Amazon a small fee and supply Vine Voices with free products that the Vine Voices are required to review. A company may benefit from the program by giving more consideration to a Vine Voice tagged review.

In this study, the company SellBy wants to know if there is any bias in review star ratings between Vine Voices and non-Vine reviewers. The specific focus here is on products one buys at grocery stores.
 

## Methods

This study is an exercise in cloud-based ETL, cloud-based data retrieval, and data analysis. This section outlines a typical cloud-based workflow.

The dataset used contains 2,402,431 reviews of grocery products as a tab separated value file `amazon_reviews_us_Grocery_v1_00.tsv` obtained from [Amazon Review Datasets](https://s3.amazonaws.com/amazon-reviews-pds/tsv/index.txt).

### ETL: Extract, Transform, Load

The first part of the study is cloud-based ETL.

#### Extract

- Obtain the tsv file
- Store the tsv in an AWS S3 bucket

#### Transform

- Use PySpark in Google Colaboratory to create a DataFrame from the tsv
- Remove mis-parsed records that occurred during spark.read of the tsv
- Create four DataFrames to match the four database tables

#### Load

- Create a PostgreSQL database on AWS RDS using pgAdmin
- Create database tables using pgAdmin

```
CREATE TABLE review_id_table (
  review_id TEXT PRIMARY KEY NOT NULL,
  customer_id INTEGER,
  product_id TEXT,
  product_parent INTEGER,
  review_date DATE -- this should be in the formate yyyy-mm-dd
);

-- This table will contain only unique values
CREATE TABLE products_table (
  product_id TEXT PRIMARY KEY NOT NULL UNIQUE,
  product_title TEXT
);

-- Customer table for first data set
CREATE TABLE customers_table (
  customer_id INT PRIMARY KEY NOT NULL UNIQUE,
  customer_count INT
);

-- vine table
CREATE TABLE vine_table (
  review_id TEXT PRIMARY KEY,
  star_rating INTEGER,
  helpful_votes INTEGER,
  total_votes INTEGER,
  vine TEXT,
  verified_purchase TEXT
);
```
Fig. SQL script. Create database tables with given schema.

- Connect to the AWS RDS instance and write each DataFrame to its database table
- Inspect each DataFrame for correct schema (see [Amazon_Reviews_ETL.ipynb](https://github.com/graemet-umich/Amazon_Vine_Analysis/blob/main/Amazon_Reviews_ETL.ipynb))
- Get table counts using pgAdmin as another check that the database tables were written to correctly

```
-- Get table counts
SELECT  (
        SELECT COUNT(*)
        FROM   customers_table
        ) AS customer_count,
        (
        SELECT COUNT(*)
        FROM   products_table
        ) AS product_count,
        (
        SELECT COUNT(*)
        FROM   review_id_table
        ) AS review_id_count,
        (
        SELECT COUNT(*)
        FROM   vine_table
        ) AS vine_count
```
Fig. SQL Query. Get the total number of records from each of the database tables.

![Table Counts](./Resources/table_counts.png)
Table. The number of records in each of the four PostgreSQL tables. 1,363,975 customers generated 2,402,431 reviews of 305,551 grocery products.

### Prepare Data for Bias of Vine Reviews Analysis

- Use PySpark in Google Colaboratory to read the vine_count table stored in the AWS RDS PostgreSQL database into a DataFrame
- Only consider reviews with at least 20 total votes
- Only consider reviews where at least 50% of the total votes are helpful votes
- Separate Vine reviews and non-Vine reviews into two DataFrames for analysis.

| State | Number of Records |
| --- | ---: |
| original tsv | 2,402,458 |
| remove mis-parsed records | 2,402,431 |
| total_votes >= 20 | 31,518 |
| helpful to total votes >= 0.5 | 28,348 |
| Vine Voices reviews | 61 |
| non-Vine reviews | 28,287 |

Table. How filtering and splitting the data affect the number of records (see [Vine_Review_Analysis.ipynb](https://github.com/graemet-umich/Amazon_Vine_Analysis/blob/main/Vine_Review_Analysis.ipynb) for details).


## Results

The summary statistics are displayed in the table below to determine the bias of Vine Voices reviews compared to non-Vine reviews.

- **Total Vine Voices and non-Vine reviews.** The total number of Vine reviews is 61, and the total number of non-Vine reviews is 28,287.
- **5-star Vine and non-Vine reviews.** The total number of 5-star Vine reviews is 20, and the total number of 5-star non-Vine reviews is 15,689.
- **Percentage of 5-star Vine and non-Vine reviews.** The percentage of 5-star Vine reviews is 33%, and the percentage of 5-star non-Vine reviews is 55.5%.

| Summary | Vine Voices | non-Vine Reviewers |
| --- | :---: | :---: |
| total reviews | 61 | 28,287 |
| 5-star reviews | 20 | 15,689 |
| percentage 5-star | 33% | 55.5% |

Table. Summary of the number of total reviews, the number of 5-star reviews, and the percentage of 5-star reviews for Vine Voices and non-Vine reviewers. 

- **Significance of 5-star review percentage difference between Vine Voices and non-Vine reviewers.** The percentage of 5-star reviews for Vine Voices (33%) is significantly less than the percentage of 5-star reviews for non-Vine reviewers (55.5%) (p=0.0003).

![Two-Sample Test for Equality of Proportions](./Resources/2-sample_prop_test.png)
Fig. A two-sample test for equality of proportions in R. The reviews_5star_count is the number of 5-star reviews. The reviews_total_count is the total number of reviews. The first element of each vector refers to the number of reviews from Vine Voices. The second element of each vector refers to the number of reviews from non-Vine reviewers.


## Summary

There is significant bias between the 5-star reviews of Vine Voices and non-Vine reviewers. Vine Voices give grocery products only 5-star ratings 33% of the time, whereas non-Vine reviews give grocery products 5-star reviews 55.5% of the time. A two-sample test for equality of proportions (p=0.003) shows that the Vine Voices percentage is significantly *less* than the non-Vine reviewers percentage. 

There are several reasons why the percentages may differ:
- Vine Voices have a genuinely different review perspectives (insightful, honest) than non-Vine reviewers.
- Perhaps Vine Voices are asked to review novel products likely not long on the market. Non-Vine reviewers not only review Vine Voices products, but many more other products more likely to be long established and well-received by the public.
- There are likely biased company/product affiliated reviewers who indisciminately give 5-star reviews.

To further assess bias between Vine Voices reviews and non-Vine reviews, the following additional studies are proposed:
- Analyze more reviews by selecting reviews with at least, say, 10 votes instead of the 20 votes required in this study.
- Repeat this study except check for bias in 1-star, 2-star, 3-star, and 4-star reviews.
    - If Vine Voices give fewer 4-star reviews, the conclusion of bias is strengthened. 
    - If Vine Voices give more 1-star and 2-star reviews, the conclusion of bias is strengthened.
    - If Vine Voices give fewer 1-star reviews, this result is an indication tha Vine Voices are more circumspect than non-Vine reviewers: Vine Voices are unwilling to give products the best or worst possible star rating.
