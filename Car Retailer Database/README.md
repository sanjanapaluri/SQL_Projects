# Car Retailer Database

The "Car Retailer" database is a retailer of scale models of classic cars. It contains essential business data such as customer information, products, sales orders, sales order line items, and more. This README provides an overview of the database schema and its tables.

## Database Schema

The database consists of the following tables:

1. `cr_customers`: Contains customer information.
2. `cr_employees`: Stores employee data.
3. `cr_offices`: Contains information about the retailer's offices.
4. `cr_orderdetails`: Contains details about items in sales orders.
5. `cr_orders`: Stores information about sales orders.
6. `cr_payments`: Contains payment details.
7. `cr_productlines`: Stores product line information.
8. `cr_products`: Contains data about individual products.

![image](https://github.com/sanjanapaluri/SQL_Projects/assets/127730680/c95a57f7-0bb4-4cac-b540-1326a297d87e)


------------------------------------------------------------------------------------------------------------------------------------------
#### Question 1: Find out the top 3 Products (“productLine”) for each month that generated the highest profit.
    Note: Profit = ( Price per unit - Buying Price per Unit ) x No. of Units Ordered
Sample Ouput: 

![image](https://github.com/sanjanapaluri/SQL_Projects/assets/127730680/204f2c77-dc4f-48ce-b9d4-c48e79d8c55e)


#### Which “productline” generated the highest profit in the June Month? 
#### Submit the 2nd highest profit amount for the month of October. 
#### Submit the “productline” which is not in the top 3 lists in the month of November.

```sql
WITH mains AS (
    SELECT
        FORMAT(orderDate, 'MMMM') AS month,
        productline,
        (priceEach - buyprice) * quantityOrdered AS profit
    FROM
        cr_orderdetails cod
    JOIN
        cr_products cp ON cod.productCode = cp.productCode
    JOIN
        cr_orders co ON cod.orderNumber = co.orderNumber
)

SELECT
    *
FROM
    (
        SELECT
            month,
            productline,
            SUM(profit) AS profit,
            DENSE_RANK() OVER (PARTITION BY month ORDER BY SUM(profit) DESC) AS rank
        FROM
            mains
        GROUP BY
            month,
            productline
    ) a
WHERE
    rank <= 3
```

#### Question 2: Find the Month on Month growth in profit for each year.
    MoM_growth is calculated as follows:

![271776671-01f50eac-e8f1-498b-af8f-c48075e15c1a](https://github.com/sanjanapaluri/SQL_Projects/assets/127730680/e2f7a5fc-706a-4a87-b3a0-f7984cffce4d)


 Note: Please make sure that MoM is calculated for each year. Meaning, that ideally MoM growth of the first month 
    of every year should be NULL as shown in the sample output below.

Sample Output:

![image](https://github.com/sanjanapaluri/SQL_Projects/assets/127730680/5d828ff9-3fe9-482b-b57d-28ce299219d1)


Only the first 3 and last months are shown for a respective year. This is the sample output. You need to have all the months of a year
in the final output.

#### What is the MoM for Feb 2005? 
#### What is the MoM for June 2004? 
#### For the year 2004, select which month shows negative MoM growth?

```sql
select
    Year,
    monthname,
    profit,
    (profit / lag(profit, 1, null) over (partition by year order by month) - 1) * 100 as MoM_Growth
from
    (select
        year(orderDate) as Year,
        month(orderDate) as Month,
        format(orderDate, 'MMMM') as monthname,
        sum((priceEach - buyPrice) * quantityOrdered) as profit
    from
        cr_orderdetails cod
        join cr_products crp on cod.productCode = crp.productCode
        join cr_orders cro on cod.orderNumber = cro.orderNumber
    group by
        year(orderDate),
        month(orderDate),
        format(orderDate, 'MMMM')
    ) a;
```

#### Question 3: For each customer i.e (customerNumber) count the number of orders above and below the average order value.
    “Average Order Value(AOV)” is calculated as below,
  ![image](https://github.com/sanjanapaluri/SQL_Projects/assets/127730680/7cbb3d49-125e-489b-992c-b83653aa1be0)


  Total Revenue is calculated as the sum of the revenue of each order.
    Each order revenue is Price per unit * No. of Units.
    Conditions to get following details,
    Order Above AVG = Actual Order Value >= AVG Order Value
    Order Below AVG = Actual Order Value < AVG Order Value

   ![image](https://github.com/sanjanapaluri/SQL_Projects/assets/127730680/00eb455d-6c6c-4231-bfaa-a324da0ef684)


   #### What is the Average Order Value? 
    Note: Round off the value to 2 decimal and use it for the rest of the calculation.
#### Submit the number of orders above and below AOV for customer number 205.

 ```sql
SELECT
    customernumber,
    contactfirstname,
    contactlastname,
    SUM(order_value >= avg_order_value) AS order_above_avg,
    SUM(order_value < avg_order_value) AS order_below_avg
FROM
    (SELECT
        c.customernumber,
        c.contactfirstname,
        c.contactlastname,
        o.ordernumber,
        od.quantityordered,
        od.priceeach,
        priceeach * quantityordered AS order_value,
        ROUND(AVG(priceeach * quantityordered) OVER (), 2) AS avg_order_value
    FROM
        cr_orderdetails od
        JOIN cr_orders o ON od.ordernumber = o.ordernumber
        JOIN cr_customers c ON c.customernumber = o.customernumber
    ) a
GROUP BY
    customernumber,
    contactfirstname,
    contactlastname;
```

#### Question 4: In the business, customer order the product in bulk and it is the utmost priority of the business to fulfill the order requirement as to when required. Hence a business has to keep checking their stock regularly to give uninterrupted delivery.
    To make that happen, create a report as shown below to get the updated stock and status to check if we are running
    out of stock (or are we going to fulfill the next customer order?)
  ![image](https://github.com/sanjanapaluri/SQL_Projects/assets/127730680/47c7a399-de91-4ac2-8e02-218e2a733aeb)


  Consider the first row as an example where product code “S18_2248”. Its initial stock quantity was 540. After the 
    first order i.e orderNumber 10100 from the customer i.e order quantity of 50, the updated stock value would be 
    (540 - 50) 490. Now, the status needs to be updated based on the next order quantity, for example, the Next order
    is of quantity 32 and we have 490 products in stock hence we can easily serve the next order request (Status is Yes)
    
    But if you look at the blue highlighted row then, in that case, “updatedStock” value is 12 and next order is of 
    quantity 46 hence the status would No.
#### Find the list of productCodes that will be getting out of Stock (i.e “are_we_going_to_fulfill_next_order” value is No)

```sql
SELECT
    orderDate,
    orderNumber,
    productCode,
    quantityOrdered,
    quantityInStock,
    product_sold_so_far,
    quantityInStock - product_sold_so_far AS updated_stock,
    IF(quantityInStock - product_sold_so_far - LEAD(quantityOrdered, 1, 0) OVER (PARTITION BY productCode ORDER BY orderDate) > 0, "yes", "no") AS Are_we_going_to_sell
FROM
    (SELECT
        c.orderDate,
        c.orderNumber,
        a.productCode,
        b.quantityOrdered,
        a.quantityInStock,
        SUM(b.quantityOrdered) OVER (PARTITION BY a.productCode ORDER BY c.orderDate) AS product_sold_so_far
    FROM
        cr_products a
        JOIN cr_orderdetails b ON a.productCode = b.productCode
        JOIN cr_orders c ON b.orderNumber = c.orderNumber
    ) m;
```

#### Question 5: Find all products whose quantity exceeds the average quantities of all product categories.

    How to get the product category?
    Split the productCode by “_” and take the left part of it. For example, S700_3962 will result in the S700 product category.
```sql
WITH averagequantities AS (
    SELECT
        LEFT(p.productcode, CHARINDEX('_', p.productcode) - 1) AS productcategory,
        AVG(od.quantityordered) AS avgquantity
    FROM
        cr_orderdetails od
        JOIN cr_products p ON od.productcode = p.productcode
    GROUP BY
        LEFT(p.productcode, CHARINDEX('_', p.productcode) - 1)
)

select aq.productcategory, p.productname, od.quantityordered, aq.avgquantity
from cr_orderdetails od
join cr_products p on od.productcode = p.productcode
join averagequantities aq on aq.productcategory = left(p.productcode, charindex('_', p.productcode) - 1)
where od.quantityordered > aq.avgquantity
```
