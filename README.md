# Northwind Database Analysis with SQL

## Overview

The goal of this project is to analyze sales data from the [Northwind database](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/northwind-pubs), a classic dataset representing a fictional company, **Northwind Traders**, which operates in the food and beverage industry on an international scale.

This analysis is conducted using [PostgreSQL](https://www.postgresql.org/), hosted within a [Docker](https://www.docker.com/) environment, to explore key **SQL** features. The goal is to answer critical business questions and extract actionable insights that can inform decision-making processes.

The insights derived from this project can serve as a model for other companies looking to generate meaningful reports and enhance their strategic decisions.


## Quick Navigation

- [Overview](#overview)

- [About the Database](#about-the-database)

    - [Database Schema](#database-schema)

    - [Entity-Relationship (ER) Diagram](#entity-relationship-er-diagram)

    - [Database Source](#database-source)

- [Business Questions](#business-questions)

- [Running the Project](#running-the-project)


## About the Database

The [Northwind database](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/northwind-pubs) is a classic dataset originally created by Microsoft to showcase the features of their relational database systems. It represents a fictional company, **Northwind Traders**, which imports and exports food and beverage products worldwide.

The structured nature of the Northwind database, along with its realistic representation of business operations, makes it an excellent case study for demonstrating data analysis and SQL concepts. The insights derived from this analysis can be applied to similar datasets, helping businesses enhance their decision-making processes and overall performance.


### Database Schema

The Northwind database contains several tables that capture various aspects of the business, such as customers, orders, products, employees, and suppliers. The key tables and their relationships are outlined below:

- **Customers**: Contains information about Northwind Traders' customers, including company names, contact details, and locations.

- **Orders**: Records details of the orders placed by customers, including the order date, shipping details, and the responsible employee.

- **Order Details**: Provides detailed information about the products included in each order, such as quantity, price, and discounts.

- **Products**: Lists the products sold by Northwind Traders, including product names, unit prices, and stock levels.

- **Suppliers**: Contains information about the suppliers of the products, including company names, contact details, and countries of origin.

- **Employees**: Stores details about Northwind Traders' employees, including their names, job titles, and territories they oversee.

- **Categories**: Categorizes the products into different groups, such as beverages, condiments, and seafood.

- **Shippers**: Records the shipping companies used to deliver orders to customers.


### Entity-Relationship (ER) Diagram

The Entity-Relationship (ER) diagram of the Northwind database illustrates the relationships between the different tables. Below is a visual representation:

<p align="center">
  <img src="imgs/northwind-er-diagram.png" width="80%">
</p>


### Database Source

The Northwind database used in this project is sourced from this [repository](https://github.com/pthom/northwind_psql), which provides a [SQL script](https://github.com/pthom/northwind_psql/blob/master/northwind.sql) to create all tables, relationships, and populate data in a PostgreSQL database management system.

[⬆️ Back to navigation](#quick-navigation)

## Business Questions

TODO

## Running the Project

TODO
