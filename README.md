# Northwind Database Analysis with SQL

# Overview

The goal of this project is to analyze sales data from the [Northwind database](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/northwind-pubs), a classic dataset representing a fictional company, **Northwind Traders**, which operates in the food and beverage industry on an international scale.

This analysis is conducted using [PostgreSQL](https://www.postgresql.org/), hosted within a [Docker](https://www.docker.com/) environment, to explore key **SQL** features. The goal is to answer critical business questions and extract actionable insights that can inform decision-making processes.

The insights derived from this project can serve as a model for other companies looking to generate meaningful reports and enhance their strategic decisions.


# Quick Navigation

- [Overview](#overview)

- [About the Database](#about-the-database)

    - [Database Schema](#database-schema)

    - [Entity-Relationship (ER) Diagram](#entity-relationship-er-diagram)

    - [Database Source](#database-source)

- [Business Questions](#business-questions)

    - [Revenue Analysis](#revenue-analysis)

    - [Products Analysis](#products-analysis)

    - [Customers Analysis](#customers-analysis)

    - [Employees Analysis](#employees-analysis)

- [Running the Project](#running-the-project)


# About the Database

The [Northwind database](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/northwind-pubs) is a classic dataset originally created by Microsoft to showcase the features of their relational database systems. It represents a fictional company, **Northwind Traders**, which imports and exports food and beverage products worldwide.

The structured nature of the Northwind database, along with its realistic representation of business operations, makes it an excellent case study for demonstrating data analysis and SQL concepts. The insights derived from this analysis can be applied to similar datasets, helping businesses enhance their decision-making processes and overall performance.


## Database Schema

The Northwind database contains several tables that capture various aspects of the business, such as customers, orders, products, employees, and suppliers. The key tables and their relationships are outlined below:

- **Customers**: Contains information about Northwind Traders' customers, including company names, contact details, and locations.

- **Orders**: Records details of the orders placed by customers, including the order date, shipping details, and the responsible employee.

- **Order Details**: Provides detailed information about the products included in each order, such as quantity, price, and discounts.

- **Products**: Lists the products sold by Northwind Traders, including product names, unit prices, and stock levels.

- **Suppliers**: Contains information about the suppliers of the products, including company names, contact details, and countries of origin.

- **Employees**: Stores details about Northwind Traders' employees, including their names, job titles, and territories they oversee.

- **Categories**: Categorizes the products into different groups, such as beverages, condiments, and seafood.

- **Shippers**: Records the shipping companies used to deliver orders to customers.


## Entity-Relationship (ER) Diagram

The Entity-Relationship (ER) diagram of the Northwind database illustrates the relationships between the different tables. Below is a visual representation:

<p align="center">
  <img src="imgs/northwind-er-diagram.png" width="80%">
</p>


## Database Source

The Northwind database used in this project is sourced from this [repository](https://github.com/pthom/northwind_psql), which provides a [SQL script](https://github.com/pthom/northwind_psql/blob/master/northwind.sql) to create all tables, relationships, and populate data in a PostgreSQL database management system.

[⬆️ Back to navigation](#quick-navigation)


# Business Questions

This section presents a series of key business questions designed to analyze the data in the Northwind database and generate insights that are valuable for informed decision-making.

The analysis is conducted across several business dimensions, including **revenue**, **products**, **customers**, and **employees**. Each dimension includes specific business questions, followed by their answers, the complete SQL queries used to derive these answers, and an alternative SQL command that queries a pre-created view for each question.


## Revenue Analysis

### 1. How does the monthly revenue evolve over time?

To answer this question, we calculate the monthly revenue along with the percentage variation from the previous month. Additionally, we compute the cumulative Year-to-Date (YTD) revenue. A sample of the results is presented in the table below.

| Year | Month | Revenue   | Variation (%) | YTD Revenue |
|:----:|:-----:|----------:|--------------:|------------:|
| 1996 | 7     | 27,861.90 | -             | 27,861.90   |
| 1996 | 8     | 25,485.28 | -8.53         | 53,347.18   |
| ...  | ...   | ...       | ...           | ...         |
| 1996 | 12    | 45,239.63 | -0.79         | 208,083.98  |
| 1997 | 1     | 61,258.07 | 35.41         | 61,258.07   |
| ...  | ...   | ...       | ...           | ...         |
| 1997 | 12    | 71,398.43 | 64.01         | 617,085.20  |
| 1998 | 1     | 94,222.11 | 31.97         | 94,222.11   |
| ...  | ...   | ...       | ...           | ...         |
| 1998 | 5     | 18,333.63 | -85.19        | 440,623.87  |

<details>
<summary><b>Full SQL Query</b></summary>

```sql
WITH monthly_revenue AS
(
    SELECT
        EXTRACT(YEAR FROM o.order_date) AS year,
        EXTRACT(MONTH FROM o.order_date) AS month,
        ROUND(
            SUM(od.quantity * od.unit_price * (1.0 - od.discount))::numeric, 2
        ) AS revenue

    FROM
        order_details od
        JOIN orders o ON od.order_id = o.order_id

    GROUP BY
        year,
        month
)

SELECT
    year AS "Year",
    month AS "Month",
    revenue AS "Revenue",
    ROUND(
        (revenue - LAG(revenue) OVER (ORDER BY year, month)) /
        LAG(revenue) OVER (ORDER BY year, month) * 100, 2
    ) AS "Variation (%)",
    SUM(revenue) OVER (
        PARTITION BY year
        ORDER BY month
    ) AS "YTD Revenue"

FROM
    monthly_revenue;
```
</details>

<details>
<summary><b>View Query</b></summary>

```sql
SELECT *
FROM vw_monthly_revenue_analysis;
```
</details>

---

### 2. Which are the top 5 months with the highest average monthly revenue?

To accurately answer this question, we first compute the total revenue for each month across all years. Then, we calculate the average revenue for each month and sort the results to identify the top 5 months with the highest average revenue. The results are displayed in the table below.

The results suggest that Northwind Traders experiences its highest average revenues during the early months of the year and around the holiday season, pointing to potential opportunities for targeted marketing and inventory planning during these periods.

| Month | Average Revenue |
|:-----:|----------------:|
| 4     | 88,415.82       |
| 1     | 77,740.09       |
| 3     | 71,700.69       |
| 2     | 68,949.46       |
| 12    | 58,319.03       |

<details>
<summary><b>Full SQL Query</b></summary>

```sql
WITH monthly_revenue AS
(
    SELECT
        EXTRACT(YEAR FROM o.order_date) AS year,
        EXTRACT(MONTH FROM o.order_date) AS month,
        ROUND(
            SUM(od.quantity * od.unit_price * (1.0 - od.discount))::numeric, 2
        ) AS revenue

    FROM
        order_details od
        JOIN orders o ON od.order_id = o.order_id

    GROUP BY
        year,
        month
)

SELECT
    month AS "Month",
       ROUND(AVG(revenue), 2) AS "Average Revenue"

FROM
    monthly_revenue

GROUP BY
    month

ORDER BY
    avg_revenue DESC

LIMIT 5;
```
</details>

<details>
<summary><b>View Query</b></summary>

```sql
SELECT *
FROM vw_top_5_avg_monthly_revenue;
```
</details>


## Products Analysis

### 3. Which are the top 5 best-selling products by quantity, and what categories do they belong to?

To answer this question, we first identify the top 5 products with the highest total quantity sold across all orders. Next, we categorize these products to understand which types of goods are driving the most sales. The results are summarized in the table below.

The analysis reveals that **Dairy Products**, particularly cheese items, dominate the top 5 best-sellers, occupying the first three positions. **Grains/Cereals** and **Confections** also exhibit significant customer demand, though not as strong as dairy products. Overall, the best-selling items fall within the food category. This insight can inform inventory management, marketing strategies, and promotional efforts, with a particular focus on the dairy category.

| Product                 | Category        | Total Quantity Sold |
|:------------------------|:----------------|--------------------:|
| Camembert Pierrot       | Dairy Products  | 1,577               |
| Raclette Courdavault    | Dairy Products  | 1,496               |
| Gorgonzola Telino       | Dairy Products  | 1,397               |
| Gnocchi di nonna Alice  | Grains/Cereals  | 1,263               |
| Pavlova                 | Confections     | 1,158               |

<details>
<summary><b>Full SQL Query</b></summary>

```sql
WITH top_5_products_sales AS
(
    SELECT
        product_id,
        SUM(quantity) AS total_quantity

    FROM
        order_details

    GROUP BY
        product_id

    ORDER BY
        total_quantity DESC

    LIMIT 5
)

SELECT
    p.product_name AS "Product",
    c.category_name AS "Category",
    t.total_quantity AS "Total Quantity Sold"

FROM
    top_5_products_sales t
    JOIN products p ON t.product_id = p.product_id
    JOIN categories c ON p.category_id = c.category_id

ORDER BY
    t.total_quantity DESC;
```
</details>

<details>
<summary><b>View Query</b></summary>

```sql
SELECT *
FROM vw_top_5_selling_products;
```
</details>

---

### 4. Which are the top 5 products generating the highest revenue, and what categories do they belong to?

To answer this question, we first identify the top 5 products with the highest total revenue and categorize them accordingly. We then examine the total quantity sold and the unit price of these products to gain further insights into their performance. The results are presented in the table below.

The results show that a **Beverages** product leads in revenue generation, primarily due to its high unit price. A **Meat/Poultry** product follows in second place, also driven by a relatively high unit price. Despite their lower unit prices, **Dairy Products** appear in the top 5 due to their high sales volumes. **Confections** also contribute significantly to overall revenue. These findings can inform strategies for pricing, marketing, and inventory management, particularly focusing on high-revenue products.

| Product                 | Category       | Total Revenue | Total Quantity Sold | Unit Price |
|:------------------------|:---------------|--------------:|--------------------:|-----------:|
| Côte de Blaye           | Beverages      | 141,396.74    | 623                 | 263.50     |
| Thüringer Rostbratwurst | Meat/Poultry   | 80,368.67     | 746                 | 123.79     |
| Raclette Courdavault    | Dairy Products | 71,155.70     | 1,496               | 55.00      |
| Tarte au sucre          | Confections    | 47,234.97     | 1,083               | 49.30      |
| Camembert Pierrot       | Dairy Products | 46,825.48     | 1,577               | 34.00      |

<details>
<summary><b>Full SQL Query</b></summary>

```sql
WITH top_5_products_revenue AS
(
    SELECT
        product_id,
        SUM(quantity * unit_price * (1.0 - discount)) AS total_revenue,
        SUM(quantity) AS total_quantity

    FROM
        order_details

    GROUP BY
        product_id

    ORDER BY
        total_revenue DESC

    LIMIT 5
)

SELECT
    p.product_name AS "Product",
    c.category_name AS "Category",
    ROUND(t.total_revenue::numeric, 2) AS "Total Revenue",
    t.total_quantity AS "Total Quantity Sold",
    ROUND(p.unit_price::numeric, 2) AS "Unit Price"

FROM
    top_5_products_revenue t
    JOIN products p ON t.product_id = p.product_id
    JOIN categories c ON p.category_id = c.category_id

ORDER BY
    t.total_revenue DESC;
```
</details>

<details>
<summary><b>View Query</b></summary>

```sql
SELECT *
FROM vw_top_5_revenue_products;
```
</details>


## Customers Analysis

### 5. Who are the top 5 customers by yearly revenue share, and how does their contribution compare across different years?

To answer this question, we identify the top 5 customers with the highest total revenue in each year. We then calculate each customer’s contribution as a percentage of the total yearly revenue to understand their relative importance to the business over time. The results are displayed in the table below.

The analysis reveals that certain customers, such as **Ernst Handel**, **QUICK-Stop**, and **Save-a-lot Markets**, consistently appear among the top contributors across multiple years. These customers account for a significant portion of Northwind's yearly revenue, emphasizing their importance. This information can help prioritize customer relationship management and tailor services to maintain and potentially increase their business.

| Year | Customer                      | Yearly Revenue | Total Year Revenue Share (%) |
|:----:|:------------------------------|---------------:|-----------------------------:|
| 1996 | Ernst Handel                  | 15,568.07      | 7.48                         |
| 1996 | QUICK-Stop                    | 11,950.08      | 5.74                         |
| 1996 | Rattlesnake Canyon Grocery    | 10,475.78      | 5.03                         |
| 1996 | Save-a-lot Markets            | 10,338.26      | 4.97                         |
| 1996 | Piccolo und mehr              | 10,033.28      | 4.82                         |
| 1997 | QUICK-Stop                    | 61,109.91      | 9.90                         |
| 1997 | Save-a-lot Markets            | 57,713.57      | 9.35                         |
| 1997 | Ernst Handel                  | 48,096.26      | 7.79                         |
| 1997 | Mère Paillarde                | 23,332.31      | 3.78                         |
| 1997 | Hungry Owl All-Night Grocers  | 20,454.40      | 3.31                         |
| 1998 | Ernst Handel                  | 41,210.65      | 9.35                         |
| 1998 | QUICK-Stop                    | 37,217.31      | 8.45                         |
| 1998 | Save-a-lot Markets            | 36,310.11      | 8.24                         |
| 1998 | Hanari Carnes                 | 23,821.20      | 5.41                         |
| 1998 | Rattlesnake Canyon Grocery    | 21,238.27      | 4.82                         |

<details>
<summary><b>Full SQL Query</b></summary>

```sql
WITH yearly_ranked_customers AS
(
    SELECT
        EXTRACT(YEAR FROM o.order_date) AS year,
        o.customer_id,
        SUM(od.quantity * od.unit_price * (1.0 - od.discount)) AS total_customer_revenue,
        RANK() OVER (
            PARTITION BY EXTRACT(YEAR FROM o.order_date)
            ORDER BY SUM(od.quantity * od.unit_price * (1.0 - od.discount)) DESC
        ) AS rank_revenue,
        SUM(SUM(od.quantity * od.unit_price * (1.0 - od.discount))) OVER (
            PARTITION BY EXTRACT(YEAR FROM o.order_date)
        ) AS total_year_revenue

    FROM
        order_details od
        JOIN orders o ON od.order_id = o.order_id

    GROUP BY
        year,
        o.customer_id
)

SELECT
    yrc.year AS "Year",
    c.company_name AS "Customer",
    ROUND(yrc.total_customer_revenue::numeric, 2) AS "Yearly Revenue",
    ROUND(
        (yrc.total_customer_revenue / yrc.total_year_revenue * 100)::numeric, 2
    ) AS "Total Year Revenue Share (%)"

FROM
    yearly_ranked_customers yrc
    JOIN customers c ON yrc.customer_id = c.customer_id

WHERE
    yrc.rank_revenue <= 5

ORDER BY
    yrc.year,
    yrc.rank_revenue;
```
</details>

<details>
<summary><b>View Query</b></summary>

```sql
SELECT *
FROM vw_top_5_customers_revenue_by_year;
```
</details>

---

### 6. Which customers place orders most frequently?

To answer this question, we calculate the average interval between consecutive orders for each customer, dividing them into quartiles based on their ordering frequency. Customers in the first and second quartiles are those who place orders most frequently. By analyzing these customers, we can identify patterns and potential opportunities for targeted marketing or loyalty programs aimed at increasing retention and order frequency. A sample of the results is presented in the table below.

The analysis indicates that customers in the first quartile have average order intervals ranging from 18 to 45 days. These customers may have more consistent purchasing behaviors and represent valuable opportunities for ongoing engagement. Customers in the second quartile, with intervals between 46 and 66 days, also exhibit frequent purchasing patterns, though slightly less so than the first quartile. Focusing on these groups can help tailor marketing strategies to further encourage frequent purchases.

| Customer                          | Avg. Order Interval (Days) | Quartile |
|:----------------------------------|:--------------------------:|:--------:|
| La corne d'abondance              | 18                         | 1        |
| Save-a-lot Markets                | 19                         | 1        |
| ...                               | ...                        | ...      |
| Wartian Herkku                    | 45                         | 1        |
| Lehmanns Marktstand               | 45                         | 1        |
| Alfreds Futterkiste               | 46                         | 2        |
| LILA-Supermercado                 | 49                         | 2        |
| ...                               | ...                        | ...      |
| Magazzini Alimentari Riuniti      | 66                         | 2        |
| Godos Cocina Típica               | 66                         | 2        |

<details>
<summary><b>Full SQL Query</b></summary>

```sql
WITH customer_order_intervals AS (
    SELECT
        customer_id,
        order_date - LAG(order_date) OVER (
            PARTITION BY customer_id
            ORDER BY order_date
        ) AS order_interval -- Time between orders in days

    FROM
        orders
),

customer_avg_order_frequency_quartiles AS (
    SELECT
        customer_id,
        AVG(order_interval) AS avg_order_interval,
        NTILE(4) OVER (ORDER BY AVG(order_interval)) AS quartile

    FROM
        customer_order_intervals

    WHERE
        order_interval IS NOT NULL

    GROUP BY
        customer_id
)

SELECT
    c.company_name AS "Customer",
    CEIL(q.avg_order_interval) AS "Avg. Order Interval (Days)",
    q.quartile AS "Quartile"

FROM
    customer_avg_order_frequency_quartiles q
    JOIN customers c ON q.customer_id = c.customer_id

WHERE
    q.quartile IN (1, 2)

ORDER BY
    q.quartile,
    q.avg_order_interval;
```
</details>

<details>
<summary><b>View Query</b></summary>

```sql
SELECT *
FROM vw_most_frequent_customers;
```
</details>


## Employees Analysis

TODO

[⬆️ Back to navigation](#quick-navigation)

# Running the Project

TODO
