![AWS Logo](./Resources/AWS_logo.png) ![PostgreSQL Logo](./Resources/PostgreSQL_logo.png) ![Amazon Vine Logo](./Resources/amazon_vine_logo.png)

# Amazon Vine Analysis
Big Data: Analyze Vine Reviews with AWS S3 Scalable Storage in the Cloud and Postgres AWS RDS Managed Relational Database Service

## Overview



## Results


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

![Table Counts](./Resources/table_counts.png)


## Summary


