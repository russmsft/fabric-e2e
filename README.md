# Microsoft Fabric End-to-End Hackathon

Build a complete data analytics platform on Microsoft Fabric using the Wide World Importers retail dataset. Over four hours, your team will move raw sales data through a medallion lakehouse, orchestrate transformations with Data Factory, build a star schema warehouse, stand up a real-time intelligence pipeline, train a machine learning model, and create a Power BI report - all within a single Fabric workspace.

## Event Details

| Detail | Value |
|--------|-------|
| Duration | 4 hours |
| Content Level | L300 (code-level: PySpark, T-SQL, KQL, MLflow) |
| Target Audience | Mixed data teams - engineers, analysts, scientists |
| Prerequisites | Organizational (Entra ID) account, Fabric Trial or F2+ capacity, modern browser |

## Challenges

| Challenge | Title | Duration | Difficulty |
|-----------|-------|----------|------------|
| [Challenge 00](challenges/challenge-00.md) | Setting Up Your Fabric Workspace | 20 min | Setup |
| [Challenge 01](challenges/challenge-01.md) | Building the Medallion Lakehouse | 30 min | Easy |
| [Challenge 02](challenges/challenge-02.md) | Orchestrating with Data Factory | 30 min | Easy |
| [Challenge 03](challenges/challenge-03.md) | Data Warehouse and Cross-Database Analytics | 40 min | Medium |
| [Challenge 04](challenges/challenge-04.md) | Real-Time Intelligence Pipeline | 45 min | Medium |
| [Challenge 05](challenges/challenge-05.md) | Data Science and Power BI Reporting | 55 min | Hard |

## Parallel Paths

Challenges 03 and 04 are independent of each other. Teams can split up and work on them simultaneously once Challenge 01 is complete. Challenge 05 is the capstone and depends on Challenges 01 and 03.

## Architecture

See the [Reference Architecture](resources/reference-architecture.md) for the full system diagram, workload integration map, and challenge dependency graph.

## Prerequisites

Before starting, each participant needs:

- An organizational account (Microsoft Entra ID). Personal Microsoft accounts are not supported.
- Access to a Microsoft Fabric capacity. A [free 60-day trial](https://learn.microsoft.com/en-us/fabric/get-started/fabric-trial) is available.
- A modern web browser (Microsoft Edge or Google Chrome recommended).
- No local software installation is required. All work is done in the Fabric portal at [app.fabric.microsoft.com](https://app.fabric.microsoft.com).

## Getting Started with Codespaces

This repository includes a dev container configuration. Click the green **Code** button above and select **Open with Codespaces** for a zero-install development environment with Python, Azure CLI, and Fabric-related libraries pre-installed. The dev container is optional - most hackathon work happens directly in the Fabric portal.

## Coach Resources

Coaches and facilitators can find guidance in the [coach](coach/) directory:

- [Facilitation Guide](coach/facilitation-guide.md) - timing, common blockers, and preparation checklist
- [Scoring Rubric](coach/scoring-rubric.md) - point system for evaluating team progress
