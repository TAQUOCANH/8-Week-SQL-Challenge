<p align="center"><h1><b>Case Study #4: Data Bank</b></h1> </p>
<p align="center">
<img src="https://8weeksqlchallenge.com/images/case-study-designs/4.png" alt="Image" width="450" height="450">

View the case study [here](https://8weeksqlchallenge.com/case-study-4/)
  
## Table Of Contents
  - [A. Customer Nodes Exploration](https://github.com/TAQUOCANH/8-Week-SQL-Challenge/blob/main/Case%20Study%20%23%204%20-%20Data%20Bank/README.md#a-customer-nodes-exploration)
  - 
  
## Introduction
There is a new innovation in the financial industry called Neo-Banks: new aged digital only banks without physical branches.

Danny thought that there should be some sort of intersection between these new age banks, cryptocurrency and the data world…so he decides to launch a new initiative - Data Bank!

Data Bank runs just like any other digital bank - but it isn’t only for banking activities, they also have the world’s most secure distributed data storage platform!

Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. There are a few interesting caveats that go with this business model, and this is where the Data Bank team need your help!

The management team at Data Bank want to increase their total customer base - but also need some help tracking just how much data storage their customers will need.

This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments!


## Problem Statement
The management team at Data Bank want to increase their total customer base - but also need some help tracking just how much data storage their customers will need.

The following case study questions include some general data exploration analysis for the nodes and transactions before diving right into the core business questions and finishes with a challenging final request!


## Datasets
![image](https://user-images.githubusercontent.com/77529445/165748352-09dfcafd-07a6-4bf0-b171-7ba0ec75aa22.png)
  
## Case Study Solutions
### A. Customer Nodes Exploration
#### 1. How many unique nodes are there on the Data Bank system?
```sql
SELECT 
	COUNT(DISTINCT node_id) AS unique_nodes
FROM data_bank.customer_nodes;
```
Result:

| unique_nodes |
| ------------ |
| 5            |

#### 2. What is the number of nodes per region?
```sql
SELECT
	region_id
	,region_name
    , COUNT(node_id) AS n_of_nodes
FROM data_bank.customer_nodes
LEFT JOIN data_bank.regions
USING(region_id)
GROUP BY region_id, region_name;
```

Result:

| region_id | region_name | n_of_nodes |
| --------- | ----------- | ---------- |
| 2         | America     | 735        |
| 5         | Europe      | 616        |
| 3         | Africa      | 714        |
| 4         | Asia        | 665        |
| 1         | Australia   | 770        |

#### 3. How many customers are allocated to each region?
```sql
SELECT
	region_id
	,region_name
    , COUNT(DISTINCT customer_id) AS n_of_customers
FROM data_bank.customer_nodes 
LEFT JOIN data_bank.regions
USING(region_id)
GROUP BY region_id, region_name;
```

Result:

| region_id | region_name | n_of_customers |
| --------- | ----------- | -------------- |
| 1         | Australia   | 110            |
| 2         | America     | 105            |
| 3         | Africa      | 102            |
| 4         | Asia        | 95             |
| 5         | Europe      | 88             |

#### 4. How many days on average are customers reallocated to a different node?
```sql
SELECT ROUND(AVG(DATEDIFF(end_date, start_date)), 2) AS avg_days
FROM data_bank.customer_nodes
WHERE end_date!='9999-12-31';
```
**Result:**

| avg_days |
| ------------ |
| 14.63        |


#### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?


```sql
WITH reallocation_days_cte AS(
SELECT
	*
	,(DATEDIFF(end_date, start_date)) AS reallocation_days
FROM customer_nodes
INNER JOIN regions USING (region_id)
WHERE end_date!='9999-12-31')

,percentile_cte AS(
SELECT
	*
	,percent_rank() over(PARTITION BY region_id
                              ORDER BY reallocation_days)*100 AS p
FROM reallocation_days_cte)
```



**Median:**

```sql
SELECT
	region_id
       ,region_name,
       ,reallocation_days
FROM percentile_cte
WHERE p >50
GROUP BY region_id;
```

**Result:**

| region_id | region_name | reallocation_days |
| --------- | ----------- | ----------------- |
| 1         | Australia   | 16                |
| 2         | America     | 16                |
| 3         | Africa      | 16                |
| 4         | Asia        | 16                |
| 5         | Europe      | 16                |


**80th percentile:**

```sql
SELECT
	region_id
       ,region_name,
       ,reallocation_days
FROM percentile_cte
WHERE p >80
GROUP BY region_id;
```

| region_id | region_name | reallocation_days |
| --------- | ----------- | ----------------- |
| 1         | Australia   | 24                |
| 2         | America     | 24                |
| 3         | Africa      | 25                |
| 4         | Asia        | 24                |
| 5         | Europe      | 25                |

**95th percentile:**

```sql
SELECT
	region_id
       ,region_name,
       ,reallocation_days
FROM percentile_cte
WHERE p >95
GROUP BY region_id;
```

| region_id | region_name | reallocation_days |
| --------- | ----------- | ----------------- |
| 1         | Australia   | 29                |
| 2         | America     | 29                |
| 3         | Africa      | 29                |
| 4         | Asia        | 29                |
| 5         | Europe      | 29                |

### B. Customer Transactions

#### 1. What is the unique count and total amount for each transaction type?

```sql
SELECT 
	txn_type
    ,COUNT(*) AS unique_count
    ,SUM(txn_amount) AS total_amont
FROM data_bank.customer_transactions
GROUP BY txn_type;
```

| txn_type   | unique_count | total_amont |
| ---------- | ------------ | ----------- |
| deposit    | 2671         | 1359168     |
| withdrawal | 1580         | 793003      |
| purchase   | 1617         | 806537      |

#### 2. What is the average total historical deposit counts and amounts for all customers?

```sql
WITH customer_deposits AS (
    SELECT
        customer_id,
        COUNT(txn_amount) AS deposit_count,
        SUM(txn_amount) AS total_deposit_amount
    FROM data_bank.customer_transactions
    WHERE txn_type = 'deposit'
    GROUP BY customer_id
)

SELECT
    ROUND(AVG(deposit_count),0) AS average_deposit_count
    ,ROUND(AVG(total_deposit_amount),2) AS average_total_deposit_amount
FROM customer_deposits;
```
**Result:**
| average_deposit_count   | average_total_deposit_amount | 
| ---------- | ------------ | 
| 5    | 508.61         | 


#### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?


```sql
WITH monthly_transactions AS (
  SELECT 
    customer_id, 
    DATE_PART('month', txn_date) AS month,
    SUM(CASE WHEN txn_type = 'deposit' THEN 0 ELSE 1 END) AS deposit_count,
    SUM(CASE WHEN txn_type = 'purchase' THEN 0 ELSE 1 END) AS purchase_count,
    SUM(CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END) AS withdrawal_count
  FROM data_bank.customer_transactions
  GROUP BY customer_id, DATE_PART('month', txn_date)
)

SELECT
  month,
  COUNT(DISTINCT customer_id) AS customer_count
FROM monthly_transactions
WHERE deposit_count > 1 
  AND (purchase_count >= 1 OR withdrawal_count >= 1)
GROUP BY month
ORDER BY month;
```
**Result:**

|month|customer_count|
|:----|:----|
|1|170|
|2|277|
|3|292|
|4|103|

#### 4. What is the closing balance for each customer at the end of the month?






