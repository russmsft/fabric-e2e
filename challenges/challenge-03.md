# Challenge 03 - Data Warehouse and Cross-Database Analytics

[< Previous Challenge](challenge-02.md) | [Home](../README.md) | [Next Challenge >](challenge-04.md)

## Introduction

Your lakehouse now holds clean Silver-layer Delta tables, and the data engineering team is happy. But the business analysts on the Wide World Importers team are not data engineers. They think in star schemas, write T-SQL, and build reports from dimensional models they can reason about. They want proper fact and dimension tables - the kind they have been building in SQL Server for years.

Microsoft Fabric gives you both. A lakehouse for flexible data engineering work and a Data Warehouse for structured, T-SQL-first analytical modeling - side by side in the same workspace. The interesting part is that you do not have to copy data between them in the traditional sense. Cross-database queries let you reach across from one item to the other using standard three-part naming syntax, so your warehouse can pull directly from your lakehouse tables.

In this challenge, you will stand up a Fabric Data Warehouse, build a dimensional star schema inside it, load data from your lakehouse Silver tables using cross-database queries, and package the load logic into a stored procedure so it can run repeatably.

## Description

Your goal is to create a Fabric Data Warehouse with a star schema and populate it from the Silver-layer tables you built in Challenge 01. You will also write analytical queries that join data across the warehouse and lakehouse boundaries.

Here is what you need to accomplish:

**1. Create the Data Warehouse**

Create a new Data Warehouse item in your existing workspace. Name it something clear like `wwi_warehouse`. This is separate from your lakehouse - it is a full T-SQL warehouse with DML support.

**2. Build a Star Schema**

Design and create the following tables in the `dbo` schema of your warehouse:

- **dim_customer** - customer_key (INT), customer_name, category, buying_group, city_key
- **dim_city** - city_key (INT), city_name, state_province, country, sales_territory
- **dim_stock_item** - stock_item_key (INT), stock_item_name, color, selling_package, buying_package
- **dim_date** - date_key (INT), date_value, day_number, month_number, month_name, quarter, year
- **fact_sales** - sale_key (BIGINT), customer_key, city_key, stock_item_key, date_key, quantity, unit_price, tax_rate, total_including_tax, profit

Use appropriate data types for each column. The exact column names may vary slightly depending on what you created in your Silver tables, but the star schema structure must be clear: one fact table surrounded by dimension tables with key relationships.

**3. Load Data Using Cross-Database Queries**

Use `INSERT...SELECT` statements with three-part naming to pull data from your lakehouse Silver tables into your warehouse dimension and fact tables. The three-part naming format is `DatabaseName.SchemaName.TableName`, where the database name is the display name of your lakehouse item.

**4. Create a Stored Procedure for Repeatable Loading**

Write a stored procedure (for example, `dbo.usp_load_fact_sales`) that loads or refreshes the fact_sales table from the lakehouse. Use a `MERGE` statement so the procedure handles both inserts and updates - this way it can run multiple times without duplicating data.

Execute the stored procedure and confirm it works.

**5. Write an Analytical Query**

Write at least one cross-database analytical query that joins your warehouse fact table with either warehouse dimension tables or lakehouse tables. For example, calculate total sales by city and product category, or find the top 10 customers by revenue.

## Success Criteria

- A Data Warehouse item named `wwi_warehouse` (or similar) exists in the workspace.
- The warehouse contains a star schema with at least four dimension tables and one fact table in the `dbo` schema.
- Dimension tables contain data loaded from the lakehouse Silver tables (row counts are non-zero).
- The fact_sales table contains data and has foreign key columns referencing the dimension tables.
- A stored procedure exists in the warehouse that uses a MERGE statement to load fact_sales.
- The stored procedure executes successfully without errors when run a second time (proving idempotent upsert behavior).
- At least one analytical query runs successfully and returns meaningful results joining fact and dimension data.

## Hints

<details>
<summary>Hint 1: Where to create the warehouse</summary>

In your workspace, look for the **+ New item** button. The Warehouse option is under the **Store data** section. Once created, the SQL query editor opens automatically. This is where you will write all your T-SQL.

</details>

<details>
<summary>Hint 2: Connecting to your lakehouse from the warehouse</summary>

Before you can write cross-database queries, you need to make your lakehouse visible in the warehouse SQL editor. Look for a **+ Warehouses** button in the Explorer pane on the left side. Click it and add your lakehouse from the OneLake catalog. Once added, you can expand it and browse its tables in the explorer.

</details>

<details>
<summary>Hint 3: Three-part naming syntax</summary>

The pattern is `ItemDisplayName.SchemaName.TableName`. If your lakehouse is called `wwi_lakehouse` and your Silver table is called `silver_dim_customer`, the three-part reference is:

```sql
wwi_lakehouse.dbo.silver_dim_customer
```

Use this syntax in your `SELECT` and `INSERT...SELECT` statements. Both items must be in the same workspace for this to work.

</details>

<details>
<summary>Hint 4: Creating tables with the right data types</summary>

Fabric Warehouse supports standard T-SQL `CREATE TABLE` syntax. A dimension table might look something like:

```sql
CREATE TABLE dbo.dim_city
(
    city_key        INT NOT NULL,
    city_name       VARCHAR(100),
    state_province  VARCHAR(100),
    country         VARCHAR(100),
    sales_territory VARCHAR(100)
);
```

Check your Silver layer column names and types before writing your CREATE TABLE statements.

</details>

<details>
<summary>Hint 5: Structuring the MERGE statement</summary>

A MERGE needs a target table, a source query, a join condition, and then clauses for what to do when rows match versus when they do not. The general shape is:

```sql
MERGE dbo.fact_sales AS target
USING (
    SELECT sale_key, customer_key, ...
    FROM wwi_lakehouse.dbo.silver_fact_sale
) AS source
ON target.sale_key = source.sale_key
WHEN MATCHED THEN
    UPDATE SET target.quantity = source.quantity, ...
WHEN NOT MATCHED THEN
    INSERT (sale_key, customer_key, ...)
    VALUES (source.sale_key, source.customer_key, ...);
```

Wrap this in a `CREATE PROCEDURE` block and finish with `END; GO`.

</details>

<details>
<summary>Hint 6: Remember - the lakehouse SQL endpoint is read-only</summary>

You cannot INSERT, UPDATE, or DELETE through the lakehouse SQL analytics endpoint. If you get permission errors, make sure you are running your DML statements against your **warehouse** tables, not the lakehouse. The warehouse is the only item that supports full DML operations. Cross-database reads from the lakehouse are fine, but all writes must target the warehouse.

</details>

## Learning Resources

- [Create a Warehouse in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/data-warehouse/create-warehouse)
- [T-SQL surface area in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/data-warehouse/tsql-surface-area)
- [Query across warehouses and lakehouses](https://learn.microsoft.com/en-us/fabric/data-warehouse/query-warehouse)
- [Transform data with stored procedures](https://learn.microsoft.com/en-us/fabric/data-warehouse/tutorial-transform-data)
- [Data warehousing in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/data-warehouse/data-warehousing)
- [Ingest data into the Warehouse](https://learn.microsoft.com/en-us/fabric/data-warehouse/ingest-data)
