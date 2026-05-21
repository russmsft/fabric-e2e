# Challenge 02 - Orchestrating with Data Factory

[< Previous Challenge](challenge-01.md) | [Home](../README.md) | [Next Challenge >](challenge-03.md)

## Introduction

Your team now has a working Spark notebook that transforms raw data into Silver Delta tables. That is great for one-off runs. But nobody wants to be the person who opens a notebook at 6 AM every morning, hits "Run All", and waits around to see if it finishes cleanly. That is not engineering - that is babysitting.

In any real data platform, transformation logic runs on a schedule without human involvement. When it fails, someone gets notified. When it succeeds, the result is recorded. This is what orchestration gives you: a pipeline that runs your notebook, handles errors with a clear message, and keeps a history of every execution so you can tell your stakeholder exactly when the last run completed and whether it worked.

In this challenge, you will build a Data Factory pipeline in Microsoft Fabric that wraps your Challenge 01 notebook in a proper orchestration layer - with parameters, branching logic, error handling, and a schedule.

## Description

Build a Data Factory pipeline in your Fabric workspace that orchestrates the notebook you created in Challenge 01. Your pipeline should not just run the notebook - it should be a production-ready workflow with parameterization, error handling, and scheduling.

Here is what your finished pipeline needs to include:

**Pipeline and parameters** - Create a new Data Factory pipeline in your workspace. Add at least one pipeline parameter - for example, a `run_date` string parameter or a parameter that controls which tables to process. This parameter should be passed through to the notebook activity so the notebook could act on it.

**Notebook activity** - Add a Notebook activity to the pipeline canvas. Configure it to run your Challenge 01 transformation notebook. Pass your pipeline parameter(s) into the notebook as base parameters.

**Error handling** - Your pipeline should not silently swallow failures. From the notebook activity, create two branching paths:

- On failure: connect to a Fail activity with a descriptive error message and an error code. The message should include useful context - not just "it broke."
- On success: connect to a second activity that records the successful completion. A Set Variable activity that captures the current timestamp into a pipeline variable works well here.

**Schedule** - Configure a recurring schedule for the pipeline. Pick a daily time that makes sense (for a hackathon, any time is fine - the point is that you know how to set one up).

**Run and verify** - Run the pipeline manually at least once. Watch the execution in the monitoring view and confirm that every activity shows a green checkmark. Verify that the Delta tables from your notebook are present and updated after the run.

You should not need to modify your Challenge 01 notebook to complete this challenge. The pipeline wraps existing logic - it does not replace it.

## Success Criteria

- A Data Factory pipeline exists in the workspace with at least four activities visible on the canvas (Notebook, Fail, and at least one success-path activity, connected with dependency arrows).
- The pipeline has at least one parameter defined, and that parameter is passed to the Notebook activity's base parameters.
- The Notebook activity is configured to run the transformation notebook from Challenge 01.
- A failure branch connects the Notebook activity to a Fail activity with a non-default error message.
- A success branch connects the Notebook activity to an activity that records completion (e.g., Set Variable with a timestamp expression).
- A schedule is configured on the pipeline (visible in the Schedule tab).
- At least one successful pipeline run is visible in the run history, with all activities showing completed status.
- The Silver Delta tables from Challenge 01 exist and contain data after the pipeline run.

## Hints

<details>
<summary>Hint 1: Where to start</summary>

In your Fabric workspace, select **+ New item** and search for **Data Pipeline**. Once the pipeline editor opens, you will find activities in the **Activities** tab at the top of the canvas. The Notebook activity is under the Data Transformation category - or you can just search for "Notebook" in the activity search bar.

</details>

<details>
<summary>Hint 2: Adding pipeline parameters</summary>

Pipeline parameters are not the same as notebook parameters. You define pipeline parameters on the pipeline itself, then reference them inside activity settings using expressions.

To add a pipeline parameter, look for the pipeline-level settings (not the activity settings). Once you have a parameter like `run_date` defined, you reference it in a Notebook activity's base parameters using the expression syntax: `@pipeline().parameters.run_date`

</details>

<details>
<summary>Hint 3: Setting up the branching paths</summary>

When you hover over the edge of a completed Notebook activity on the canvas, you will see small connector icons. These let you draw dependency lines to the next activity. The key is that each connector has a condition type:

- A green checkmark connector means "on success"
- A red X connector means "on failure"

Drag the failure connector to your Fail activity. Drag the success connector to your Set Variable (or other success-tracking) activity. You need to add these activities to the canvas first before you can connect them.

</details>

<details>
<summary>Hint 4: Configuring the Fail activity and Set Variable</summary>

The Fail activity has two required fields in its Settings tab: **Error message** and **Error code**. You can use expressions here. For example, a message like:

```
@concat('Notebook execution failed for run date: ', pipeline().parameters.run_date)
```

For the Set Variable activity, you first need to create a pipeline variable (look for the Variables tab at the bottom of the pipeline canvas). Create a string variable, then configure the Set Variable activity to set it. A useful expression for a completion timestamp:

```
@utcnow()
```

</details>

<details>
<summary>Hint 5: Schedule and first run</summary>

You will find the **Schedule** button on the **Home** tab of the pipeline editor ribbon. Set the frequency to "Day" and pick any time. You need to set a start date and time zone.

Before running manually, save the pipeline first - Fabric will not let you run unsaved changes. Hit **Run** from the Home tab, and the **Output** tab at the bottom of the editor will show real-time progress. Click on any activity row in the output to see its detailed input and output, which is especially useful for debugging.

Remember: on-demand runs do not trigger failure email notifications. Only scheduled runs send those alerts.

</details>

## Learning Resources

- [Create your first pipeline in Data Factory](https://learn.microsoft.com/en-us/fabric/data-factory/create-first-pipeline-with-sample-data)
- [Notebook activity in Data Factory pipelines](https://learn.microsoft.com/en-us/fabric/data-factory/notebook-activity)
- [Expression language in Data Factory](https://learn.microsoft.com/en-us/fabric/data-factory/expression-language)
- [Pipeline runs - on-demand, scheduled, and event-based](https://learn.microsoft.com/en-us/fabric/data-factory/pipeline-runs)
- [Monitor pipeline runs](https://learn.microsoft.com/en-us/fabric/data-factory/monitor-pipeline-runs)
- [Fail activity](https://learn.microsoft.com/en-us/fabric/data-factory/fail-activity)
- [Activity overview - Data Factory](https://learn.microsoft.com/en-us/fabric/data-factory/activity-overview)
