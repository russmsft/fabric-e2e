# Scoring Rubric - Microsoft Fabric E2E Hackathon

## Overview

Teams are scored on a 120-point scale: 110 base points across 6 challenges plus 10 bonus points.

## Challenge Scores

### Challenge 00: Setting Up Your Fabric Workspace (10 points)

| Criterion | Points | Full Marks | Partial Marks |
|-----------|--------|------------|---------------|
| Workspace created and assigned to capacity | 3 | Workspace visible in Fabric portal, capacity assigned | Workspace exists but capacity not confirmed |
| Lakehouse created | 3 | Lakehouse named wwi_lakehouse (or similar) exists | Lakehouse exists with unclear naming |
| Sample data loaded | 4 | All 5 Parquet files visible in Files/raw/ | Some files present, or files in wrong location |

### Challenge 01: Building the Medallion Lakehouse (20 points)

| Criterion | Points | Full Marks | Partial Marks |
|-----------|--------|------------|---------------|
| Notebook exists with lakehouse attached | 3 | Notebook in workspace, wwi_lakehouse pinned as default | Notebook exists but lakehouse not attached |
| All 5 Silver Delta tables created | 5 | 5 tables with silver_ prefix visible in Tables/ | 3-4 tables created, or missing naming convention |
| Data transformations applied | 5 | Column renaming, type casting, null handling, dedup visible in code | Only 1-2 transformation types applied |
| Computed column on fact table | 3 | At least one derived column (e.g., total_including_tax) | Column exists but calculation is incorrect |
| Tables queryable via SQL endpoint | 4 | SELECT returns data with correct column names and types | Tables appear but columns have issues |

### Challenge 02: Orchestrating with Data Factory (15 points)

| Criterion | Points | Full Marks | Partial Marks |
|-----------|--------|------------|---------------|
| Pipeline exists with activities | 3 | Pipeline with 4+ activities visible on canvas | Pipeline exists with fewer than 4 activities |
| Notebook activity configured | 3 | Points to Challenge 01 notebook, runs successfully | Activity exists but notebook not selected |
| Parameters defined and passed | 3 | At least 1 parameter defined and passed to notebook | Parameter defined but not passed through |
| Error handling (Fail activity) | 3 | Fail activity on failure branch with custom message | Fail activity exists but no custom message |
| Schedule configured | 3 | Schedule visible in Schedule tab with frequency set | No schedule configured |

### Challenge 03: Data Warehouse and Cross-Database Analytics (20 points)

| Criterion | Points | Full Marks | Partial Marks |
|-----------|--------|------------|---------------|
| Warehouse exists | 3 | Warehouse item visible in workspace | - |
| Star schema created | 5 | 4 dimension tables + 1 fact table with appropriate columns | Some tables missing or schema is flat (no star) |
| Data loaded via cross-database queries | 4 | Tables populated using three-part naming from lakehouse | Data loaded but not via cross-database method |
| MERGE stored procedure | 5 | Stored procedure uses MERGE, runs idempotently (2nd run no errors) | Procedure exists but uses INSERT only, or fails on re-run |
| Analytical query | 3 | At least one query joining fact and dimension data with aggregation | Query exists but is a simple SELECT without joins |

### Challenge 04: Real-Time Intelligence Pipeline (20 points)

| Criterion | Points | Full Marks | Partial Marks |
|-----------|--------|------------|---------------|
| Eventhouse + Eventstream connected | 4 | Eventhouse exists, Eventstream routes sample data to KQL table, data flowing | Eventhouse exists but no data flowing |
| KQL queries written | 4 | 3 distinct queries: filter, time-bucketed aggregation, time series average | 1-2 queries only |
| Materialized view created | 4 | Materialized view exists and returns aggregated results | View definition attempted but query errors |
| Dashboard with 3 tiles | 4 | Time chart + categorical breakdown + KPI stat tile, all rendering | Fewer than 3 tiles or tiles showing errors |
| Data Activator rule active | 4 | Rule in Started state with condition and email action | Rule exists but not started, or no action configured |

### Challenge 05: Data Science and Power BI Reporting (25 points)

| Criterion | Points | Full Marks | Partial Marks |
|-----------|--------|------------|---------------|
| ML model trained on Silver data | 5 | Model trained in notebook using scikit-learn on joined Silver tables | Model trained but on trivial or unjoined data |
| MLflow experiment tracking | 4 | Experiment visible with logged params (2+), metrics (1+), model | Experiment exists but missing params or metrics |
| Model registered with signature | 4 | Model in registry, signature visible in artifact details | Model registered but no signature |
| PREDICT batch scoring | 4 | sales_predictions Delta table exists with prediction column | Predictions generated but not written to Delta table |
| Power BI report with 3 visuals | 4 | Bar/column chart, KPI card, detail table all present | Fewer than 3 visuals or visuals not connected to data |
| DAX measure | 4 | At least one custom measure (accuracy, total predicted, etc.) | Measure attempted but formula incorrect |

### Bonus Points (10 points)

| Criterion | Points | Description |
|-----------|--------|-------------|
| Code quality | 3 | Clean, well-organized notebooks with comments. Pipeline activities named descriptively. SQL follows conventions. |
| Creative analytics | 3 | Team went beyond minimum requirements - additional visualizations, creative KQL queries, advanced ML techniques, or insightful analytical findings. |
| Team collaboration | 2 | Evidence of parallel work (Challenge 03/04 split), clear task division, all members contributing. |
| Presentation quality | 2 | Team can clearly explain their solution, walk through the architecture, and articulate design decisions. |

## Score Summary

| Challenge | Max Points |
|-----------|-----------|
| Challenge 00 | 10 |
| Challenge 01 | 20 |
| Challenge 02 | 15 |
| Challenge 03 | 20 |
| Challenge 04 | 20 |
| Challenge 05 | 25 |
| Bonus | 10 |
| **Total** | **120** |

## Scoring Tiers

| Tier | Score Range | Description |
|------|-----------|-------------|
| Gold | 100-120 | Completed all challenges with high quality. Bonus points earned. |
| Silver | 75-99 | Completed most challenges. Minor gaps in quality or completeness. |
| Bronze | 50-74 | Completed setup and core challenges. Struggled with medium/hard. |
| Participant | Below 50 | Made meaningful progress but did not complete core challenges. |
