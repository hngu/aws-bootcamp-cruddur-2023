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