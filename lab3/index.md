# Lab 03: Orchestration using MWAA

::alert[Please complete [How to start?](/howtostart/awseevnt/s3-and-local-file.html) and [Lab 1](/lab1.html) sections before starting this lab.]

## Apache Airflow Basics

In this module we will learn basics of Apache Airflow by answering a few questions like what is Airflow, why we need Airflow, key concepts and components and Airflow UI walk-through using Amazon Managed workflows for Apache Airflow (MWAA).

What is Airflow?
Apache Airflow is an open-source platform to programmatically author, schedule, and monitor your data pipelines or workflows. It was created at Airbnb in 2014 to manage the company’s increasingly complex workflows and was open-sourced from the beginning.

Before we dive into Airflow lets understand some generic workflow management concepts which will further solidify your understanding.

**Workflow** is a sequence of tasks that processes a set of data helping you to build pipelines.

**Scheduling** is the process of planning, controlling, and optimizing when a particular task should be done.

**Authoring** a workflow using Airflow is done by writing python scripts to create DAGs (Directed acyclic graphs).