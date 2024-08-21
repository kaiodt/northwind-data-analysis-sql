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

TODO


## Employees Analysis

TODO

[⬆️ Back to navigation](#quick-navigation)

# Running the Project

TODO
