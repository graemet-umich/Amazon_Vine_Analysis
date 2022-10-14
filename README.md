# Amazon_Vine_Analysis
Module 16: Big Data

## Overviewe


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


