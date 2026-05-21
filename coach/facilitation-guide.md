# Facilitation Guide - Microsoft Fabric E2E Hackathon

## Pre-Event Preparation

### Capacity and Access

- Verify all participants have organizational (Entra ID) accounts. Personal Microsoft accounts (outlook.com, hotmail.com, gmail.com) will not work with Fabric.
- Confirm Fabric capacity is available. Options: Fabric Trial (free, 60-day, F64-equivalent per user), or a paid F-SKU (F2 through F2048). Trial capacity is shared across all workspaces a user creates.
- If using a shared tenant, ensure the Fabric admin has enabled these workloads in the admin portal: Data Engineering, Data Science, Data Warehouse, Real-Time Intelligence, Data Factory, Power BI.
- Test the sample data download URL before the event: `https://azuresynapsestorage.blob.core.windows.net/sampledata/WideWorldImportersDW/tables/dimension_city.parquet` - this should return a downloadable Parquet file.

### Team Composition

- Recommended team size: 3-5 people.
- Ideal mix: at least one person comfortable with Python/PySpark, at least one comfortable with T-SQL, and at least one with Power BI experience.
- Teams with all one skill set will struggle on challenges outside their comfort zone - encourage mixing.

### Time-Saving Options

- Consider pre-creating Fabric workspaces for each team to avoid trial activation and capacity assignment delays during Challenge 00.
- Pre-loading the WWI sample data into a shared lakehouse and creating shortcuts from team lakehouses can save 10-15 minutes.
- Have a backup plan if trial activation fails (some tenants have trials disabled by admin policy).

## Agenda and Timing Guide

| Time Block | Challenge | Coach Focus |
|-----------|-----------|-------------|
| 0:00 - 0:20 | Challenge 00: Setup | Help teams with trial activation and workspace creation. Most common issue: capacity not assigned to workspace. |
| 0:20 - 0:50 | Challenge 01: Lakehouse | Watch for Spark cold start confusion (30-90 sec first run). Biggest blocker: using `.save()` instead of `.saveAsTable()`. |
| 0:50 - 1:20 | Challenge 02: Data Factory | Most teams find this intuitive. Help with expression syntax (`@pipeline().parameters.x`). Remind teams to save before running. |
| 1:20 - 2:05 | Challenge 03 OR 04 | Parallel! Let teams choose or split. Challenge 03 is SQL-heavy, Challenge 04 is KQL-heavy. |
| 2:05 - 3:00 | Challenge 05 (Part A+B) | This is where teams need the most help. MLflow signature is the number one gotcha. Autolog exclusive mode is number two. |
| 3:00 - 3:40 | Challenge 05 (Part C) | Power BI portion. Help with semantic model discovery and DAX syntax. |
| 3:40 - 4:00 | Buffer / Wrap-up | Review solutions, Q&A, showcase team work. |

## Common Blockers by Challenge

### Challenge 00: Setup

| Issue | Solution |
|-------|----------|
| Trial activation fails | Tenant admin may have disabled trials. Check admin portal > Tenant settings > Users can try Microsoft Fabric paid features. Alternative: assign a paid F-SKU capacity. |
| Workspace not showing capacity option | The workspace advanced settings only show capacities the user has access to. Ensure the user is a capacity admin or contributor. |
| Notebook cannot find lakehouse | The lakehouse must be pinned as the default in the notebook explorer panel. Click "Add" under Lakehouses and select wwi_lakehouse. |

### Challenge 01: Medallion Lakehouse

| Issue | Solution |
|-------|----------|
| Spark session takes forever to start | First cold start is 30-90 seconds. This is normal. Subsequent cells run fast. |
| Tables not appearing in SQL endpoint | Team used `.save("path")` instead of `.saveAsTable("name")`. Only `saveAsTable` registers in the metastore. |
| Column names with spaces cause issues | PySpark handles spaces in column names but they cause problems in SQL. Rename to snake_case. |
| "Table already exists" error | The team is not using `.mode("overwrite")`. Add it before `.saveAsTable()`. |

### Challenge 02: Data Factory

| Issue | Solution |
|-------|----------|
| Pipeline won't run | Team forgot to save. Fabric requires explicit save before run. |
| Expression syntax errors | Common mistake: missing `@` prefix or wrong casing. Correct: `@pipeline().parameters.run_date`, not `@Pipeline().Parameters.run_date`. |
| Notebook activity shows "failed" but notebook works manually | Check that the notebook has the lakehouse pinned as default. Pipeline-launched notebooks may not inherit the lakehouse binding. |

### Challenge 03: Data Warehouse

| Issue | Solution |
|-------|----------|
| Cross-database query fails with "not found" | The three-part name uses the item **display name**, not a GUID. If the lakehouse is named `wwi_lakehouse`, use `wwi_lakehouse.dbo.table_name`. Both items must be in the same workspace. |
| INSERT fails on lakehouse | The lakehouse SQL analytics endpoint is read-only. All DML (INSERT, UPDATE, DELETE, MERGE) must target the **warehouse**, not the lakehouse. |
| MERGE syntax error | Common issues: missing semicolon after MERGE, forgetting the `AS target`/`AS source` aliases, or not matching column counts in INSERT VALUES. |

### Challenge 04: Real-Time Intelligence

| Issue | Solution |
|-------|----------|
| No data flowing after publishing Eventstream | After publishing, click "Configure" on the Eventhouse destination node to complete column mapping. Data does not flow until configuration is finished. |
| Materialized view creation fails | The `.create-or-alter materialized-view` command must be run as a management command, not a regular query. Ensure the KQL queryset is connected to the correct database. |
| Data Activator rule not firing | The rule must be in **Started** state, not just saved. Click the Start button on the rule. Also check that the condition uses a stateful operator (BECOMES, INCREASES) to avoid trigger spam. |

### Challenge 05: Data Science + Power BI

| Issue | Solution |
|-------|----------|
| Custom MLflow metrics not appearing | Autologging runs in `exclusive=True` mode by default. Call `mlflow.autolog(exclusive=False)` before starting the run. |
| PREDICT function fails with signature error | The model was logged without `infer_signature()`. Team must re-log the model with `signature=infer_signature(X_train, y_train)` and re-register it. |
| No semantic model available for Power BI | Default semantic models are no longer auto-created (changed Sept 2025). Team needs to create one manually from the lakehouse or warehouse settings. |
| Direct Lake falling back to DirectQuery | Check if RLS is applied at the SQL analytics endpoint. RLS forces DirectQuery fallback. For the hackathon, avoid applying RLS. |

## Parallel Track Guidance

- Challenges 03 and 04 are designed to run in parallel after Challenge 01.
- If a team has strong SQL skills, recommend Challenge 03 first.
- If a team is more curious about streaming or KQL, recommend Challenge 04 first.
- Only Challenge 03 is a prerequisite for Challenge 05 (not 04).
- Advanced teams can split: half the team works on Challenge 03 while the other half does Challenge 04, then they reconvene for Challenge 05.

## Verification Commands

Quick checks coaches can run to verify team progress:

**Challenge 00:** Open workspace > confirm lakehouse exists > expand Files > raw > verify 5 Parquet files.

**Challenge 01:** Open lakehouse > Tables section should show 5 silver_ tables > switch to SQL endpoint > run `SELECT COUNT(*) FROM silver_fact_sale`.

**Challenge 02:** Open pipeline > verify 4+ activities on canvas > check run history for at least one successful run.

**Challenge 03:** Open warehouse > verify star schema tables in explorer > run `SELECT TOP 5 * FROM dbo.fact_sales` > execute stored procedure and confirm no errors.

**Challenge 04:** Open Eventhouse > query table `YourTable | count` > check Real-Time Dashboard has 3 tiles > verify Activator rule shows "Started" status.

**Challenge 05:** Open ML experiment > verify logged run with metrics > check lakehouse for sales_predictions table > open Power BI report > verify 3 visuals and at least one DAX measure in the field list.
