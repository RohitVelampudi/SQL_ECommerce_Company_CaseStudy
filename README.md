# SQL_ECommerce_Company_CaseStudy

## 1. Identify the top 3 cities with the highest number of customers to determine key markets for targeted marketing and logistic optimization.

```sql
SELECT location, COUNT(*) AS number_of_customers
FROM Customers
GROUP BY location
ORDER BY number_of_customers DESC
LIMIT 3;
```
Output:

| location | number_of_customers |
|----------|---------------------|
| Delhi    |                  16 |
| Chennai  |                  15 |
| Jaipur   |                  11 |

Outcome: Delhi, Chennai, Jaipur are the cities must be focused as a part of marketing strategies.

## 2. Determine the distribution of customers by the number of orders placed. This insight will help in segmenting customers into one-time buyers, occasional shoppers, and regular customers for tailored marketing strategies.

```sql
SELECT NumberOfOrders, COUNT(customer_id) AS CustomerCount
FROM (
    SELECT customer_id, COUNT(order_id) AS NumberOfOrders
    FROM Orders
    GROUP BY customer_id
) AS customer_orders
GROUP BY NumberOfOrders
ORDER BY NumberOfOrders ASC;
```

Output:

| NumberOfOrders | CustomerCount |
|----------------|---------------|
|              1 |            26 |
|              2 |            26 |
|              3 |            18 |
|              4 |             6 |
|              5 |             6 |
|              6 |             1 |
|              8 |             1 |

Outcome: As the Number of orders increases, the Customer count decreases. And,Considering 2-4 Orders as 'Occasional shoppers', this ustomers category does the company experiences the most.

## 3. Identify products where the average purchase quantity per order is 2 but with a high total revenue, suggesting premium product trends.

```sql
select product_id as Product_Id,
  avg(quantity) as AvgQuantity,
  sum(quantity*price_per_unit) as TotalRevenue
from OrderDetails
group by product_id
having AvgQuantity=2
order by AvgQuantity ,TotalRevenue desc
;
```

Output:

| Product_Id | AvgQuantity | TotalRevenue |
|------------|-------------|--------------|
|          1 |      2.0000 |      1620000 |
|          8 |      2.0000 |       390000 |

Outcome: Product 1 exhibit the highest total revenue among products with an average purchase quantity of two.

## 4. For each product category, calculate the unique number of customers purchasing from it. This will help understand which categories have wider appeal across the customer base.

```sql
select p.category, count(distinct(o.customer_id)) as unique_customers from
Products p 
join OrderDetails od on od.product_id=p.product_id
join Orders o on o.order_id=od.order_id
group by p.category
order by unique_customers desc;
```
Output:

| category      | unique_customers |
|---------------|------------------|
| Electronics   |               79 |
| Wearable Tech |               61 |
| Photography   |               45 |

Outcome: Electronics category needs more focus as it is in high demand among the customers

## 5. Analyze the month-on-month percentage change in total sales to identify growth trends.

```sql
WITH monthly_sales AS (
    SELECT 
        DATE_FORMAT(order_date, '%Y-%m') AS Month,
        SUM(total_amount) AS TotalSales,
        LAG(SUM(total_amount), 1) OVER (ORDER BY DATE_FORMAT(order_date, '%Y-%m')) AS PreviousMonthSales
    FROM Orders
    GROUP BY Month
)

SELECT Month, TotalSales,
  ROUND(((TotalSales-PreviousMonthSales)/PreviousMonthSales)*100,2) as PercentChange
FROM monthly_sales
ORDER BY Month;
```
Output:

| Month   | TotalSales | PercentChange |
|---------|------------|---------------|
| 2023-03 |     789000 |          NULL |
| 2023-04 |    1704000 |        115.97 |
| 2023-05 |    1582000 |         -7.16 |
| 2023-06 |    1040000 |        -34.26 |
| 2023-07 |    2568000 |        146.92 |
| 2023-08 |    1800000 |        -29.91 |
| 2023-09 |    2927000 |         62.61 |
| 2023-10 |    1497000 |        -48.86 |
| 2023-11 |    1151000 |        -23.11 |
| 2023-12 |    2774000 |        141.01 |
| 2024-01 |    1555000 |        -43.94 |
| 2024-02 |     396000 |        -74.53 |

Outcome: As per Sales Trend Analysis question, During Feb 2024 the sales experienced the largest decline.And, Sales fluctuated with no clear trend from March to August.

## 6. Examine how the average order value changes month-on-month. Insights can guide pricing and promotional strategies to enhance order value.

```sql
WITH monthly_sales AS (
    SELECT 
        DATE_FORMAT(order_date, '%Y-%m') AS Month,
        avg(total_amount) AS AvgOrderValue,
        LAG(avg(total_amount), 1) OVER (ORDER BY DATE_FORMAT(order_date, '%Y-%m')) AS PreviousMonthSales
    FROM Orders
    GROUP BY Month
)

SELECT Month, AvgOrderValue,
  ROUND((AvgOrderValue-PreviousMonthSales),2) as ChangeInValue
FROM monthly_sales
ORDER BY ChangeInValue desc;
```

output:

| Month   | AvgOrderValue | ChangeInValue |
|---------|---------------|---------------|
| 2023-12 |   132095.2381 |      36178.57 |
| 2023-04 |    81142.8571 |      20450.55 |
| 2023-06 |   104000.0000 |      16111.11 |
| 2023-08 |   112500.0000 |      13730.77 |
| 2023-11 |    95916.6667 |      12750.00 |
| 2023-09 |   121958.3333 |       9458.33 |
| 2023-05 |    87888.8889 |       6746.03 |
| 2024-01 |   129583.3333 |      -2511.90 |
| 2023-07 |    98769.2308 |      -5230.77 |
| 2023-10 |    83166.6667 |     -38791.67 |
| 2024-02 |    44000.0000 |     -85583.33 |
| 2023-03 |    60692.3077 |          NULL |

outcome: December has the highest change in the average order value.

## 7.Based on sales data, identify products with the fastest turnover rates, suggesting high demand and the need for frequent restocking.

```sql
select product_id,
  count(*) as SalesFrequency
from OrderDetails
group by product_id
order by SalesFrequency desc
limit 5;
```

output:

| product_id | SalesFrequency |
|------------|----------------|
|          7 |             78 |
|          3 |             68 |
|          4 |             68 |
|          2 |             67 |
|          8 |             65 |

outcome: The product_id '7' has the highest turnover rates and needs to be restocked frequently.

## 8. List products purchased by less than 40% of the customer base, indicating potential mismatches between inventory and customer interest.

```sql
WITH ProductCustomerCounts AS (
    SELECT 
        p.product_id,
        p.name,
        COUNT(DISTINCT o.customer_id) AS unique_customer_count
    FROM 
        Products p
    JOIN 
        OrderDetails od ON od.product_id = p.product_id
    JOIN 
        Orders o ON o.order_id = od.order_id
    GROUP BY 
        p.product_id, p.name
),
CustomerBase AS (
    SELECT 
        COUNT(DISTINCT customer_id) AS total_customers
    FROM 
        Customers
),
FilteredProducts AS (
    SELECT 
        p.product_id,
        p.name,
        p.unique_customer_count,
        (c.total_customers * 0.4) AS threshold_40_percent
    FROM 
        ProductCustomerCounts p,
        CustomerBase c
    WHERE 
        p.unique_customer_count < (c.total_customers * 0.4)
)
SELECT 
    product_id,
    name, 
    unique_customer_count as UniqueCustomerCount
FROM 
    FilteredProducts;
```
output:
| product_id | name             | UniqueCustomerCount |
|------------|------------------|---------------------|
|          1 | Smartphone 6"    |                  36 |
|          8 | Wireless Earbuds |                  38 |

outcome: Poor visibility on the platform might caused certain products have purchase rates below 40% of the total customer base.
Implement targeted marketing campaigns to raise awareness and interest could be a strategic action to improve the sales of these underperforming products.

## 9. Evaluate the month-on-month growth rate in the customer base to understand the effectiveness of marketing campaigns and market expansion efforts.

```sql
with firstmonthsale as (

    select min(date_format(order_date,'%Y-%m')) as FirstPurchaseMonth,customer_id
    from orders
    group by customer_id

)

select FirstPurchaseMonth, count(*) as TotalNewCustomers
from firstmonthsale
group by FirstPurchaseMonth
order by FirstPurchaseMonth;
```

output: 
| FirstPurchaseMonth | TotalNewCustomers |
|--------------------|-------------------|
| 2023-03            |                11 |
| 2023-04            |                18 |
| 2023-05            |                11 |
| 2023-06            |                 8 |
| 2023-07            |                11 |
| 2023-08            |                 9 |
| 2023-09            |                 5 |
| 2023-10            |                 3 |
| 2023-11            |                 1 |
| 2023-12            |                 4 |
| 2024-01            |                 2 |
| 2024-02            |                 1 |

outcome: It is downward trend in customer base, which implies the marketing campaign are not much effective.

## 10.Identify the months with the highest sales volume, aiding in planning for stock levels, marketing efforts, and staffing in anticipation of peak demand periods.

```sql
select date_format(order_date,'%Y-%m') as Month,
  sum(total_amount) as TotalSales 
from Orders
group by month
order by TotalSales desc
limit 3;
```
output:
| Month   | TotalSales |
|---------|------------|
| 2023-09 |    2927000 |
| 2023-12 |    2774000 |
| 2023-07 |    2568000 |

outcome: September, December are the months which will require major restocking of product and increased staffs.
