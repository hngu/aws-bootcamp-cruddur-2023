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
- Building a microservice oriented cloud architecture
- Iron Triangle of Project Management
- Frontend: React
- Backend: Python/Flask
- Build it on AWS and setup budget monitoring
- Investors: inform cost and timing
- CTO: inform technical requirements and future state
- Good Architecture:
    - Requirements, Risks, Assumptions, Constraints
    - Requirements: what you have to achieve. Should be verifiable, or measurable and feasible. 
    - Risks: what can prevent you from being successful. You must mitigate these.
    - Assumptions: factors that are held to be true for the planning and implementation phases
    - Constraints: limitations to the project
    - AWS Well Architectured Tool is good framework to build your napkin design

### Task: Create a Budget
- You get 2 free budgets
- To create a budget, search for Budget
- Then create a zero spend budget for this bootcamp
- You can also set a monthly budget, usage budget, credit budget, etc.
- You can set a threshold where if it hits a percentage of spend, notify you


### Task: Create a new IAM Admin User
- Logged in as the root user, create a new user with IAM
- In the next step, add the user to a group or create a group if there isn’t one. 
- Require the user to reset the password on first login
- You can add an account alias to your AWS account if you like but it must be unique (wistia, wordstream, uc-compass)
- To create an account alias, go to IAM and there should be a  link to create account alias.
- When assigning a role for the user, use the AdminAccess for now, but it may not be the best practice
- Then sign in to that user and reset your password
- Once reset, add MFA key under IAM
- Then create access credentials for this user by clicking on yourself in IAM users
- Then click the create access key. This access key is mostly for the CLI. Download the CSV. You can only have 2 access keys.
- Then go on the main branch of AWS bootcamp (that you cloned) and launch gitpod
- We want to install AWS CLI in gitpod
- The steps to install AWS CLI can be found via Google search, but basically we download a zip file and run the install script from AWS.
- Once that is installed, enable auto prompt via: `aws --cli-auto-prompt`
- When you run `aws sts get-caller-identity` you should get an error since there is no identity set in gitpod
- Take the AWS Access keys and export them (remember to add quotes around the values!)
- In gitpod, we have a `.gitpod.yml` file to setup the aws-cli
- We also use `gp env` to store the env variables

### Task: Create a billing and budget alarm
- In your root account, make sure you have billing alert enabled under billing preferences
- Then create a budget programmatically using AWS CLI aws budgets create-budget
- Create a budget json and a notification with subscribers json (it is in the repo for reference)


### Cloud Security
- Identify and inform the business of any technical vulnerabilities or risks that the business may be exposed to
- Cloud security requires practice
- Step 1: add MFA to root user (root user is the most powerful user)
- Regions: AWS resources are region specific. Start with one region and only branch out if needed
- Step 2: Create AWS Organization unit
- The root user logs in to the management account.
- Create AWS Organization units for each of the Business units (Engineering, Finance) or have two folders: active accounts and standby accounts. The advantage is to segregate resources and cost by function. There are also billing and savings benefits. So have one root AWS account and create child AWS accounts.
- Step 3: Enable AWS Cloud Trail
    - Will cost you money but a few cents for free tier
- Step 4: Setup IAM Users
    - Enable MFA for all human users
    - Principle of least privilege: give the user the least amount of privileges for them to do their job
    - Do not use root user to do things. Use a new IAM user with admin privileges to create new IAM users
    - IAM is a Global service not tied to a region
- Step 5: Setup IAM roles
    - Roles and policies go hand in hand

### Rotating Keys/Scrubbing Sensitive Data
- When you rotate AWS credentials, you may have to run through all the code repositories to make sure those credentials are removed as well. It is generally good practice to do it this way.
- Deactivate the AWS credentials in AWS
- Then you can use open source tools like `trufflehog` and `bfg` to find secrets and replace them with something else


