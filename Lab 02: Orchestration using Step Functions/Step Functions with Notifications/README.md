# Step Functions with Notifications

## Orchestration using multiple AWS services


In the previous sections of this workshop, we created a simple Step Function state machine to pass parameters to and run a glue job. We then added an EventBridge rule to trigger our state machine based on s3 Put event.
In this section of the workshop, we will expand on the already built state machine, by adding tasks to notify .

**Subsribing to SNS topics**

To be able to receive notifications from our state machine, we will need to subscribe to the SNS topics already created in your account.

1.	Go to [AWS Simple Notification Service](https://us-east-2.console.aws.amazon.com/sns/). In the navigation pane on the left, click **Topics**, you should see two topics: `glueworkshop-sns-failure-notification` and `glueworkshop-sns-success-notification`

2. It is recommended to subscribe to at least `glueworkshop-sns-success-notification` topic. To do so select the topic, then click **Create subscription**

3. For **Protocol**, select `email`, then enter the email address you will be using to receive the notifications in the **Endpoint** field. Click **Create subscription**

![failed state machine](/static/Glue%20Jobs/Lab%203/step-functions-screenshots/lab8-3-0.png)

You should receive an email asking to confirm your subscription to the `glueworkshop-sns-success-notification` topic. Click `Confirm subscription` from the email you received to confirm and be able to receive the glue job SNS notifications.

**Adding Tasks to the Step Function state machine**

1.	Go to [AWS Step Function console](https://us-east-2.console.aws.amazon.com/states/). In the navigation pane on the left, click **States Machine**, then select the state machine you created in the previous step of this workshop.

2. Click on **Edit**, then replace the exisitng state machine definition with the  the following state machine language code 

```JSON
{
  "Comment": "A description of my state machine",
  "StartAt": "nytaxi-glue-job",
  "States": {
    "nytaxi-glue-job": {
      "InputPath": "$",
      "ResultPath": "$.gluejobresponse",
      "Type": "Task",
      "Resource": "arn:aws:states:::glue:startJobRun.sync",
      "Parameters": {
        "JobName": "ny-taxi-transformed-cli-jb",
        "NumberOfWorkers.$": "$.glue_max_capacity_dpus",
        "WorkerType": "G.1X",
        "Arguments": {
          "--JOB_NAME.$": "$.JOB_NAME",
          "--BUCKET_NAME.$": "$.BUCKET_NAME",
          "--SRC_PREFIX.$": "$.SRC_PREFIX",
          "--TRG_PREFIX.$": "$.TRG_PREFIX"
        }
      },
      "Next": "Notification Choice",
      "Catch": [
        {
          "ErrorEquals": [
            "States.ALL"
          ],
          "Next": "Notification Choice",
          "ResultPath": "$.glue_error"
        }
      ]
    },
    "Notification Choice": {
      "Type": "Choice",
      "Choices": [
        {
          "And": [
            {
              "Variable": "$.gluejobresponse.JobRunState",
              "IsPresent": true
            },
            {
              "Variable": "$.gluejobresponse.JobRunState",
              "StringEquals": "SUCCEEDED"
            }
          ],
          "Next": "Data Written Success Message"
        }
      ],
      "Default": "Data Failed to Write Message"
    },
    "Data Written Success Message": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sns:publish",
      "Parameters": {
        "Subject.$": "States.Format('Data successfully written from {} into S3', $.JOB_NAME)",
        "Message.$": "States.Format('Data with parameters: {} was successfully written from {} into S3.\n\n It can be found under path:\n {}',  $.SRC_PREFIX, $.BUCKET_NAME, $.TRG_PREFIX)",
        "TopicArn": "arn:aws:sns:us-east-2:ACCOUNT_NUMBER:lab8-sns-success-notification"
      },
      "Next": "Succeed State"
    },
    "Data Failed to Write Message": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sns:publish",
      "Parameters": {
        "Subject.$": "States.Format(' {} Glue job failed', $.JOB_NAME)",
        "Message.$": "States.Format('Data with parameters: {} failed to write from {} into path {}',  $.BUCKET_NAME, $.SRC_PREFIX, $.SRC_PREFIX)",
        "TopicArn": "arn:aws:sns:us-east-2:ACCOUNT_NUMBER:lab8-sns-failure-notification"
      },
      "Next": "Fail State"
    },
    "Fail State": {
      "Type": "Fail",
      "Error": "ErrorCode",
      "Cause": "Caused By Message"
    },
    "Succeed State": {
      "Type": "Succeed"
    }
  }
}
```

Notice how the state machine updated the visual representation instantly

![failed state machine](/static/Glue%20Jobs/Lab%203/step-functions-screenshots/lab8-3-1.png)


3. Click **Workflow studio** on the right side of the screen.

4. There, you will select `Data Failed to Write Message` task and under the `API Parameters` section, select the SNS `failure-notification` topic ARN 

![failed state machine](/static/Glue%20Jobs/Lab%203/step-functions-screenshots/lab8-3-2.png)


5. Repeat `step 4` for `Data Written Success Message`, using the corresponding SNS `success-notification` topic ARN.

6. **Apply and exit** then **Save** to save the new state machine.

7. To test our state machine, we will be simulating an s3 Put object event by running the following command in the Cloud9 terminal.

```bash
 aws s3 cp  ~/environment/green_tripdata.csv s3://${BUCKET_NAME}/input/lab2/eventdriven/
```

We can see that our state machine ran successfully and a `Success` email notification received, if you chose to subscribe to the SNS Success topic.

![failed state machine](/static/Glue%20Jobs/Lab%203/step-functions-screenshots/lab8-3-3.png)

This concludes the Lab 2 section of this workshop.

You may now proceed to [Lab 03: Orchestration using MWAA](/Lab%2003%3A%20Orchestration%20using%20MWAA/README.md) section of this workshop.