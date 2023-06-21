# Lab 02: Orchestration using Step Functions

> **Alert**
> Please complete the prerequisite in [How to start?](/howtostart/awseevnt/s3-and-local-file.html) and [Lab 1](/lab1.html) sections before starting this lab.

**AWS Step Functions**

[AWS Step Functions](https://aws.amazon.com/step-functions/) is a low-code visual workflow service used to orchestrate AWS services, automate business processes, and build serverless applications. Workflows manage failures, retries, parallelization, service integrations, and observability so developers can focus on higher-value business logic.

Step Functions is based on state machines and tasks. A state machine is a workflow. A task is a state in a workflow that represents a single unit of work that another AWS service performs. Each step in a workflow is a state.

Orchestrating a series of individual serverless applications, managing retries, and debugging failures can be challenging. As your distributed applications become more complex, the complexity of managing them also grows. With its built-in operational controls, Step Functions manages sequencing, error handling, retry logic, and state, removing a significant operational burden from your team.

You can now proceed to create [your first Step Function State Machine](/Lab%2002%3A%20Orchestration%20using%20Step%20Functions/Create%20First%20Step%20Function/README.md)