# Challenge 00 - Setting Up Your Fabric Workspace

[< Home](../README.md) | [Challenge 01 >](challenge-01.md)

## Introduction

Welcome to the Microsoft Fabric End-to-End Data Analytics hackathon. Over the next four hours your team will build a complete retail analytics platform for Wide World Importers, a fictional global wholesale and distribution company. You will work across every major Fabric workload: Lakehouse, Data Factory, Data Warehouse, Real-Time Intelligence, Data Science, and Power BI.

Here is the road ahead:

- **Challenge 00 (this one)** - Stand up your Fabric workspace and load the raw data.
- **Challenge 01** - Build a medallion lakehouse with Bronze and Silver layers using Spark notebooks.
- **Challenge 02** - Orchestrate the data pipeline with Data Factory.
- **Challenge 03** - Create a star-schema Data Warehouse and run cross-database queries.
- **Challenge 04** - Wire up a real-time intelligence pipeline with Eventstream, Eventhouse, and KQL.
- **Challenge 05** - Train a machine learning model with MLflow and build a Power BI report using Direct Lake mode.

This first challenge is straightforward. You will set up the foundational workspace and load the sample data that drives every subsequent challenge. Take your time here to make sure everything is solid before moving on.

## Description

Your team needs to accomplish the following:

1. **Confirm your identity and license.** Sign in to [app.fabric.microsoft.com](https://app.fabric.microsoft.com) with an organizational account (Entra ID). Personal Microsoft accounts (outlook.com, hotmail.com) will not work. If your organization does not have a Fabric capacity, activate the free 60-day Fabric Trial from your profile menu.

2. **Create a new Fabric workspace.** Name it something your team can identify easily (for example, `fabric-e2e-team1`). Under the Advanced settings, assign the workspace to your Fabric capacity, whether that is a Trial capacity or a paid F-SKU.

3. **Create a lakehouse.** Inside your new workspace, create a Lakehouse item named `wwi_lakehouse`. This lakehouse is the foundation for Challenges 01 through 05.

4. **Load the Wide World Importers sample data.** Create a new notebook in the workspace and attach `wwi_lakehouse` as the default lakehouse using the explorer panel on the left. Paste and run the following code to download the Parquet source files into the lakehouse Files section:

   ```python
   import requests
   import os

   base_url = "https://azuresynapsestorage.blob.core.windows.net/sampledata/WideWorldImportersDW/tables/"
   tables = ["dimension_city", "dimension_customer", "dimension_date", "dimension_stock_item", "fact_sale"]

   for table in tables:
       url = f"{base_url}{table}.parquet"
       local_path = f"/lakehouse/default/Files/raw/{table}.parquet"
       os.makedirs(os.path.dirname(local_path), exist_ok=True)
       response = requests.get(url)
       with open(local_path, "wb") as f:
           f.write(response.content)
       print(f"Downloaded {table}")
   ```

   This places five Parquet files into the `Files/raw/` directory of your lakehouse. These files represent the Bronze layer of the medallion architecture you will build in Challenge 01.

5. **Verify the data loaded correctly.** Navigate back to the lakehouse explorer view. Click the refresh icon and expand **Files > raw**. You should see five Parquet files: `dimension_city.parquet`, `dimension_customer.parquet`, `dimension_date.parquet`, `dimension_stock_item.parquet`, and `fact_sale.parquet`.

6. **Confirm the SQL analytics endpoint.** In the lakehouse view, use the mode selector in the top-right corner to switch to the **SQL analytics endpoint**. The endpoint should load successfully, though no tables will appear yet because the data sits in the Files area, not the managed Tables area. You will promote data to Delta tables in Challenge 01.

## Success Criteria

- A Fabric workspace exists and is assigned to a Fabric-enabled capacity (Trial or F-SKU).
- A lakehouse named `wwi_lakehouse` exists in the workspace.
- The `Files/raw/` folder in the lakehouse contains five Parquet files: `dimension_city`, `dimension_customer`, `dimension_date`, `dimension_stock_item`, and `fact_sale`.
- The SQL analytics endpoint is accessible from the lakehouse mode selector.

## Hints

<details>
<summary>Hint 1: Getting into Fabric</summary>

Sign in to [app.fabric.microsoft.com](https://app.fabric.microsoft.com). If you see a banner that says "Get started with Microsoft Fabric" or the homepage loads normally, you are in. If you do not have a Fabric capacity assigned, click your profile icon in the top-right corner and select **Start trial** to activate a 60-day free trial.

</details>

<details>
<summary>Hint 2: Creating the workspace</summary>

Click **Workspaces** in the left navigation bar, then click **+ New workspace**. Give it a descriptive name. Expand the **Advanced** section and under **License mode**, select your Fabric capacity. If you activated a trial, it will appear as "Trial". Click **Apply**.

</details>

<details>
<summary>Hint 3: Creating the lakehouse</summary>

Inside your workspace, click **+ New item**. In the item gallery, search for or scroll to **Lakehouse**. Click it, name it `wwi_lakehouse`, and click **Create**. The lakehouse explorer will open showing an empty Files and Tables section.

</details>

<details>
<summary>Hint 4: Loading data via notebook</summary>

Go back to your workspace and click **+ New item > Notebook**. When the notebook opens, look at the left panel. If your lakehouse is not attached, click **Add** under the Lakehouses section and select `wwi_lakehouse`. Once it is attached, paste the download code into the first cell and click the **Run** button (or press Shift+Enter). Wait for all five tables to print "Downloaded".

</details>

<details>
<summary>Hint 5: Verifying the data</summary>

After the notebook finishes, click the lakehouse name in the left explorer panel to open it in lakehouse view. Click the circular **refresh** icon at the top of the explorer. Expand **Files > raw**. You should see five `.parquet` files listed. If the `raw` folder is missing, re-run the notebook and check for error messages in the cell output.

</details>

## Learning Resources

- [Create a workspace in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/get-started/create-workspaces)
- [Create a lakehouse in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/data-engineering/create-lakehouse)
- [Microsoft Fabric trial](https://learn.microsoft.com/en-us/fabric/get-started/fabric-trial)
- [Lakehouse end-to-end tutorial](https://learn.microsoft.com/en-us/fabric/data-engineering/tutorial-lakehouse-introduction)
- [Load data into a lakehouse](https://learn.microsoft.com/en-us/fabric/data-engineering/load-data-lakehouse)
