# Challenge 01 - Building the Medallion Lakehouse

[< Previous Challenge](challenge-00.md) | [Home](../README.md) | [Next Challenge >](challenge-02.md)

## Introduction

Your team just finished loading the Wide World Importers sample data into the lakehouse. Five Parquet files sitting in Files/raw/ - cities, customers, dates, stock items, and sales. The data is there, but nobody outside of a Spark session can see it. Your Power BI developers are asking where the tables are. Your SQL analysts want to write queries. And all they see in the SQL analytics endpoint is nothing.

That is because files in the Files/ section of a Fabric lakehouse are just files. They do not show up as tables. They are invisible to the SQL analytics endpoint, invisible to cross-database queries from the warehouse, and invisible to Power BI Direct Lake. To make data queryable across Fabric, you need to write it as Delta tables in the Tables/ section.

This is the core idea behind the medallion architecture. Raw data lands in a "bronze" zone (your Parquet files in Files/raw/). You read it, clean it, fix types, drop junk rows, and write the result as managed Delta tables - your "silver" layer. Later challenges will build a "gold" layer in the data warehouse and use these silver tables for machine learning. But right now, the job is straightforward: read Parquet, transform it, write Delta.

You will do this with a PySpark notebook directly in the Fabric browser. No local tooling required.

## Description

Create a new Fabric notebook in your workspace and attach your **wwi_lakehouse** as the default lakehouse. In this notebook, write PySpark code that reads each of the five Parquet files from Files/raw/, applies data quality transformations, and writes the results as managed Delta tables.

Your notebook should handle all five datasets:

- **dimension_city**
- **dimension_customer**
- **dimension_date**
- **dimension_stock_item**
- **fact_sale**

For each dataset, your transformations must address these data quality concerns:

1. **Schema cleanup** - Column names in the raw files use spaces and mixed casing (e.g., "Invoice Date Key", "Unit Price"). Rename them to a consistent snake_case convention.

2. **Type casting** - Date columns arrive as strings. Numeric columns that should be integers or decimals may be stored as longs or doubles. Cast columns to their correct types explicitly.

3. **Null handling** - Decide what to do with null values. For dimension tables, you may want to fill nulls with sensible defaults. For the fact table, rows missing a customer key are not useful - drop them.

4. **Duplicate removal** - Check for and remove duplicate rows based on the natural key of each table.

5. **Computed columns** - For the fact_sale table, add at least one derived column. A good candidate: a total amount that accounts for quantity, unit price, and tax rate.

Write each cleaned DataFrame as a managed Delta table using a "silver_" prefix naming convention (e.g., silver_fact_sale, silver_dim_city). Use `overwrite` mode so the notebook is re-runnable.

After the notebook finishes, verify two things:

- The tables appear in the lakehouse explorer under the Tables/ section
- The tables are queryable through the SQL analytics endpoint using T-SQL SELECT statements

## Success Criteria

- A Spark notebook exists in the workspace with wwi_lakehouse attached as the default lakehouse.
- The notebook reads all five Parquet files from Files/raw/ using PySpark.
- At least one transformation per dataset addresses column renaming, type casting, null handling, or duplicate removal.
- The fact_sale transformation includes at least one computed column (e.g., a total amount calculation).
- Five managed Delta tables with a "silver_" prefix are visible in the lakehouse explorer under Tables/.
- Running `SELECT TOP 10 *` against any silver table in the SQL analytics endpoint returns data with the expected cleaned column names and types.
- The notebook can be re-run without errors (idempotent writes using overwrite mode).

## Hints

<details>
<summary>Hint 1: Where do I start with the notebook?</summary>

From your lakehouse view, look for the "Open notebook" option in the toolbar. When you create a new notebook this way, the lakehouse is automatically pinned as the default. That means file paths like `Files/raw/dimension_city.parquet` resolve without needing the full abfss:// URI. If you created the notebook from the workspace view instead, you will need to add the lakehouse manually using the lakehouse explorer panel on the left side of the notebook.

</details>

<details>
<summary>Hint 2: Reading the Parquet files</summary>

Use `spark.read.format("parquet").load(...)` with a relative path that starts with `Files/`. Each raw file is a folder containing Parquet part files, or a single Parquet file - either way, Spark handles it the same way. Try reading one file first and call `.printSchema()` and `.show(5)` to see what you are working with before writing any transformations.

</details>

<details>
<summary>Hint 3: Column renaming to snake_case</summary>

You can rename columns one at a time with `.withColumnRenamed()`, but that gets tedious with many columns. A more practical approach: loop through `df.columns`, convert each name to snake_case using Python string methods (`.lower().replace(" ", "_")`), and apply all the renames at once. PySpark's `toDF()` method accepts a list of new column names in order.

</details>

<details>
<summary>Hint 4: Writing managed Delta tables</summary>

The key method is `.write.mode("overwrite").format("delta").saveAsTable("silver_fact_sale")`. Using `saveAsTable()` instead of `.save()` with a path is important - it registers the table in the Fabric metastore so it appears in the lakehouse explorer and the SQL analytics endpoint. If you use `.save()` with a file path, the data lands in Files/ and is not visible as a table.

</details>

<details>
<summary>Hint 5: Tables not showing up in the SQL analytics endpoint</summary>

After your notebook finishes, give it a moment. The SQL analytics endpoint metadata syncs automatically but not instantly - it can take 15-30 seconds. Click the refresh button in the lakehouse explorer. If tables still do not appear, check that you used `saveAsTable()` and not `.save()`. Also confirm you did not accidentally write to a path under Files/ by specifying a location string. Let `saveAsTable()` handle the storage location for you.

</details>

## Learning Resources

- [Explore data in your lakehouse with a notebook](https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-notebook-explore)
- [Create and manage Fabric notebooks](https://learn.microsoft.com/en-us/fabric/data-engineering/author-execute-notebook)
- [Delta Lake tables in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-and-delta-tables)
- [SQL analytics endpoint of the lakehouse](https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-sql-analytics-endpoint)
- [Medallion lakehouse architecture in Fabric](https://learn.microsoft.com/en-us/fabric/onelake/onelake-medallion-lakehouse-architecture)
