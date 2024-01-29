# Week 0 — Billing and Architecture

### Creating Budget Alarm

```
aws budgets create-budget \
    --account-id 975050062056 \
    --budget file://aws/json/budget.json \
    --notifications-with-subscribers file://aws/json/budget-notifications-with-subscribers.json
```