# Create Glue Job - Console (Optional)

::alert[Please complete the prerequisite [How to start?](/howtostart/awseevnt/s3-and-local-file.html) section before starting this lab.]

In this section of the lab we will create, execute and verify the results of an AWS Glue job for ingesting the data from a csv file stored on s3.

**Prepare S3 bucket**

During the workshop environment setup, an Amazon S3 bucket was created for storing lab files and CloudTrail logs. 

**Create the Glue job to load data from the csv file**

Now that we have the source code file for the Glue job, follow the instructions below to create and run a Glue job with this source code file. 

**Enable Job Bookmark and run the Glue job**


1. Go to the [AWS Glue console](https://us-east-2.console.aws.amazon.com/glue/), click **ETL jobs** on the left, select `Spark script editor` then `Create`.


2. Copy the following script to replace the existing one in glue.

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

3. Navigate to `Job details` and paste `ny-taxi-transformed-console-jb` as the name for the job.

4. For the **IAM Role** select `AWSGlueServiceRole-glueworkshop` from the drop down list

5. Expand the **Advanced properties** section and change the **Maximum concurrency** value to `3`

6. Scroll all the way to the **Job parameters** section, hit `Add new parameter` then for the key field add 

```bash
--BUCKET_NAME
```
for the value field add your account number to the following parameter value:

:::alert{header="Important" type="warning"}
Change **YOUR_ACCOUNT_NUMBER** to your specific account number
:::

```bash
glueworkshop-YOUR_ACCOUNT_NUMBER-us-east-2
```

7. Repeat step 7 for the following three parameters:
```bash
--SRC_PREFIX : data/raw/step-green
--TRG_PREFIX : target/glue-transformed
--JOB_NAME   : glue-console-job
```

8. Click on `Save` button at the top right corner of the job editor panel to save the changes.

    ::alert[Please inspect your job configuration, it should look similar to this Job]

    ![Glue job](/static/Glue%20Jobs/Lab%202/glue-job-screens/gluejob-config1.jpeg)
    ![Glue job](/static/Glue%20Jobs/Lab%202/glue-job-screens/gluejob-advance-config.png)
    ![Glue job](/static/Glue%20Jobs/Lab%202/glue-job-screens/gluejob-advance-parameters.png)

9. Click on `Run` and navigate to the **Runs** tab to inspect the latest job execution. Under **Recent job runs**, Wait for the job to finish with **Run status** as `Succeeded`. 


We have now successfully created and run our first Glue job. The job reads data from a CSV file containing NYC taxi data stored in s3 and loads it to a different path in s3.
