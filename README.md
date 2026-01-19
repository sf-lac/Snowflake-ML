# ❄️💰 End-to-end Snowflake ML Workflow for Loan Lending Prediction

This repository contains a complete __end-to-end machine learning workflow__ implemented in a __Snowflake Notebook__ using __Snowflake ML__, covering:

- Data ingestion & feature engineering
- Distributed model training with hyperparameter optimization
- Model explainability
- Model registry & deployment
- Inference (warehouse, stored procedure, and Snowpark Container Services)
- Model observability & monitoring

This workflow demonstrates how to build, optimize, deploy, monitor, and serve a loan lending prediction model entirely within Snowflake - no data movement required.

## 🏗️ Architecture Overview

![](./images/architechture/architecture.png)

## 🔄 Overview of Workflow

__1. Data Ingestion and Loading__

- Load raw data from a staged CSV.
- Use __`Snowflake ML DataSource and DataSink APIs`__ with Ray for distributed ingestion and preprocessing.
---
__2. Feature Engineering__

- Transform raw data and register reusable transformations via the __`Snowflake Feature Store`__.
- Define __`Entities`__ and __`Feature Views`__ for consistent feature generation.
- Materialize __`Datasets`__ for model training and inference using __FeatureStore.generate_dataset__ method.
---
__3. Model Training and Hyperparameter Optimization (HPO)__

- Train a baseline model using __Snowflake ML XGBClassifier__.
- Use a __dataset_map__ mapping train/test datasets to  corresponding __`DataConnector`__ objects for ingestion into the HPO workflow. 
- Use Snowflake's __`HPO API`__ to perform parallelized hyperparameter tuning with hyperparameters search space definition, choice of search algorithm, and scalable executions across compute nodes.
- Use __`Snowflake ML Experiment Tracking`__ to log hyperparameters and metrics.
---
__4. Model Registry and Lifecycle__

- Use the __`Snowflake Model Registry`__ to operationalize trained models and run inference in Snowflake.
- Register models with versioning, metadata, and metrics using Model Registry's __log_model__ method.
- Manage lifecycle transitions across environments with flexible governance.
---
__5. Model Explainability__

- Compute __Shapley values__ via the Model Registry to attribute predictions to input features.
- Use representative __background data__ for consistent explanations.
---
__6. Model Inference__

__Warehouse Inference__

- Best for small-medium CPU models with dependencies in Snowflake Conda.
- Use __run__ method of a __`ModelVersion` (mv)__ object retrieved from the Model Registry to make predictions, passing in the __input inference data__ as a __Snowpark or pandas Dataframe__ and  the __function of the model__ to call (e.g., __predict__).
- __Model predictions__ are returned by __mv.run__ as a __Dataframe of the same type__ as the input.

__Batch Warehouse Inference with Stored Procedure__

- Wrap __mv.run__ call in a __Snowflake Python stored procedure__.
- Automates __table-base batch predictions__ and saves joined output/input back to Snowflake.
- Supports __schema evolution__ and __reusable calls from SQL or Python__, suitable for __production workloads__.

__Model Serving via Snowpark Container Services (SPCS)__

- __Deploys the model__ as a __`container service`__ running on a __`compute pool`__.
- Supports __custom dependencies__, __scalable compute__, __internal and external secure HTTP endpoints__ for inference within Snowflake and external applications.
- __Create a service for SPCS deployment__ using __`mv.create_service`__.
- __Run inference__ using the service by calling __`mv.run`__ including the __service_name__ parameter. 

---

__7. ML Observability and Model Monitoring__

__Observability__

Track __production model behavior__ with __metrics__ on:
- __Drift__ (input or output distribution changes)
- __Performance degradation__
- __Volume changes__
- __Segment-level__ behavior

__Monitoring__

Define __`Model Monitor`__ objects with:
- Monitored __ModelVersion + Function__
- __Source__ table
- __Warehouse__ compute
- __Refresh and aggregation__ settings
- Optional Baseline table for drift comparison
- Optional ID, prediction, actual, and segment columns

__Querying Monitor Metrics__

| Metric Type         | Query Function                     | Description                                                                  |
| ------------------- | ---------------------------------- | ---------------------------------------------------------------------------- |
| Drift Metrics       | `MODEL_MONITOR_DRIFT_METRIC`       | Returns metrics on distribution or statistical shifts in monitored features. |
| Performance Metrics | `MODEL_MONITOR_PERFORMANCE_METRIC` | Returns metrics tracking model prediction quality over time.                 |
| Statistical Metrics | `MODEL_MONITOR_STAT_METRIC`        | Returns statistics on data quality, counts, or null values.                  |

## 🔐 Governance

__`RBAC`__:
- __Warehouse__, __Feature Store__ usage
- __Model Registry__, __Service__ privileges

## 🏁 Summary

This repository illustrates a __full Snowflake-native ML lifecycle__ from __ingestion to deployment__, __serving__, and __monitoring__ - emphasizing:

- __Reproducibility__
- __Scalability__
- __Governed workflows__
- __Enterprise readiness__
