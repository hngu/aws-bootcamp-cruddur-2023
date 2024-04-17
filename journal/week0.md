# Week 0 — Billing and Architecture

### Creating Budget Alarm

```
aws budgets create-budget \
    --account-id 975050062056 \
    --budget file://aws/json/budget.json \
    --notifications-with-subscribers file://aws/json/budget-notifications-with-subscribers.json
```

### Create Billing Alarm

Create a SNS topic:
```
aws sns create-topic --name billing-alarm
```
NOTE - SNS: Simple Notification Service

The command should create a topic ARN which you will pass to:
```
aws sns subscribe \
    --topic-arn "TopicARN" \
    --protocol email \
    --notification-endpoint hngu@wistia.com
```
It should return pending confirmation. When you add a notification receipient like email, it wants to verify it.
Make sure you choose the right AWS region then go to SNS -> Topics to view the pending confirmation. Then confirm it via the email it sent.

After that create the cloudwatch alarm:
```
aws cloudwatch put-metric-alarm --cli-input-json file://aws/json/alarm-config.json
```

# My Notes

### Prerequisites

1. Create a github account
    1. Enable MFA under password and authentication to make it harder for someone to hack into your account
1. Create a gitpod account by signing up with your github account
    1. Not sure what it is for?
1. Install the gitpod chrome extension for easier navigation
1. Make sure you can access github codespaces. Github codespaces is a cloud dev environment where you can write, run and test your code all in your browser instead of your local environment
1. Always review your github account’s 3rd party integrations and clean up any unnecessary ones you may have
1. Create a free 12 month AWS account
    1. Remember to close the account after 12 months
    1. Setup MFA immediately
    1. Credit card is linked here
1. Create a honeycomb.io account for distributed tracing
1. Create a rollbar account for logging application errors, crashes for debugging applications
1. Have a password manager setup

### Introduction


