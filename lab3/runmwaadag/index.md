---
chapter: false
title: "View, Execute and Monitor your DAG"
weight: 43
---

::alert[Please complete the prerequisite [How to start?](/howtostart/awseevnt/s3-and-local-file.html) and [Lab1](/lab1.html)  section before starting this lab.]


# Adding your first DAG in your MWAA Environment

1. In [AWS Cloud9](https://us-east-2.console.aws.amazon.com/cloud9/), click on the menu bar **File** and then **New File**. This will open a new empty file. Paste the following code in the file.
``` python
from datetime import timedelta  
import airflow  
from airflow import DAG  
from airflow.providers.amazon.aws.sensors.s3_prefix import S3PrefixSensor
from airflow.providers.amazon.aws.operators.glue import AwsGlueJobOperator
from airflow.providers.amazon.aws.operators.glue_crawler import AwsGlueCrawlerOperator

## Setting S3 bucket name
S3_BUCKET_NAME = 'glueworkshop-CHANGE_AWS_ACCOUNT-us-east-2' #enter your S3 bucketname here  

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

2. On line 9 of the pasted script, change the `CHANGE_AWS_ACCOUNT` to your own AWS account number.

3. Save the file as `mwaa-dag.py`, under the `glueworkshop` home directory 

![failed state machine](/static/Glue%20Jobs/Lab%204/lab4-1-1.png)

4. Run the following AWS CLI command in the Cloud9 terminal to copy the newly updated dag into the s3 path our MWAA is expecting it to be.
```bash
    aws s3 cp ~/environment/mwaa-dag.py s3://mwaa-${AWS_ACCOUNT_ID}-us-east-2/dags/
```

5. Navigate to the [Airflow environment page](https://us-east-2.console.aws.amazon.com/mwaa/home?region=us-east-2#environments), select your existing MWAA environment and click on the **Open Airflow UI** link to your Airflow UI

![failed state machine](/static/Glue%20Jobs/Lab%204/lab4-1-0.png)

::alert[You may have to wait a few minutes for the DAG to show up, if no success please review previous steps.]

![failed state machine](/static/Glue%20Jobs/Lab%204/lab4-2-0.png)

6. To simulate an S3 event that will trigger our MWAA dag, please run the following command in the cloud9 terminal.

```bash
aws s3 cp ~/environment/green_tripdata.csv s3://${BUCKET_NAME}/data/raw/mwaa-green/
```
![failed state machine](/static/Glue%20Jobs/Lab%204/lab4-2-1.png)

This concludes the Lab 3 section of this workshop.