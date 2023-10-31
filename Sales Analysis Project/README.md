## Sales Analysis Project Overview

Welcome to the Sales Analysis Project! In this project, we will explore the data of a retail store chain called LES to gain valuable insights into their sales performance and customer behavior. To perform these analyses, we will be working with three main tables in the database:

### 1. Sales_fact
This table contains detailed data at the customer-transaction-product level, providing essential information about sales transactions.

### 2. Category_dim
The Category_dim table serves as a crucial resource for mapping products to their respective categories and sub-categories.

### 3. Geography_dim
Geography_dim is another significant table in our database, which helps us map store locations to specific cities, states, and countries.

**Important Note:** The "Priceusd" column in the dataset represents values in Indian Rupees (INR).

Let's proceed to break down the project into individual tasks and questions to uncover meaningful insights from the data. Enjoy the journey of data analysis and exploration!

------------------------------------------------------------------------------------------------------------------------------------------
#### Question 1: What is the highest sales amount in a day?

```sql
SELECT
  date,
  SUM(priceusd * quantity) AS sales
FROM
  `sales_fact`
GROUP BY
  date
ORDER BY
  sales DESC
LIMIT 1
```

#### Question 2: What is the total sales of the category ‘Dairy’?

```sql
SELECT
  category_desc,
  SUM(priceusd * quantity) AS sales
FROM
  `sales_fact` a
INNER JOIN
  `category_dim` b
ON
  a.product_id = b.product_id
GROUP BY
  category_desc
```

#### Question 3: Which city has the highest sales?

```sql
SELECT
  city,
  SUM(priceusd * quantity) AS sales
FROM
  `sales_fact` a
INNER JOIN
  `geography_dim` b
ON
  a.store_id = b.store_id
GROUP BY
  city
ORDER BY
  sales DESC
LIMIT 1
```

#### Question 4: How many customers spent less than Rs. 3000?

```sql
SELECT
  COUNT(*)
FROM
  (SELECT
    customer_id,
    SUM(priceusd * quantity) AS sales
  FROM
    `sales_fact` a
  GROUP BY
    customer_id
  HAVING
    sales < 3000
  ORDER BY
    sales DESC) a
```

#### Question 5: What is the sales of the category ‘Cereals’ in the city Bangalore?

```sql
SELECT
  category_desc,
  SUM(priceusd * quantity) AS sales
FROM
  `sales_fact` a
INNER JOIN
  `category_dim` b
ON
  a.product_id = b.product_id
INNER JOIN
  `geography_dim` c
ON
  a.store_id = c.store_id
WHERE
  city = 'Bangalore'
GROUP BY
  category_desc
```

#### Question 6: Which category is the top category in terms of sales for the location Mumbai?

```sql
SELECT
  category_desc,
  SUM(priceusd * quantity) AS sales
FROM
  `sales_fact` a
INNER JOIN
  `category_dim` b
ON
  a.product_id = b.product_id
INNER JOIN
  `geography_dim` c
ON
  a.store_id = c.store_id
WHERE
  city = 'Mumbai'
GROUP BY
  category_desc
ORDER BY
  sales DESC
```

#### Question 7: What is the highest sales value of category Drinks and Beverages in a single transaction? Value should reflect only sales of the mentioned category.

```sql
SELECT
  transaction_id,
  SUM(priceusd * quantity) AS sales
FROM
  `sales_fact` a
INNER JOIN
  `category_dim` b
ON
  a.product_id = b.product_id
WHERE
  category_desc = 'Drinks & Bevrages'
GROUP BY
  transaction_id
ORDER BY
  sales DESC
```

#### Question 8: What is the average amount spent per customer in Chennai?

```sql
SELECT
  city,
  SUM(priceusd * quantity) / COUNT(DISTINCT customer_id) AS avg_amt_spent
FROM
  `sales_fact` a
INNER JOIN
  `geography_dim` b
ON
  a.store_id = b.store_id
WHERE
  city = 'Chennai'
GROUP BY
  city
```

#### Question 9: What is the sales amount of the lowest selling product in ‘Cereals’?

```sql
SELECT
  a.product_id,
  SUM(priceusd * quantity) AS sales
FROM
  `sales_fact` a
INNER JOIN
  `category_dim` b
ON
  a.product_id = b.product_id
WHERE
  category_desc = 'Cereals'
GROUP BY
  a.product_id
ORDER BY
  sales ASC
```

#### Question 10: What is the average revenue per customer in Maharashtra?

```sql
SELECT
  state,
  SUM(priceusd * quantity) / COUNT(DISTINCT customer_id) AS ARPU
FROM
  `sales_fact` a
INNER JOIN
  `geography_dim` b
ON
  a.store_id = b.store_id
WHERE
  state = 'Maharashtra'
GROUP BY
  state
```

#### Question 11: How many customers in Karnataka spent less than Rs 3000?

```sql
SELECT
  COUNT(*)
FROM
  (SELECT
    customer_id,
    SUM(priceusd * quantity) AS sales
  FROM
    `sales_fact` a
  INNER JOIN
    `geography_dim` c
  ON
    a.store_id = c.store_id
  WHERE
    state = 'Karnataka'
  GROUP BY
    customer_id
  HAVING
    sales < 3000) a
```

#### Question 12: How many cities have average revenue per customer lesser than Rs 3500?

```sql
SELECT
  city,
  SUM(priceusd * quantity) / COUNT(DISTINCT customer_id) AS ARPU
FROM
  `sales_fact` a
INNER JOIN
  `geography_dim` c
ON
  a.store_id = c.store_id
GROUP BY
  city
HAVING
  ARPU < 3500
ORDER BY
  ARPU DESC
```

#### Question 13: Which product was bought by the most number of customers?

```sql
SELECT
  product_id,
  COUNT(DISTINCT customer_id) AS customers
FROM
  `sales_fact`
GROUP BY
  product_id
ORDER BY
  customers DESC
```

#### Question 14: How many products were bought by at least 5 customers in Maharashtra?

```sql
SELECT
  COUNT(*)
FROM
  (SELECT
    product_id,
    COUNT(DISTINCT customer_id) AS customers
  FROM
    `geography_dim` a
  LEFT JOIN
    `sales_fact` b
  ON
    a.store_id = b.store_id
  WHERE
    state = 'Maharashtra'
  GROUP BY
    product_id
  HAVING
    customers >= 5) a
```

#### Question 15: What is the highest average amount spent per product by a customer?

```sql
SELECT
  customer_id,
  SUM(priceusd * quantity) / SUM(quantity) AS avg_amt_spent
FROM
  `sales_fact`
GROUP BY
  customer_id
ORDER BY
  avg_amt_spent DESC
LIMIT 1
```
