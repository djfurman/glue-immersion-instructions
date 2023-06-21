# Prepare S3 Bucket and Clone Files

We will use [AWS Cloud9](https://aws.amazon.com/cloud9/) to run shell commands, edit and run Python scripts for the labs. Cloud9 is a cloud-based integrated development environment (IDE) that lets you write, run, and debug your code with just a browser. It combines the rich code editing features of an IDE such as code completion, hinting, and step-through debugging, with access to a full Linux server for running and storing code.

**Prepare Cloud9 Variables, Local Directories, Files & Workshop Configurations**

**1.** Go to the [AWS Cloud9 console](https://us-east-2.console.aws.amazon.com/cloud9/) in your environment and you should see a Cloud9 with name `glueworkshop`. Click **Open IDE** button to enter Cloud9 IDE environment. 

![Cloud 9 console](/static/howtostart/awseevnt/s3-and-local-file/cloud9-1.png)

**2.** In Cloud9, click on the menu bar **Window** and then **New Terminal**. This will create a new tab and load a command-line terminal. You will use this terminal window throughout the lab to execute the AWS CLI commands and scripts.

![new Cloud9 terminal](/static/howtostart/awseevnt/s3-and-local-file/cloud9-2.png)

**3.** Copy below commands (*always use the tiny **copy icon** on the top-right-corner of the code block!!!*) and paste it in your **Cloud9 Command Line Terminal**:

> **Note:**
> The below commands will automatically download and execute a script called **one-step-setup.sh**. The script name speaks for itself, by run it, all the required pre-steps will be automatically configured for this workshop.


~~~shell
cd ~/environment/

echo "==================> DOWNLOADING & EXECUTING THE ONE-STEP-SETUP SCRIPT <====================
$(curl -s 'https://static.us-east-1.prod.workshops.aws/51892261-738f-43d3-85b8-cac1355bb467/static/download/howtostart/awseevnt/s3-and-local-file/one-step-setup.sh?Key-Pair-Id=K36Q2WVO3JP7QD&Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9zdGF0aWMudXMtZWFzdC0xLnByb2Qud29ya3Nob3BzLmF3cy81MTg5MjI2MS03MzhmLTQzZDMtODViOC1jYWMxMzU1YmI0NjcvKiIsIkNvbmRpdGlvbiI6eyJEYXRlTGVzc1RoYW4iOnsiQVdTOkVwb2NoVGltZSI6MTY4Nzk2ODQxN319fV19&Signature=CodBQ5BVX~2RMe28fQHZSAyjygRRylVTnbAYYGGUjUOR93UTGEdFX7IvpGq8YvMcIne5ot06PI~UwH6QqUAMJYRFTW3XOEuS58FSlAQz522el1CCxJAwPPJOBcwj7YdOJRfW7ytGZ7pnwciLoHb7nvWLkDPRgqsfNrc~OvbLKShm~SnByxL3B88cfeL4rhMdtYIvCvhUSYuigeOKGZh6h-3O8vm4GadmTP9rMcZQtOnye4iDjl2PAUs5cIMpkMzPac9TWTjGsSKyK5qYhigpVPXPm-IAXzzRvhjub7q6PfGwZe7dy-Ngnn1zKLSisHcUd41KSE-OjcyWdegA671oow__' --output ~/environment/glue-workshop/library/one-step-setup.sh --create-dirs)
==========================================================================================="

. ./glue-workshop/library/one-step-setup.sh  'https://static.us-east-1.prod.workshops.aws/public/f7d69c1d-5f04-4b2c-bd7a-e77e4892bbb6/static/0/'



~~~

> **Note**
> We can verify that our environmental variables have been successfully configured by running the following commands :::

```bash
echo ${BUCKET_NAME}
echo ${AWS_REGION}
echo ${AWS_ACCOUNT_ID}
```

After setting our environmental variables, we need to create our S3 paths and copy the workshop files we will use during the workshop. Copy and run the following AWS CLI commands into your cloud9 terminal. 

~~~shell
aws s3api put-object --bucket ${BUCKET_NAME} --key data/raw/step-green/
aws s3api put-object --bucket ${BUCKET_NAME} --key trainingday-scripts/
aws s3api put-object --bucket ${BUCKET_NAME} --key target/
aws s3api put-object --bucket ${BUCKET_NAME} --key input/lab2/eventdriven/
aws s3api put-object --bucket mwaa-${AWS_ACCOUNT_ID}-us-east-2 --key dags/

aws sns delete-topic --topic-arn arn:aws:sns:us-east-2:${AWS_ACCOUNT_ID}:lab8-sns-failure-notification
aws sns delete-topic --topic-arn arn:aws:sns:us-east-2:${AWS_ACCOUNT_ID}:lab8-sns-success-notification
aws sns create-topic --name glueworkshop-sns-failure-notification
aws sns create-topic --name glueworkshop-sns-success-notification

$(curl -s https://raw.githubusercontent.com/MazenAB/Glue-immersion-day-MWAA/main/green_tripdata.csv --output ~/environment/green_tripdata.csv --create-dirs)

aws s3 cp green_tripdata.csv s3://${BUCKET_NAME}/data/raw/step-green/

$(curl -s https://raw.githubusercontent.com/MazenAB/Glue-immersion-day-MWAA/main/FurtherRequiredInlinePolicy.json --output ~/environment/FurtherRequiredInlinePolicy.json --create-dirs)

/bin/sed -i "s/AWS_ACCOUNT_ID/${AWS_ACCOUNT_ID//./_}/g" FurtherRequiredInlinePolicy.json

aws iam  put-role-policy --role-name AWSEC2ServiceRole-etl-ttt-demo --policy-name FurtherRequiredInlinePolicy --policy-document file://~/environment/FurtherRequiredInlinePolicy.json

aws cloudtrail put-event-selectors --trail-name glueworkshop-trail --event-selectors '[{"ReadWriteType": "WriteOnly","IncludeManagementEvents": false,"DataResources": [{"Type":"AWS::S3::Object","Values": ["arn:aws:s3:::'"$BUCKET_NAME"'/input/lab2/eventdriven/"]}]}]'

$(curl -s https://raw.githubusercontent.com/MazenAB/Glue-immersion-day-MWAA/main/gluemwaatemplate-git.yaml --output ~/environment/gluemwaatemplate.yaml --create-dirs)

aws cloudformation deploy --template-file  ~/environment/gluemwaatemplate.yaml --stack-name Mwaa-env --capabilities CAPABILITY_NAMED_IAM

~~~



You are finished setting up the workshop environment and can move on to [Lab 01: Creating your First Glue Job](/Lab%2001%3A%20Creating%20your%20First%20Glue%20Job/README.md) now.