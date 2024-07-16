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
