# Week 0 — Billing and Architecture

Getting the AWS CLI Working

We'll be using the AWS CLI often in this bootcamp, so we'll proceed to installing this account.

Install AWS CLI

We are going to install the AWS CLI when our Gitpod enviroment lanuches.
We are are going to set AWS CLI to use partial autoprompt mode to make it easier to debug CLI commands.
The bash commands we are using are the same as the [AWS CLI Install Instructions]https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
Update our .gitpod.yml to include the following task.

tasks:
  - name: aws-cli
    env:
      AWS_CLI_AUTO_PROMPT: on-partial
    init: |
      cd /workspace
      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      unzip awscliv2.zip
      sudo ./aws/install
      cd $THEIA_WORKSPACE_ROOT
We'll also run these commands indivually to perform the install manually

Create a new User and Generate AWS Credentials

Go to (IAM Users Console](https://us-east-1.console.aws.amazon.com/iamv2/home?region=us-east-1#/users) andrew create a new user
Enable console access for the user
Create a new Admin Group and apply AdministratorAccess
Create the user and go find and click into the user
Click on Security Credentials and Create Access Key
Choose AWS CLI Access
Download the CSV with the credentials
Set Env Vars

We will set these credentials for the current bash terminal

export AWS_ACCESS_KEY_ID=""
export AWS_SECRET_ACCESS_KEY=""
export AWS_DEFAULT_REGION=us-east-1
We'll tell Gitpod to remember these credentials if we relaunch our workspaces

gp env AWS_ACCESS_KEY_ID=""
gp env AWS_SECRET_ACCESS_KEY=""
gp env AWS_DEFAULT_REGION=us-east-1
Check that the AWS CLI is working and you are the expected user

aws sts get-caller-identity
You should see something like this:


Enable Billing

We need to turn on Billing Alerts to recieve alerts...

In your Root Account go to the Billing Page
Under Billing Preferences Choose Receive Billing Alerts
Save Preferences
Creating a Billing Alarm

Create SNS Topic

We need an SNS topic before we create an alarm.
The SNS topic is what will delivery us an alert when we get overbilled
aws sns create-topic
We'll create a SNS Topic

aws sns create-topic --name billing-alarm
which will return a TopicARN

We'll create a subscription supply the TopicARN and our Email

aws sns subscribe \
    --topic-arn TopicARN \
    --protocol email \
    --notification-endpoint your@email.com

Check your email and confirm the subscription

Create Alarm

aws cloudwatch put-metric-alarm
Create an Alarm via AWS CLI
We need to update the configuration json script with the TopicARN we generated earlier
We are just a json file because --metrics is is required for expressions and so its easier to us a JSON file.

aws cloudwatch put-metric-alarm --cli-input-json file://aws/json/alarm_config.json

Create an AWS Budget

aws budgets create-budget

Get your AWS Account ID

aws sts get-caller-identity --query Account --output text
Supply your AWS Account ID
Update the json files
This is another case with AWS CLI its just much easier to json files due to lots of nested json

# This is using AWS to Create a Budget

aws budgets create-budget \
    --account-id $ACCOUNT_ID \
    --budget file://aws/json/budget.json \
    --notifications-with-subscribers file://aws/json/budget-notifications-with-subscribers.json


# Create SNS Topic

aws sns create-topic --name billing-alarm(Name Of Topic)


# Create SNS Subscribe
aws sns subscribe --topic-arn "arn:aws:sns:ca-central-1:891377298769:billing-alarm" --protocol email --notification-endpoint "yykdigital@hotmail.com"



# Create the Alarm Config


{
  "AlarmName": "DailyEstimatedCharges",
  "AlarmDescription": "This alarm would be triggered if the daily estimated charges exceeds 1$",
  "ActionsEnabled": true,
  # Creating an AlarmActions
  "AlarmActions": [
      "arn:aws:sns:ca-central-1:891377298769:billing-alarm"
  ],
  "EvaluationPeriods": 1,
  "DatapointsToAlarm": 1,
  "Threshold": 1,
  "ComparisonOperator": "GreaterThanOrEqualToThreshold",
  "TreatMissingData": "breaching",
  "Metrics": [{
      "Id": "m1",
      "MetricStat": {
          "Metric": {
              "Namespace": "AWS/Billing",
              "MetricName": "EstimatedCharges",
              "Dimensions": [{
                  "Name": "Currency",
                  "Value": "USD"
              }]
          },
          "Period": 86400,
          "Stat": "Maximum"
      },
      "ReturnData": false
  },
  {
      "Id": "e1",
      "Expression": "IF(RATE(m1)>0,RATE(m1)*86400,0)",
      "Label": "DailyEstimatedCharges",
      "ReturnData": true
  }]
}

# Command Line Interface to Create the CloudWatch Alarm
aws cloudwatch put-metric-alarm --cli-input-json file://aws/json/alarm-config.json


 





