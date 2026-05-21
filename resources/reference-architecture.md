# Reference Architecture - Microsoft Fabric End-to-End Analytics

## Architecture Overview

This hackathon builds a complete analytics platform for Wide World Importers using Microsoft Fabric. The architecture spans six workloads, all connected through OneLake as the unified storage layer. Each challenge adds a new piece to the architecture until the full picture is in place.

```
                     Wide World Importers
                      Parquet Source Data
                             |
                             v
              +------------------------------+
              |     OneLake / Lakehouse       |
              |         (wwi_lakehouse)       |
              |                              |
              |   Files/raw/ ---- Bronze     |
              |       |                      |
              |       v                      |
              |   [Spark Notebook]           |
              |   (Challenge 01)             |
              |       |                      |
              |       v                      |
              |   Tables/ ------ Silver      |
              +------------------------------+
                  |          |            |
                  |          |            |
      +-----------+    +-----+-----+     +-------------------+
      |                |           |                         |
      v                v           v                         v
+------------+  +-----------+  +----------------+   +-----------------+
| Data       |  | Data      |  | ML Notebook    |   | Eventstream     |
| Factory    |  | Warehouse |  | (Challenge 05) |   | (Challenge 04)  |
| Pipeline   |  | Star      |  |                |   |                 |
| (Chall 02) |  | Schema    |  | MLflow Model   |   | Sample Retail   |
|            |  | (Chall 03)|  | PREDICT func   |   | Events          |
+------------+  +-----------+  +----------------+   +-----------------+
      |              |                |                      |
      v              v                v                      v
 Orchestration  Cross-DB         Predictions          +-------------+
 + Scheduling   Queries          (Delta Table)        | Eventhouse  |
                     |                |               | KQL DB      |
                     |                |               | (Chall 04)  |
                     +-------+--------+               +-------------+
                             |                              |
                             v                              v
                     +----------------+            +-----------------+
                     | Power BI       |            | Real-Time       |
                     | Direct Lake    |            | Dashboard       |
                     | Report         |            | + Data          |
                     | (Challenge 05) |            | Activator       |
                     +----------------+            +-----------------+
```

## OneLake - The Unified Storage Layer

OneLake is the single data lake that backs every Fabric workload. When you create a lakehouse, warehouse, or eventhouse, the data lands in OneLake in open formats (Delta Parquet for tables, standard Parquet/CSV for raw files). This means every workload can read from and write to the same storage without copying data between services.

In this hackathon, the `wwi_lakehouse` lakehouse is the central hub. Raw Parquet files land in `Files/raw/`, Spark notebooks promote them to Delta tables under `Tables/`, and the Data Warehouse, ML notebooks, and Power BI all read from those same tables through OneLake shortcuts or cross-database queries.

## Medallion Architecture

The data follows a three-layer medallion pattern:

| Layer  | Location                  | Format        | Purpose                                      | Challenge |
|--------|---------------------------|---------------|----------------------------------------------|-----------|
| Bronze | Lakehouse Files/raw/      | Parquet       | Raw ingestion, no transformation              | 00        |
| Silver | Lakehouse Tables/         | Delta         | Cleaned, typed, deduplicated dimension and fact tables | 01        |
| Gold   | Data Warehouse            | Delta (managed) | Star schema with relationships, aggregation-ready | 03        |

## Workload Integration Map

| Challenge | Workload              | Reads From                  | Writes To                        | Key Technology            |
|-----------|-----------------------|-----------------------------|----------------------------------|---------------------------|
| 00        | Lakehouse             | External Parquet URL        | Files/raw/ (Bronze)              | Python, requests          |
| 01        | Data Engineering      | Files/raw/ (Bronze)         | Tables/ (Silver Delta)           | PySpark, Delta Lake       |
| 02        | Data Factory          | Lakehouse Tables            | Pipeline orchestration           | Data Pipelines, Activities|
| 03        | Data Warehouse        | Lakehouse Tables (Silver)   | Warehouse tables (Gold)          | T-SQL, Cross-DB queries   |
| 04        | Real-Time Intelligence| Eventstream sample source   | Eventhouse KQL DB                | Eventstream, KQL          |
| 05        | Data Science + BI     | Lakehouse + Warehouse tables| MLflow model, Power BI report    | MLflow, PREDICT, Direct Lake |

## Challenge Dependency Graph

```
Challenge 00 (Setup)
      |
      v
Challenge 01 (Lakehouse)
      |
      +-----------+-----------+
      |           |           |
      v           v           v
Challenge 02  Challenge 03  Challenge 04
(Data Factory) (Warehouse)  (Real-Time)
                  |
                  +-----+
                        |
                        v
                  Challenge 05
                  (Data Science + BI)
```

Challenges 03 and 04 are independent of each other and can run in parallel once Challenge 01 is complete. Challenge 05 depends on both Challenge 01 (Silver tables) and Challenge 03 (Gold warehouse tables). Challenge 02 depends only on Challenge 01.

## Technology Stack

| Technology       | Version / Detail                  | Used In        |
|------------------|-----------------------------------|----------------|
| PySpark          | Fabric Spark Runtime              | Challenges 01, 05 |
| Delta Lake       | Open-source table format          | Challenges 01, 03, 05 |
| T-SQL            | Fabric Data Warehouse dialect     | Challenge 03   |
| KQL              | Kusto Query Language              | Challenge 04   |
| Python           | 3.11+                             | Challenges 00, 01, 05 |
| MLflow           | Fabric-integrated tracking        | Challenge 05   |
| scikit-learn     | ML model training                 | Challenge 05   |
| Power BI         | Direct Lake mode                  | Challenge 05   |
| Data Factory     | Fabric-native pipelines           | Challenge 02   |
| Eventstream      | Real-time event ingestion         | Challenge 04   |
| Data Activator   | Alert rules on real-time data     | Challenge 04   |

![alt text](<Fabric Hackathon.png>)