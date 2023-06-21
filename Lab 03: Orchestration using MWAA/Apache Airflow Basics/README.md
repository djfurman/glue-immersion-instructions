# Apache Airflow Basics

## Concepts
As discussed in the last module Airflow helps us to author a workflow using python wherein we define all the stages of a data pipeline. In this module we will understand these core concepts which we need to use to create these workflows / pipelines.

## Key Concepts

**DAG (Directed Acyclic Graph)**: DAGs are collections of tasks and describe how to run a workflow written in Python. Pipelines are designed as a directed acyclic graph by dividing a pipeline into tasks that can be executed independently. Then these tasks are combined logically as a graph.

**Task**: A Task defines a unit of work within a DAG; it is represented as a node in the DAG graph. Each task is an implementation of an Operator, for example a PythonOperator to execute some Python code, or a BashOperator to run a Bash command. After an operator is instantiated, it’s referred to as a “task.”

**Task instance**: A task instance represents a specific run of a task characterized by a DAG, a task, and a point in time.

**Operators**: Operators are atomic components in a DAG describing a single task in the pipeline. They determine what gets done in that task when a DAG runs. Airflow provides operators for common tasks. It is extensible, so you can define your own custom operators.

**Sensors**: Are special types of operators whose purpose is to wait on some external or internal trigger. Some common types of sensors are:

* ExternalTaskSensor: waits on another task (in a different DAG) to complete execution.

* HivePartitionSensor: waits for a specific value of partition of hive table to get created

* S3KeySensor: S3 Key sensors are used to wait for a specific file or directory to be available on an S3 bucket.

* Hooks: Provide a uniform interface to access external services like S3, MySQL, Hive, EMR, etc. Hooks are the building blocks for operators to interact with external services.

* Scheduling: The DAGs and tasks can be run on demand or can be scheduled to be run at a certain frequency defined as a cron expression in the DAG.

## Basics of writing a DAG
**Step-1**: Import Modules

**Step-2**: Define default args

**Step-3**: Instantiate DAGs / Create a DAGs object

**Step-4**: Define Tasks

**Step-5**: Define Dependencies

Below is a DAG base template which you can use and build upon during this workshop:


```python
from datetime import timedelta  
import airflow  
from airflow import DAG  
from airflow.providers.amazon.aws.sensors.s3_prefix import S3PrefixSensor
from airflow.providers.amazon.aws.operators.glue import AwsGlueJobOperator
from airflow.providers.amazon.aws.operators.glue_crawler import AwsGlueCrawlerOperator

## Setting S3 bucket name
S3_BUCKET_NAME = 'glueworkshop-CHANGE_AWS_ACCOUNT-us-east-2' #<<Change your AWS account here

##Configuring DAG 
default_args = {  
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': airflow.utils.dates.days_ago(1),
    'retries': 0,
    'retry_delay': timedelta(minutes=2),
    'provide_context': True,
    'email': ['airflow@example.com'],
    'email_on_failure': False,
    'email_on_retry': False
}

dag = DAG(  
    'glue_mwaa_scheduler',
    default_args=default_args,
    dagrun_timeout=timedelta(hours=2),
    schedule_interval='0 3 * * *'
)

##Task to wait for a file to be dropped

s3_sensor = S3PrefixSensor(  
  task_id='s3_sensor',  
  bucket_name=S3_BUCKET_NAME,  
  prefix='data/raw/mwaa-green',  
  dag=dag  
)

##Task to start a glue job
glue_task = AwsGlueJobOperator(  
    task_id="glue_task",  
    job_name='ny-taxi-transformed-cli-jb',  
    iam_role_name='AWSGlueServiceRole-default',  
    script_args={'--BUCKET_NAME': S3_BUCKET_NAME,
                 '--JOB_NAME': 'glue-job-mwaa',
                 '--SRC_PREFIX': 'data/raw/mwaa-green',
                 '--TRG_PREFIX': 'target/mwaa-transformed'},
    dag=dag) 



s3_sensor >> glue_task

```