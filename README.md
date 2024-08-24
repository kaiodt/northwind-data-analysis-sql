# Northwind Database Analysis with SQL

# Overview

The goal of this project is to analyze sales data from the [Northwind database](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/northwind-pubs), a classic dataset representing a fictional company, **Northwind Traders**, which operates in the food and beverage industry on an international scale.

This analysis is conducted using [PostgreSQL](https://www.postgresql.org/), hosted within a [Docker](https://www.docker.com/) environment, to explore key **SQL** features. The goal is to answer critical business questions and extract actionable insights that can inform decision-making processes.

The insights derived from this project can serve as a model for other companies looking to generate meaningful reports and enhance their strategic decisions.

To run this project locally, check out the [Running the Project](#running-the-project) section for instructions.


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
    "Average Revenue" DESC

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

### 7. Which employees generated the highest revenue and handled the most orders in 1997?

To answer this question, we identified the employees who generated the highest total revenue and handled the most orders in the year 1997. Then, we rank the employees based on these metrics to gain insights into the contributions of each one to the company’s overall performance. The results are summarized in the table below.

The results show that **Margaret Peacock** led both in revenue generation and order handling, followed closely by **Janet Leverling** and **Nancy Davolio**. This information is valuable for understanding the top-performing employees and could guide decisions related to employee recognition, resource allocation, and performance improvement strategies.

| Employee         | Revenue    | Share of All Revenue (%) | Rank by Revenue | Orders | Share of All Orders (%) | Rank by Quantity |
|:-----------------|-----------:|-------------------------:|:---------------:|-------:|:-----------------------:|:----------------:|
| Margaret Peacock | 128,809.79 | 20.87                    | 1               | 81     | 19.85                   | 1                |
| Janet Leverling  | 108,026.16 | 17.51                    | 2               | 71     | 17.40                   | 2                |
| Nancy Davolio    | 93,148.08  | 15.09                    | 3               | 55     | 13.48                   | 3                |
| Andrew Fuller    | 70,444.14  | 11.42                    | 4               | 41     | 10.05                   | 5                |
| Robert King      | 60,471.19  | 9.80                     | 5               | 36     | 8.82                    | 6                |
| Laura Callahan   | 56,032.62  | 9.08                     | 6               | 54     | 13.24                   | 4                |
| Michael Suyama   | 43,126.37  | 6.99                     | 7               | 33     | 8.09                    | 7                |
| Steven Buchanan  | 30,716.47  | 4.98                     | 8               | 18     | 4.41                    | 9                |
| Anne Dodsworth   | 26,310.39  | 4.26                     | 9               | 19     | 4.66                    | 8                |

<details>
<summary><b>Full SQL Query</b></summary>

```sql
WITH totals_1997 AS
(
    SELECT
        COUNT(DISTINCT od.order_id) AS total_order_quantity,
        SUM(od.quantity * od.unit_price * (1.0 - od.discount)) AS total_revenue

    FROM
        order_details od
        JOIN orders o ON od.order_id = o.order_id

    WHERE
        EXTRACT(YEAR FROM o.order_date) = 1997
),

orders_by_employees AS
(
    SELECT
        o.employee_id,
        COUNT(DISTINCT od.order_id) AS order_quantity,
        SUM(od.quantity * od.unit_price * (1.0 - od.discount)) AS total_revenue

    FROM
        order_details od
        JOIN orders o ON od.order_id = o.order_id

    WHERE
        EXTRACT(YEAR FROM o.order_date) = 1997

    GROUP BY
        o.employee_id
)

SELECT
    CONCAT(e.first_name, ' ', e.last_name) AS "Employee",
    ROUND(oe.total_revenue::numeric, 2) AS "Revenue",
    ROUND(
        (oe.total_revenue / t.total_revenue * 100)::numeric, 2
    ) AS "Share of All Revenue (%)",
    RANK() OVER (ORDER BY oe.total_revenue DESC) AS "Rank by Revenue",
    oe.order_quantity AS "Orders",
    ROUND(
        oe.order_quantity::numeric / t.total_order_quantity * 100, 2
    ) AS "Share of All Orders (%)",
    RANK() OVER (ORDER BY oe.order_quantity DESC) AS "Rank by Quantity"

FROM
    orders_by_employees oe
    JOIN employees e ON oe.employee_id = e.employee_id
    CROSS JOIN totals_1997 t;
```
</details>

<details>
<summary><b>View Query</b></summary>

```sql
SELECT *
FROM vw_employee_performance_1997;
```
</details>

---

### 8. Which employees have the most consistent performance throughout the year?

To determine which employees maintain consistent performance throughout the year, we analyze the standard deviation of their monthly revenue and the number of orders they handle each month. A lower standard deviation indicates more consistent performance, while a higher value suggests variability. By identifying employees with the lowest standard deviations in both revenue and order count, we can pinpoint those who consistently meet expectations without significant fluctuations. The results are presented in the table below.

The analysis shows that **Michael Suyama** and **Steven Buchanan** exhibit the most consistent performance in terms of revenue and order count. Their standard deviations are significantly lower compared to others, indicating steady and reliable output. This information is valuable for recognizing dependable employees and possibly assigning them to critical roles that require consistent performance.

| Employee         | Average Monthly Revenue | Revenue Std. Dev. | Average Monthly Orders | Orders Std. Dev. |
|:-----------------|------------------------:|------------------:|-----------------------:|-----------------:|
| Michael Suyama   | 3,519.67                | 2,456.20          | 3.19                   | 1.47             |
| Steven Buchanan  | 3,821.79                | 3,289.35          | 2.33                   | 1.28             |
| Anne Dodsworth   | 4,294.89                | 4,865.14          | 2.39                   | 1.54             |
| Laura Callahan   | 5,515.75                | 4,925.09          | 4.52                   | 2.45             |
| Margaret Peacock | 10,125.69               | 5,278.70          | 6.78                   | 3.01             |
| Nancy Davolio    | 8,352.50                | 5,739.65          | 5.35                   | 2.82             |
| Robert King      | 5,931.82                | 6,255.19          | 3.43                   | 2.13             |
| Janet Leverling  | 9,218.77                | 6,926.75          | 5.77                   | 3.12             |
| Andrew Fuller    | 7,240.77                | 7,340.03          | 4.17                   | 3.68             |

<details>
<summary><b>Full SQL Query</b></summary>

```sql
WITH monthly_employee_results AS (
    SELECT
        e.employee_id,
        DATE_TRUNC('month', o.order_date) AS month,
        SUM(od.quantity * od.unit_price * (1.0 - od.discount)) AS monthly_revenue,
        COUNT(DISTINCT o.order_id) AS monthly_order_count

    FROM
        orders o
        JOIN order_details od ON o.order_id = od.order_id
        JOIN employees e ON o.employee_id = e.employee_id

    GROUP BY
        e.employee_id,
        month
),

employee_performance_consistency AS (
    SELECT
        employee_id,
        AVG(monthly_revenue) AS avg_monthly_revenue,
        STDDEV(monthly_revenue) AS revenue_stddev,
        AVG(monthly_order_count) AS avg_monthly_order_count,
        STDDEV(monthly_order_count) AS order_count_stddev

    FROM
        monthly_employee_results
        
    GROUP BY
        employee_id

)

SELECT
    CONCAT(e.first_name, ' ', e.last_name) AS "Employee",
    ROUND(ep.avg_monthly_revenue::numeric, 2) AS "Average Monthly Revenue",
    ROUND(ep.revenue_stddev::numeric, 2) AS "Revenue Std. Dev.",
    ROUND(ep.avg_monthly_order_count, 2) AS "Average Monthly Orders",
    ROUND(ep.order_count_stddev, 2) AS "Orders Std. Dev."

FROM
    employee_performance_consistency ep
    JOIN employees e ON ep.employee_id = e.employee_id

ORDER BY
    revenue_stddev;
```
</details>

<details>
<summary><b>View Query</b></summary>

```sql
SELECT *
FROM vw_employee_performance_consistency;
```
</details>


[⬆️ Back to navigation](#quick-navigation)


# Running the Project

## Pre-Installed PostgreSQL

If you already have a working [PostgreSQL](https://www.postgresql.org/) instance, follow these steps to set up the Northwind database.


### 1. Clone the Repository

Begin by cloning this repository to your local machine:

```bash
git clone https://github.com/kaiodt/northwind-data-analysis-sql.git
```

### 2. Create a New Database (Optional)

If you prefer to create a new database for the Northwind data, you can do so by running the following command in your PostgreSQL query tool or command line:

```sql
CREATE DATABASE northwind;
```

### 3. Execute the SQL Script

Open your PostgreSQL query tool (e.g., pgAdmin, psql) connected to your desired database (either an existing one or the newly created `northwind` database). Then, execute the [`northwind.sql`](northwind.sql) script to create and populate all the necessary tables, views, and other database objects.


## Running PostgreSQL and pgAdmin in Docker

If you prefer to use [Docker](https://www.docker.com/) for setting up [PostgreSQL](https://www.postgresql.org/) and [pgAdmin](https://www.pgadmin.org/), follow these instructions.


### Pre-Requisites

Ensure you have [Docker](https://www.docker.com/) and [Docker Compose](https://docs.docker.com/compose/install/) installed on your system.


### 1. Clone the Repository

Begin by cloning this repository to your local machine:

```bash
git clone https://github.com/kaiodt/northwind-data-analysis-sql.git
```

### 2. Set Up Docker Containers

- Navigate to the project directory:

    ```bash
    cd northwind-data-analysis-sql
    ```

- Use Docker Compose to start PostgreSQL and pgAdmin services:

    ```bash
    docker-compose up -d
    ```

- This will spin up two containers in the detached (`-d`) mode:

    - `postgres-db`: The PostgreSQL database server.

    - `pgadmin`: A web-based PostgreSQL administration tool.


### 3. Access pgAdmin

- Open your browser and go to http://localhost:5050 to access pgAdmin.

    > You may need to wait a few seconds until this URL becomes available after starting the pgAdmin container.

- If accessing pgAdmin for the first time, you might be asked to set a master password.

- Then, log in with the credentials defined in the [docker-compose.yml](docker-compose.yaml) file:

    - **Email Address**: `admin@admin.com`

    - **Password**: `admin`


### 4. Create a New Server in pgAdmin

- After logging into pgAdmin, click the `Add New Server` button in the **Quick Links** section.

- In the **General** tab, choose a name for the server, e.g., `northwind-server`. Leave the other fields unchanged.

- In the **Connection** tab, fill in the following details:

    - **Host name/address**: `postgres`

    - **Port**: `5432`

    - **Maintenance database**: `postgres`

    - **Username**: `postgres`

    - **Password**: `postgres`

- Then, click the `Save` button.

- The newly created server should appear under the `Servers` folder on the **Object Explorer** (left-hand side of the screen).

- Under the new server, you'll find the `northwind` database containing all tables and views used in this project.


### Stopping and Removing Containers

To stop and remove the Docker containers created for the project, use the following command:

```bash
docker compose down
```
This command will stop and remove both the PostgreSQL and pgAdmin containers, as well as the associated network.

Please note that the volumes `postgres_data` and `pgadmin_data` were created to persist the data for both PostgreSQL and pgAdmin, ensuring that your data remains intact even after the containers are removed. If you also wish to remove these volumes and delete the associated data, use the following command:

```bash
docker compose down --volumes
```

This command will stop and remove the containers, the network, and the persistent volumes, resulting in the deletion of all stored data. **Use this with caution, only if you no longer need the data.**


## Running the Business Questions Views

As mentioned in the [Business Questions](#business-questions) section, for each question, a view was created to generate the corresponding answer. Below you can find a table showing the view for each question:

| Question                                                                                                                            | View Name                             |
|:------------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------|
| [Question 1](#1-how-does-the-monthly-revenue-evolve-over-time)                                                                      | `vw_monthly_revenue_analysis`         |
| [Question 2](#2-which-are-the-top-5-months-with-the-highest-average-monthly-revenue)                                                | `vw_top_5_avg_monthly_revenue`        |
| [Question 3](#3-which-are-the-top-5-best-selling-products-by-quantity-and-what-categories-do-they-belong-to)                        | `vw_top_5_selling_products`           |
| [Question 4](#4-which-are-the-top-5-products-generating-the-highest-revenue-and-what-categories-do-they-belong-to)                  | `vw_top_5_revenue_products`           |
| [Question 5](#5-who-are-the-top-5-customers-by-yearly-revenue-share-and-how-does-their-contribution-compare-across-different-years) | `vw_top_5_customers_revenue_by_year`  |
| [Question 6](#6-which-customers-place-orders-most-frequently)                                                                       | `vw_most_frequent_customers`          |
| [Question 7](#7-which-employees-generated-the-highest-revenue-and-handled-the-most-orders-in-1997)                                  | `vw_employee_performance_1997`        |
| [Question 8](#8-which-employees-have-the-most-consistent-performance-throughout-the-year)                                           | `vw_employee_performance_consistency` |


[⬆️ Back to navigation](#quick-navigation)
