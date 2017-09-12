# DynamoDB Capacity Scheduler

A Lambda function that scales DynamoDB table capacity up or down on a schedule, using a cron-like format.

## How it works

For each invocation of a CloudWatch Event (default: every minute):

1. Download scaling plans from a private S3 bucket
2. Determine which policy in each plan should be used based on the current date and time
3. Check that each table is active (i.e. not currently undergoing a scaling event)
4. Check that the current capacity values do not already match the policy
5. Update the table's capacity

## Setup
Clone this repository. Then create a private S3 bucket to hold a JSON file for each table whose capacity you want to control, called a _scaling plan_. A plan specifies the table name, whether the plan is enabled or not and at least two _policies_. A policy has a name, a schedule in crontab format, plus minimum, maximum and target values for both read and write capacity. See `example-scaling-plan.json` for the format.

A scaling plan can have multiple policies. The Lambda function will take the most recently passed schedule when determining which policy to use, so that a missed scaling attempt (e.g. from a failed API call) will correct itself on the next invocation.

In your cloned repo, edit `cloudformation/parameters.json`. Change `SchedulerS3Bucket`'s value to match the S3 bucket you created. Edit `cloudformation/config.yaml` and do the same thing for `scheduler_s3_bucket`'s value.

Run `make init` and `make create-stack`. Once the Cloudformation stack is created, copy the ARN of the Lambda function from the stack outputs and run (with the actual ARN): 
`ARN=arn:aws:lambda:us-west-2:111111111111:function:my-function-name make deploy`

## Local Development
This repository is based on the excellent [Local Lambda Toolkit](https://github.com/dliggat/local-lambda-toolkit) by `dliggat`, which has an overview of the directory structure and `Makefile` used here. In addition, it allows local Lambda invocation and unit test development. Well worth a look!


