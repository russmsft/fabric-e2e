# Challenge 05 - Data Science and Power BI Reporting

[< Previous Challenge](challenge-04.md) | [Home](../README.md)

## Introduction

Your team has spent the last several challenges building a data platform from scratch - ingesting raw Wide World Importers data, shaping it into Silver Delta tables, orchestrating pipelines, and standing up a star schema in the Fabric Data Warehouse. That is a solid foundation, but nobody at Wide World Importers gets excited about well-organized tables. They want answers. Specifically, the sales director wants to know which upcoming orders are likely to generate high profit and which ones are not worth the expedited shipping cost.

This challenge puts the Data Science and Power BI workloads to work on top of the data you already have. You will train a machine learning model inside a Fabric notebook, register it with MLflow so it is versioned and reproducible, then score new data at scale using the PREDICT function. Finally, you will build a Power BI report that connects to those prediction results through Direct Lake mode - no data imports, no scheduled refreshes, just Delta files read straight from OneLake.

This is the capstone. Everything you built in earlier challenges feeds into this one.

## Description

### Part A - Train and Track an ML Model (25 minutes)

Create a new notebook in your Fabric workspace dedicated to machine learning work. Your goal is to train a model that predicts something useful about Wide World Importers sales data and track the entire experiment with MLflow.

Start by loading data from your Silver layer tables. You will need to join `silver_fact_sale` with one or more dimension tables to get a feature-rich dataset. Pick a prediction target that makes sense for the business:

- **Classification approach:** predict whether a sale will exceed a profit threshold you define (for example, profit greater than $50)
- **Regression approach:** predict the `total_including_tax` amount for a sale

Prepare your features, split into training and test sets, and train a scikit-learn model of your choice (random forest, logistic regression, gradient boosting - whatever fits the problem).

Here is where it gets important: use MLflow to track everything about this experiment. Name your experiment `wwi-sales-prediction`. Log the parameters you chose, the evaluation metrics you computed, and the trained model itself. Register the finished model in the Fabric model registry under a name like `wwi-sales-model`.

There is one detail that will save you from a painful debugging session later: the model **must** have a signature attached when you log it. Without that signature, the PREDICT function in Part B will refuse to work, and the error message will not make it obvious why.

After the notebook runs successfully, open the experiment view in the Fabric portal. Confirm you can see your logged metrics, parameters, and the registered model version.

One more thing to watch for: Fabric auto-enables MLflow autologging when you import the `mlflow` package. By default, autologging runs in exclusive mode, which means your own manual `log_metric` and `log_param` calls get silently ignored. If you find that your custom metrics are not showing up in the experiment, that is probably why.

### Part B - Batch Scoring with PREDICT (15 minutes)

With a trained and registered model in hand, score a batch of sales data using the PREDICT function. The PREDICT function in Fabric uses `MLFlowTransformer` from the `synapse.ml.predict` package. It loads your registered model by name and version, transforms a Spark DataFrame of input features, and appends a prediction column.

Prepare a scoring dataset from your Silver layer - this should contain the same feature columns you used during training, but without the target column. Run the model against this data and write the resulting predictions back to your lakehouse as a new Delta table named `sales_predictions`.

After writing the table, open the SQL analytics endpoint for your lakehouse in the Fabric portal. The `sales_predictions` table should appear there alongside your other tables. If it does not show up, you may need to wait a moment for the endpoint metadata to sync.

### Part C - Power BI Report with Direct Lake (15 minutes)

Build a Power BI report that brings your prediction results to life. The report should connect to a semantic model backed by your lakehouse or warehouse, and it must use Direct Lake mode - meaning data is read directly from the Delta Parquet files in OneLake without being imported into the Power BI model.

Your report needs at least three visualizations:

- A bar or column chart comparing actual values against predicted values, broken down by a dimension such as city, customer category, or stock item group
- A KPI card showing a summary metric - model accuracy for a classifier, or total predicted revenue for a regression model
- A detail table listing individual predictions alongside key attributes like customer name, stock item, and sale date

Create at least one DAX measure. For a classification model, a good measure would be prediction accuracy (correct predictions divided by total predictions using the DIVIDE function). For a regression model, try total predicted revenue as a SUM over the prediction column.

Before you call this done, verify that your report is actually running in Direct Lake mode. If Row-Level Security is applied at the SQL analytics endpoint, Direct Lake will silently fall back to DirectQuery - a different execution path with different performance characteristics.

## Success Criteria

- A Fabric notebook exists that trains an ML model on data from the Silver layer tables.
- An MLflow experiment named `wwi-sales-prediction` is visible in the Fabric portal with at least one completed run.
- The experiment run contains logged parameters (at least 2), metrics (at least 1), and a registered model.
- The registered model has a signature (visible in the model artifact details).
- A Delta table named `sales_predictions` exists in the lakehouse containing scored prediction results.
- The `sales_predictions` table is visible in the lakehouse SQL analytics endpoint.
- A Power BI report exists with at least 3 visualizations showing prediction results.
- The report includes at least one custom DAX measure.
- The report connects via Direct Lake mode (not Import or DirectQuery).

## Hints

<details>
<summary>Hint 1: Getting started with data preparation</summary>

Your Silver layer has fact and dimension tables. Use PySpark to load and join them. Something like joining `silver_fact_sale` with `silver_dim_stock_item` or `silver_dim_customer` gives you feature columns beyond just the raw sale amounts. Convert the resulting Spark DataFrame to a pandas DataFrame for scikit-learn training. Think about which columns are actually predictive and which ones are just identifiers that would leak information or add noise.

</details>

<details>
<summary>Hint 2: Autologging is blocking your custom metrics</summary>

When you `import mlflow` in a Fabric notebook, autologging turns on automatically with `exclusive=True`. That means any `mlflow.log_metric()` or `mlflow.log_param()` calls you write will be ignored without warning. Call `mlflow.autolog(exclusive=False)` before you start your run. This lets autologging continue to capture its default metrics while also recording your custom ones.

</details>

<details>
<summary>Hint 3: The model signature - why PREDICT fails without it</summary>

The PREDICT function (MLFlowTransformer) needs to know the exact input schema of your model. That schema comes from the model signature. When logging your model, create the signature before the `log_model` call:

```python
from mlflow.models.signature import infer_signature
signature = infer_signature(X_train, y_train)
```

Then pass `signature=signature` as a parameter when you call `mlflow.sklearn.log_model(...)`. If you skip this, the model will register fine but PREDICT will throw an error when you try to transform data.

</details>

<details>
<summary>Hint 4: Registering the model so PREDICT can find it</summary>

Logging a model as an artifact is not the same as registering it. PREDICT looks for models in the Fabric model registry, not in run artifacts. You can register during logging by passing `registered_model_name="wwi-sales-model"` to `log_model`, or you can register separately after the run using `mlflow.register_model()` with the run URI. Either way, note the model version number - you will need it for the MLFlowTransformer in Part B.

</details>

<details>
<summary>Hint 5: Setting up MLFlowTransformer for batch scoring</summary>

The `MLFlowTransformer` class lives in `synapse.ml.predict`. You need to tell it three things: which columns from your DataFrame to use as inputs, what to name the output column, and which registered model (name and version) to load. The `inputCols` parameter takes a list of column names that must match the features your model was trained on - same names, same order, same types. If you trained on a pandas DataFrame and you are scoring a Spark DataFrame, watch out for column type mismatches between pandas and Spark.

</details>

<details>
<summary>Hint 6: Writing predictions back as a Delta table</summary>

After calling `model.transform(scoring_df)`, you get back a Spark DataFrame with your prediction column appended. Write it to the lakehouse using:

```python
predictions_df.write.mode("overwrite").format("delta").saveAsTable("sales_predictions")
```

If the table does not appear in the SQL analytics endpoint immediately, give it a minute. The endpoint metadata syncs on a short delay. You can also try refreshing the lakehouse view in the portal.

</details>

<details>
<summary>Hint 7: Creating a report with Direct Lake</summary>

In the Fabric portal, navigate to your lakehouse and find the semantic model associated with it. If no default semantic model exists, you can create one from the lakehouse or warehouse settings. From the semantic model, select "New Report" to open the Power BI report editor. Tables from your lakehouse - including `sales_predictions` - will appear in the field list. Direct Lake is the default connection mode for semantic models backed by Fabric lakehouses, so you should not need to configure it manually. To verify the mode, check the semantic model settings and look for the storage mode listed on each table.

</details>

<details>
<summary>Hint 8: Writing a DAX measure for prediction accuracy</summary>

For a classification model where you have both the actual label and the predicted label in the same table, a simple accuracy measure looks like:

```
Prediction Accuracy = DIVIDE(
    COUNTROWS(FILTER(sales_predictions, sales_predictions[actual] = sales_predictions[prediction])),
    COUNTROWS(sales_predictions)
)
```

Adjust column names to match your table. For a regression model, consider a total predicted revenue measure using `SUM(sales_predictions[prediction])` instead.

</details>

## Learning Resources

- [Machine learning experiments in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/data-science/machine-learning-experiment)
- [Machine learning models in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/data-science/machine-learning-model)
- [Train models with scikit-learn in Fabric](https://learn.microsoft.com/en-us/fabric/data-science/train-models-scikit-learn)
- [PREDICT model scoring in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/data-science/model-scoring-predict)
- [MLflow autologging in Fabric](https://learn.microsoft.com/en-us/fabric/data-science/mlflow-autologging)
- [Direct Lake overview](https://learn.microsoft.com/en-us/fabric/fundamentals/direct-lake-overview)
- [Create measures in Power BI](https://learn.microsoft.com/en-us/power-bi/transform-model/desktop-measures)
- [Data Science end-to-end tutorial](https://learn.microsoft.com/en-us/fabric/data-science/tutorial-data-science-introduction)
