---
chapter: true
title: "Lab 01: Creating your First Glue Job"
weight: 10
---

::alert[Please complete the prerequisite in [How to start?](/howtostart/awseevnt/s3-and-local-file.html) section before start this lab.]

In this lab, we will configure AWS Glue job to read data from our NYC taxi dataset and write it back to s3 in a partitioned way. AWS Glue job will be configured to receive 4 job parameters.

<!-- ![Lab 1 Architecture](/static/lab1/lab1_architecture.png) -->

There are multiple ways to create a Glue job. In this workshop, we will be [Using AWS CLI](/lab1/create-job-cli.html) but you can also try [Using AWS Console](/lab1/create-job-console.html)

