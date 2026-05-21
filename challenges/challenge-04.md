# Challenge 04 - Real-Time Intelligence Pipeline

[< Previous Challenge](challenge-03.md) | [Home](../README.md) | [Next Challenge >](challenge-05.md)

## Introduction

Your operations team has been asking for the same thing for months: a way to watch incoming events as they happen, not hours later in a batch report. They want to see spikes the moment they occur, track volume in real time, and get an alert before something goes wrong - not after.

This is a common ask in any data platform project, and it is exactly what Fabric Real-Time Intelligence was built for. The stack connects an event source to a KQL database through an Eventstream, gives you a query language purpose-built for time series data, and lets you pin live visuals to a dashboard that refreshes on its own. On top of that, Data Activator watches the stream and fires notifications when conditions you define are met.

This challenge stands on its own. You will use Fabric's built-in sample data generator as your event source, so there is no dependency on the lakehouse data from earlier challenges. Your team can start this challenge at the same time another group tackles the Data Warehouse in Challenge 03.

By the end of this challenge, you will have a working pipeline that ingests streaming events, queries them with KQL, displays them on an auto-refreshing dashboard, and triggers an alert when a value crosses a threshold.

## Description

Your goal is to build an end-to-end real-time analytics pipeline inside your existing Fabric workspace. The pipeline has four stages: ingest, query, visualize, and act.

**Stage 1 - Ingest:** Stand up an Eventhouse in your workspace. Create an Eventstream that uses one of Fabric's built-in sample data sources (Bicycles or Yellow Taxi work well) and route that stream into a new table in the Eventhouse's KQL database. After publishing, confirm that rows are landing in the table.

**Stage 2 - Query:** Open a KQL queryset connected to your database. Write at least three distinct queries:

- A filtered query that returns recent rows matching a condition
- An aggregation that buckets event counts into time intervals and renders a time chart
- A time series query that computes an average metric over a rolling window

Then create a materialized view that pre-computes a useful aggregation so downstream consumers get fast results without re-scanning the raw table every time.

**Stage 3 - Visualize:** Create a Real-Time Dashboard backed by your KQL database. The dashboard must have at least three tiles:

- A time chart showing event volume over time
- A chart that breaks down events by a categorical field (neighborhood, street, location, or similar)
- A stat or summary tile that shows a single KPI value

Turn on auto-refresh so the dashboard updates without manual intervention.

**Stage 4 - Act:** Connect Data Activator to your pipeline. Define an object based on a grouping identifier from your stream, create a rule with a condition (for example, "when a metric exceeds a threshold"), set the action to send an email, and activate the rule.

## Success Criteria

- An Eventhouse exists in the workspace with a KQL database containing at least one table that is actively receiving streaming data.
- A published Eventstream connects a sample data source to the Eventhouse KQL database, and new rows appear when the table is queried.
- A KQL queryset contains at least three queries: a filtered query, a time-bucketed aggregation with `render timechart`, and a time series average.
- A materialized view exists on the KQL database and returns pre-aggregated results when queried.
- A Real-Time Dashboard has at least three tiles (time chart, categorical breakdown, KPI stat) and auto-refresh is turned on.
- A Data Activator rule is in the active/started state with a defined condition and email action.

## Hints

<details>
<summary>Hint 1: Getting the Eventhouse and Eventstream connected</summary>

Look for "Eventhouse" in the **+ New item** menu of your workspace. When you create one, a KQL database with the same name appears automatically - you do not need to create the database separately.

For the Eventstream, also create it from **+ New item**. When adding a source, pick **Sample data** and choose one of the presets. The Bicycles dataset is a good fit because it has numeric and categorical fields.

</details>

<details>
<summary>Hint 2: Routing the stream to the database</summary>

After adding your sample source in the Eventstream editor, add a **destination** and pick **Eventhouse**. Select **Direct ingestion** as the mode. You will point it at your Eventhouse, select the KQL database, and create a new table.

After publishing the Eventstream, you still need to configure the ingestion - click "Configure" on the destination node to map columns and set data types. Once that is done, data should start flowing within a minute or two.

</details>

<details>
<summary>Hint 3: Writing effective KQL queries</summary>

KQL uses a pipe syntax that reads left to right. Start with the table name, pipe into operators. A few patterns to try:

```kql
YourTable | where Timestamp > ago(1h) | take 50
```

For time-bucketed counts, the `bin()` function groups timestamps into intervals:

```kql
YourTable | summarize count() by bin(Timestamp, 1m)
```

Add `| render timechart` at the end of a summarize query to get a visual directly in the queryset.

</details>

<details>
<summary>Hint 4: Creating a materialized view</summary>

Materialized views are created with a management command, not a regular query. The command starts with `.create-or-alter materialized-view` and contains a KQL query body wrapped in braces. The body must be a single `summarize` statement over your raw table.

A useful pattern is `arg_max(Timestamp, *)` grouped by a key column - this gives you the latest record for each unique key. You can query the materialized view by name just like a regular table.

</details>

<details>
<summary>Hint 5: Building the Real-Time Dashboard</summary>

Create the dashboard from **+ New item > Real-Time Dashboard**. Before adding tiles, you need to add a **data source** from the Manage tab - point it at your KQL database via the OneLake catalog.

Switch to **Editing** mode to add tiles. Each tile needs a KQL query. For a stat tile, write a query that returns a single row with one numeric value. For the auto-refresh setting, look under the **Manage** tab while in edit mode.

</details>

<details>
<summary>Hint 6: Setting up Data Activator</summary>

You can add an Activator destination directly from your Eventstream, or create a Reflex item from a Real-Time Dashboard tile. Either path works.

When configuring the rule, pick a field to use as your **object identifier** - this is how Activator groups events (for example, all events from the same bike station). Then set a property to monitor, add a detection condition (look for "becomes", "increases by", or "is greater than"), and choose email as the action. Do not forget to click **Start** on the rule - a saved rule that is not started will not fire.

</details>

## Learning Resources

- [Eventhouse overview](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/eventhouse)
- [Create an Eventhouse](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/create-eventhouse)
- [Eventstream overview](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/event-streams/overview)
- [Add sample data to an Eventstream](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/event-streams/add-source-sample-data)
- [Route Eventstream to an Eventhouse](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/event-streams/add-destination-kql-database)
- [KQL quick reference](https://learn.microsoft.com/en-us/kusto/query/kql-quick-reference)
- [Materialized views in Fabric](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/materialized-view)
- [Create a Real-Time Dashboard](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/dashboard-real-time-create)
- [Data Activator introduction](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/data-activator/activator-introduction)
- [Activator detection conditions](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/data-activator/activator-detection-conditions)
