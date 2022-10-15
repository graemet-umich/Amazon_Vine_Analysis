![Amazon Vine Logo](./Resources/amazon_vine_logo.png) ![AWS Logo](./Resources/AWS_logo.png) ![PostgreSQL Logo](./Resources/PostgreSQL_logo.png)

# Amazon Vine Analysis
Big Data: Analyze Vine Reviews with AWS S3 Scalable Storage in the Cloud and PostgreSQL AWS RDS Managed Relational Database Service

## Overview



## Results

The dataset used contains 2,402,431 reviews of grocery products as a tab separated value file `amazon_reviews_us_Grocery_v1_00.tsv` obtained from [Amazon Review Datasets](https://s3.amazonaws.com/amazon-reviews-pds/tsv/index.txt). 

### ETL: Extract, Transform, Load

The first part of the study is cloud-based ETL.

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
Fig. Create database tables with given schema.

- Obtain the tsv file
- Store the tsv in an AWS S3 bucket
- Use PySpark in Google Colaboratory to create a DataFrame from the tsv
- Remove mis-parsed records that occurred during spark.read of the tsv
- Create four DataFrames to match the four database tables
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
SQL Query. Get the total number of records from each of the database tables.

![Table Counts](./Resources/table_counts.png)
Table. The number of records in each of the four PostgreSQL tables. 1,363,975 customers generated 2,402,431 reviews of 305,551 grocery products.


### Determine Bias of Vine Reviews

# TODO below


| State | Number of Records |
| --- | ---: |
| original tsv | 2,402,458 |
| remove mis-parsed records | 2,402,431 |
| total_votes >= 20 | 31,518 |
| helpful to total votes >= 0.5 | 28,348 |
| vine reviews | 61 |
| non-Vine reviews | 28,287 |

Table. 

![Two-Sample Test for Equality of Proportions](./Resources/2-sample_prop_test.png)
Fig. A two-sample test for equality of proportions in R. The paid_vine_count is the number of 5-star reviews. The total_count is the total number of reviews. The first element of each vector refers to the number of Vine reviews. The second element of each vector refers to the number of non-Vine reviews. The percentage of 5-star Vine reviews (33%) is significantly less than the percentage of 5-star non-Vine reviews (55.5%) (p=0.0003).

## Summary


