---
chapter: false
title: Create Glue Job - CLI
weight: 13
---
::alert[Please complete the prerequisite [How to start?](/howtostart/awseevnt/s3-and-local-file.html) section before starting this lab.]

In this section of the lab we will create, execute and verify the results of an AWS Glue job using CLI commands.The job will be ingesting the data from a csv file stored on s3.

**Prepare S3 bucket**

During the workshop environment setup, an Amazon S3 bucket was created for storing lab files and CloudTrail logs. 

**Create Glue job source code file**

1. In [AWS Cloud9](https://us-east-2.console.aws.amazon.com/cloud9/), click on the menu bar **File** and then **New File**. This will open a new empty file. Paste the following code in the file.
    ```python
    import sys
    from awsglue.transforms import *
    from awsglue.utils import getResolvedOptions
    from awsglue.context import GlueContext
    from awsglue.job import Job
    from pyspark.context import SparkContext

    ## @params: [JOB_NAME]
    args = getResolvedOptions(sys.argv, ['JOB_NAME','BUCKET_NAME','SRC_PREFIX','TRG_PREFIX'])

    sc = SparkContext()
    glueContext = GlueContext(sc)
    spark = glueContext.spark_session
    job = Job(glueContext)
    job.init(args['JOB_NAME'], args)

    ##constructing the source and target folders
    src_path = "s3://"+ args['BUCKET_NAME']+"/"+args['SRC_PREFIX'] +"/"
    trg_path = "s3://"+ args['BUCKET_NAME']+"/"+args['TRG_PREFIX']+"/"

    print ("source file path:", src_path)

    ##creating source dataframe from raw table

    srcDyF  = glueContext.create_dynamic_frame.from_options(
        format_options={"quoteChar": '"', "withHeader": True, "separator": ","},
        connection_type="s3",
        format="csv",
        connection_options={
            "paths": [src_path],
            "recurse": True,
        },
    
    )
    srcDyF.show(10)

    ## Dropping columns with NULL values from the source dynamic frame
    dropnullfields3 = DropNullFields.apply(frame = srcDyF, transformation_ctx = "dropnullfields3")

    ##writing to target s3 location
    datasink4 = glueContext.write_dynamic_frame.from_options(frame = dropnullfields3, connection_type = "s3", connection_options = {"path": trg_path }, format = "parquet")


    job.commit()

    ```

2. Save this file by clicking on the menu bar **File** and then **Save As...**.  Enter `ny-taxi-transformed-cli-jb.py` as **Filename** and click **Save**.


**Create the Glue job to load data from the csv file**

Now that we have the source code file for the Glue job, follow the instructions below to create and run a Glue job with this source code file. 

3. In [AWS Cloud9](https://us-east-2.console.aws.amazon.com/cloud9/), click on the menu bar **Window** and then **New Terminal**. This will open a new terminal window for you. 

4. In this Cloud9 terminal, run the below commands which will copy your source code file to Amazon S3 and create a Glue ETL job with the source code in S3 bucket. 

    ```bash
    cd ~/environment

    aws s3 cp ny-taxi-transformed-cli-jb.py s3://${BUCKET_NAME}/trainingday-scripts/ny-taxi-transformed-cli-jb.py

    aws glue create-job \
        --name ny-taxi-transformed-cli-jb \
        --role AWSGlueServiceRole-glueworkshop \
        --command "Name=glueetl,ScriptLocation=s3://${BUCKET_NAME}/trainingday-scripts/ny-taxi-transformed-cli-jb.py,PythonVersion=3" \
        --glue-version '4.0' \
        --default-arguments "{ \
        \"--BUCKET_NAME\": \"${BUCKET_NAME}\", \
        \"--SRC_PREFIX\": \"data/raw/step-green\", \
        \"--TRG_PREFIX\": \"target/glue-transformed\", \
        \"--JOB_NAME\": \"glue-console-job\", \
        \"--enable-metrics\": \"\", \
        \"--enable-spark-ui\": \"true\", \
        \"--TempDir\": \"s3://${BUCKET_NAME}/output/lab1/temp/\", \
        \"--spark-event-logs-path\": \"s3://${BUCKET_NAME}/output/lab1/sparklog/\" \
        }" \
        --number-of-workers 4 \
        --worker-type Standard
    ```



5. After the job is created we can now start it using the `start-job-run` CLI command:
```bash
    aws glue start-job-run \
        --job-name ny-taxi-transformed-cli-jb \
        --arguments "{ \
        \"--BUCKET_NAME\" :\"${BUCKET_NAME}\", \
        \"--SRC_PREFIX\": \"data/raw/step-green\", \
        \"--TRG_PREFIX\": \"target/glue-transformed\", \
        \"--JOB_NAME\": \"glue-console-job\" \
        }"
```

6. To check the status of Glue job run, go to the [AWS Glue console](https://us-east-2.console.aws.amazon.com/glue/), click **Jobs** on the left, select job `ny-taxi-transformed-cli`. In the **Runs** tab, check the **Recent job runs**. Wait for the job to finish with **Run status** as `Succeeded`. 

    ![Glue job status](/static/Glue%20Jobs/Lab%202/glue-job-screens/SUCCESS.png) 

We have now successfully created and run our first Glue job. The job reads data from a CSV file containing NYC taxi data stored in s3 and loads it to a different path in s3.
