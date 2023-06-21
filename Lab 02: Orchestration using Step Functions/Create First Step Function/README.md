# Create First Step Function

::alert[Please complete the prerequisite [How to start?](/Lab%2000%3A%20Login%20and%20Initial%20Setup/README.md) and [Lab 01: Creating your First Glue Job](/Lab%2001%3A%20Creating%20your%20First%20Glue%20Job/README.md) sections before starting this lab.]


In this section of the lab, we will show you how to use the graphic user interface in Step Function to build a workflow to manage the execution of multiple Glue jobs and Glue crawlers. 

You are looking to use AWS Step functions when you are looking to build orchestration with multiple AWS Services along with AWS Glue. 

**Building Step Function Workflow**

1. Go to [AWS Step Function console](https://us-east-2.console.aws.amazon.com/states/). In the navigation pane on the left, click **States Machine**, then click **Create States Machine**.

![Create state machine](/static/Glue%20Jobs/Lab%203/step-functions-screenshots/lab8-1-0.png)

2.	Choose **Design your workflow visually** and choose **Type** as `Standard`.

![Define state machine Property](/static/Glue%20Jobs/Lab%203/step-functions-screenshots/lab8-1-1.png)

3. In **Design workflow**, in the **Action** search box tab on the left, type `Glue`. Drag and drop **AWS Glue - StartJobRun** state into the dotted box in the design canvas. 

4. 
    * Under **Configuration** tab on the right:
        * Set **State name** value to `NYC taxi data job`
        * Replace the **API Parameters** sections with:
        ```json
        {
        "JobName":"ny-taxi-transformed-cli-jb",
        "Arguments":{
            "--JOB_NAME.$":"$.JOB_NAME",
            "--BUCKET_NAME.$":"$.BUCKET_NAME",
            "--SRC_PREFIX.$":"$.SRC_PREFIX",
            "--TRG_PREFIX.$":"$.TRG_PREFIX"
                }
            }
        ```
        :::alert{header="Important" type="warning"}
Check the box for the **Wait for task to complete - optional** . Without this checked, the state machine will not wait on a response back from the glue job
:::

![Glue job state](/static/Glue%20Jobs/Lab%203/step-functions-screenshots/lab8-1-3.png)

5. Hit `Next` to move to the following **Review generated code** page. Inspect the code, then hit `Next` again

![Glue job state](/static/Glue%20Jobs/Lab%203/step-functions-screenshots/lab8-1-4.png)

6. On the **Specify state machine settings** page, change the **Permissions** section to `Choose an existing role` and then select `AWSStepFunctionRole-glueworkshop` from the drop down.

![Glue job state](/static/Glue%20Jobs/Lab%203/step-functions-screenshots/lab8-1-5.png)

7. Hit **Create state machine**, you should now see `State machine successfully created`

8. Select `Start execution` and paste the following JSON parameter set as the State Machine's input

:::alert{header="Important" type="warning"}
Change **REPLACE_ACCOUNT_NUMBER** to your corresponding account number of the **BUCKET_NAME** parameter!
:::

```json
{
    "JOB_NAME": "glue-step-job",
    "BUCKET_NAME": "glueworkshop-REPLACE_ACCOUNT_NUMBER-us-east-2",
    "SRC_PREFIX": "data/raw/step-green",
    "TRG_PREFIX" : "target/step-transformed"
}
```
![Glue job state](/static/Glue%20Jobs/Lab%203/step-functions-screenshots/lab8-1-6.png)

After a few minutes, we can see that our State Machine completed successfully. Feel free to inspect the different execution tabs or selecting the task to drill down on the task details

![Glue job state](/static/Glue%20Jobs/Lab%203/step-functions-screenshots/lab8-1-7.png)


You may now proceed to the [Event Driven Orchestration](/Lab%2002%3A%20Orchestration%20using%20Step%20Functions/Event%20Driven%20Orchestration/README.md) section of this lab.