# Event Driven Orchestration

**Amazon EventBridge**

[Amazon EventBridge](https://aws.amazon.com/eventbridge/) is a serverless event bus service that you can use to connect your applications with data from a variety of sources. EventBridge delivers a stream of real-time data from your applications, software as a service (SaaS) applications, and AWS services to targets such as AWS Lambda functions, HTTP invocation endpoints using API destinations, or event buses in other AWS accounts. EventBridge receives an event, an indicator of a change in environment, and applies a rule to route the event to a target. Rules match events to targets based on either the structure of the event, called an event pattern, or on a schedule.

In this section, we demonstrate how an S3 event generated when new files are uploaded to a specific S3 location can trigger execution of workflow. 

> **NOTE**
> If you want to start a workflow with Amazon S3 data events, you must ensure that events for the S3 bucket of interest are logged to AWS CloudTrail and EventBridge. A new CloudTrail for S3 events is created as part of workshop environment. If you want to use the same S3 event pattern in your own environment, you must create a CloudTrail trail. For more information, see [Creating a trail for your AWS account](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-and-update-a-trail.html).

**Event driven Orchestration**

1. We will create a new rule with name `glueworkshop-rule` in Amazon EventBridge that specifies Amazon **S3** as the event source, **PutObject** as the event name, and the lab bucket name as a request parameter with prefix pointing to the folder we want to monitor which is `s3://${BUCKET_NAME}/input/lab2/eventdriven/`. Whenever a new object is put in the S3 bucket folder, the rule will trigger its targets, where in our case, the Step Functions state machine. Run the following command in Cloud9 terminal.

```bash
aws events put-rule \
    --name "glueworkshop-rule" \
    --event-pattern "{ \
                        \"source\": [\"aws.s3\"], \
                        \"detail-type\": [\"AWS API Call via CloudTrail\"], \
                        \"detail\": { \
                            \"eventSource\": [\"s3.amazonaws.com\"], \
                            \"eventName\": [\"PutObject\"], \
                            \"requestParameters\": { \
                                \"bucketName\": [\"${BUCKET_NAME}\"], \
                                \"key\": [{\"prefix\": \"input/lab2/eventdriven/\"}]
                            } \
                        } \
                    }"
```

2. Next, we will add the Step Function state machine created earlier as the target to the EventBridge rule `glueworkshop-rule`. Run the following command in Cloud9 terminal.

```bash
aws events put-targets \
    --rule glueworkshop-rule \
    --targets "Id"="stepfunction","Arn"="arn:aws:states:${AWS_REGION}:${AWS_ACCOUNT_ID}:stateMachine:MyStateMachine","RoleArn"="arn:aws:iam::${AWS_ACCOUNT_ID}:role/AWSEventBridgeInvokeRole-glueworkshop" \
    --region ${AWS_REGION}
```
3. Go to the [Amazon EventBridge Console](https://us-east-2.console.aws.amazon.com/events/), click **Rules** on the left, click on `glueworkshop-rule` and you will see the details of the rule. Scroll down and you should see a target with type **Step Functions state machine**. Every time this rule receives a matching event, it will trigger the targets.

![Rule targets](/static/Glue%20Jobs/Lab%203/step-functions-screenshots/lab8-2-1.png)

**Filtering Event Elements**

1. Go to the [Amazon EventBridge Console](https://us-east-2.console.aws.amazon.com/events/), click **Rules** on the left, click on `glueworkshop-rule` and you will see the details of the rule. Select the **Targets** tab and then `edit` to edit the target

![failed state machine](/static/Glue%20Jobs/Lab%203/step-functions-screenshots/lab8-2-2.png)

7. Expand the `Additional settings` section and select `Input transformer` from the `Configure target input` section then hit `Configure input transformer`.

![failed state machine](/static/Glue%20Jobs/Lab%203/step-functions-screenshots/lab8-2-3.png)

8. Scroll down to the **Target input transformer** and under `Input path` section, add the following 
```JSON
{
  "Bucket_Name": "$.detail.requestParameters.bucketName",
  "SRC_PREFIX": "$.detail.requestParameters.key"
}
```
![Rule targets](/static/Glue%20Jobs/Lab%203/step-functions-screenshots/lab8-2-5.png)

for the `Template` section, add the following

```JSON
{
  "glue_max_capacity_dpus": 2,
  "JOB_NAME": "ny-taxi-transformed-console-jb",
  "BUCKET_NAME": "<Bucket_Name>",
  "SRC_PREFIX": "<SRC_PREFIX>",
  "TRG_PREFIX": "target/event-transformed"
}
```

![Rule targets](/static/Glue%20Jobs/Lab%203/step-functions-screenshots/lab8-2-6.png)

Hit `Confirm`, `Next`, `Next` then `Update Rule`


9. We will trigger our target rule again by running the following command in Cloud9 terminal.

```bash
aws s3 cp  ~/environment/green_tripdata.csv s3://${BUCKET_NAME}/input/lab2/eventdriven/
```

Our Step Function state machine should now run successfully using the parameters that were passed by our EventBridge rule


![failed state machine](/static/Glue%20Jobs/Lab%203/step-functions-screenshots/lab8-2-4.png)


You may now proceed to the [Step Functions with Notifications](/Lab%2002%3A%20Orchestration%20using%20Step%20Functions/Step%20Functions%20with%20Notifications/README.md) section of this lab.