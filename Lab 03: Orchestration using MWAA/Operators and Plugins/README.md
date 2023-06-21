# Operators and Plugins

## Operators

Operators form the core of Airflow tasks. They define a single unit-of-work within the Airflow model and are the gateway to customizing Airflow to run business logic. The operator library is a key component of what makes Airflow a compelling choice as an orchestration engine.

The Airflow community features hundreds of pre-built operators and subclassing operators is very simple.

The official airflow documentation goes into some depth of how operators work and can be found below:

* **[Airflow Operators User Guide](https://airflow.apache.org/docs/apache-airflow/stable/howto/operator/index.html)**

* **[Airflow Operators Reference](https://airflow.apache.org/docs/apache-airflow/stable/_api/airflow/operators/index.html)**

* **[Airflow Operator Concepts](https://airflow.apache.org/docs/apache-airflow/stable/concepts.html#concepts-operators)**

At a high-level, operators are Python classes. When you want to customize an Airflow operator, we subclass the BaseOperator class from Airflow.


## Plugins

Plugins can be thought of as a set of operators, hooks, and sensors that we'd like to import and consider as a group. You can pass your plugins as a packaged **.zip** file to **MWAA** by specifying the location of the zip file in s3.