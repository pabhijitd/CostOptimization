AWS Serverless Cost Optimization Automation

• Developed a serverless AWS solution using Lambda, SNS, EventBridge, CloudWatch, and Python (Boto3).
• Automated detection of unattached EBS volumes to identify cost optimization opportunities.
• Configured SNS-based email alerts and EventBridge scheduled execution.
• Troubleshot and resolved IAM authorization issues related to EC2 and SNS APIs.
• Improved cloud governance through automated resource monitoring.


# AWS Serverless Cost Optimization Automation

## Automated Detection of Unattached EBS Volumes Using AWS Lambda, SNS, and EventBridge

### 📌 Project Overview

Unattached Amazon EBS volumes continue to incur storage costs even when they are no longer associated with EC2 instances. These orphaned resources often remain unnoticed and contribute to unnecessary cloud spending.

This project implements a **serverless AWS automation solution** that automatically detects unattached EBS volumes and sends email notifications to administrators using Amazon SNS.

---

## 🚀 Features

- Detects unattached EBS volumes automatically
- Sends email alerts using Amazon SNS
- Runs on a scheduled basis using Amazon EventBridge
- Uses AWS Lambda and Python (Boto3)
- Serverless architecture with minimal operational overhead
- Improves AWS cost visibility and governance

---

## 🏗️ Solution Architecture

```text
                Amazon EventBridge
                        │
                        ▼
               AWS Lambda Function
             (UnusedEBSVolumeChecker)
                        │
                        ▼
                 EC2 DescribeVolumes
                        │
                        ▼
          Filter Volumes in "Available" State
                        │
                        ▼
                   Amazon SNS
                        │
                        ▼
                 Email Notification
```

---

## 🛠️ Technologies Used

- AWS Lambda
- Amazon EC2
- Amazon SNS
- Amazon EventBridge
- AWS IAM
- Amazon CloudWatch
- Python 3.12
- Boto3 SDK

---

# Implementation Steps

## Step 1: Create SNS Topic

Created an SNS topic to send notifications.

```text
Amazon SNS
→ Topics
→ Create Topic
```

Configuration:

```text
Topic Name:
stale-resource-alerts

Type:
Standard
```

Created an email subscription and confirmed it through the verification email.

---

## Step 2: Create Lambda Function

Created a Lambda function to execute the resource scanning logic.

```text
AWS Lambda
→ Create Function
```

Configuration:

```text
Function Name:
UnusedEBSVolumeChecker

Runtime:
Python 3.12
```

---

## Step 3: Configure Environment Variables

Added SNS Topic ARN as an environment variable.

```text
Lambda
→ Configuration
→ Environment Variables
```

Variable:

```text
SNS_TOPIC_ARN=<SNS Topic ARN>
```

Example:

```text
arn:aws:sns:ap-south-1:123456789012:stale-resource-alerts
```

---

## Step 4: Implement Lambda Function

The Lambda function performs the following tasks:

1. Connects to EC2 using Boto3
2. Retrieves all EBS volumes
3. Filters unattached volumes (`available` state)
4. Generates a notification message
5. Publishes the message to SNS

### Lambda Code

```python
import boto3
import os

ec2 = boto3.client('ec2')
sns = boto3.client('sns')

SNS_TOPIC_ARN = os.environ['SNS_TOPIC_ARN']

def lambda_handler(event, context):

    unused_volumes = []

    volumes = ec2.describe_volumes(
        Filters=[
            {
                'Name': 'status',
                'Values': ['available']
            }
        ]
    )

    for volume in volumes['Volumes']:

        volume_id = volume['VolumeId']
        size = volume['Size']

        unused_volumes.append(
            f"Volume ID: {volume_id}, Size: {size} GB"
        )

    if unused_volumes:

        sns.publish(
            TopicArn=SNS_TOPIC_ARN,
            Subject='Unused EBS Volumes Found',
            Message="\n".join(unused_volumes)
        )

    return {
        'unused_volumes_found': len(unused_volumes)
    }
```

---

## Step 5: Create Test EBS Volume

To validate the solution:

```text
EC2
→ Volumes
→ Create Volume
```

Configuration:

```text
Type: gp3
Size: 5 GB
```

The volume was intentionally left unattached.

Expected State:

```text
Available
```

---

## Step 6: Configure Test Event

Created a simple Lambda test event.

```json
{}
```

Executed the Lambda function.

---

# Challenges and Troubleshooting

## Issue #1: EC2 Permission Denied

### Error

```text
UnauthorizedOperation

User is not authorized to perform:
ec2:DescribeVolumes
```

### Root Cause

Lambda execution role lacked EC2 read permissions.

### Resolution

Added IAM permission:

```json
{
    "Effect": "Allow",
    "Action": "ec2:DescribeVolumes",
    "Resource": "*"
}
```

Alternatively attached:

```text
AmazonEC2ReadOnlyAccess
```

### Result

✅ Lambda successfully retrieved volume information.

---

## Issue #2: SNS Publish Permission Denied

### Error

```text
AuthorizationError

User is not authorized to perform:
sns:Publish
```

### Root Cause

Lambda execution role lacked permissions to publish to SNS.

### Resolution

Added IAM permission:

```json
{
    "Effect": "Allow",
    "Action": "sns:Publish",
    "Resource": "*"
}
```

### Result

✅ Lambda successfully published notifications to SNS.

---

# IAM Permissions

### Custom Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeVolumes"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "sns:Publish"
      ],
      "Resource": "*"
    }
  ]
}
```

### AWS Managed Policy

```text
AWSLambdaBasicExecutionRole
```

Used for CloudWatch logging.

---

## Step 7: Verify Email Notification

After permissions were corrected, Lambda executed successfully and sent an email.

### Sample Notification

```text
Subject:
Unused EBS Volumes Found

Volume ID: vol-0123456789abcdef

Size: 5 GB
```

---

## Step 8: Schedule Daily Execution

Configured EventBridge to automate scanning.

```text
Amazon EventBridge
→ Rules
→ Create Rule
```

Configuration:

```text
Name:
DailyUnusedEBSCheck

Schedule:
rate(1 day)

Target:
UnusedEBSVolumeChecker
```

---

## Step 9: Validate CloudWatch Logs

Used CloudWatch Logs for troubleshooting and monitoring.

```text
CloudWatch
→ Log Groups
→ /aws/lambda/UnusedEBSVolumeChecker
```

Verified:

- Function execution
- IAM errors
- SNS publishing activities
- Invocation metrics

---

# Project Testing

### Test Scenario

| Component | Result |
|------------|---------|
| Lambda Execution | ✅ Success |
| EC2 Volume Discovery | ✅ Success |
| SNS Notification | ✅ Success |
| Email Alert | ✅ Success |
| EventBridge Schedule | ✅ Configured |
| CloudWatch Logging | ✅ Success |

---

# Business Benefits

- Proactively identifies unused AWS storage resources
- Reduces cloud waste and unnecessary costs
- Eliminates manual infrastructure audits
- Improves cloud governance
- Demonstrates serverless automation

---

# Future Enhancements

This solution can be extended to detect:

- Unused EBS Snapshots
- Unused Elastic IP Addresses
- Unused AMIs
- Idle Load Balancers
- Unused Security Groups

Possible future integrations:

- Slack Notifications
- Microsoft Teams Alerts
- Cost-Saving Reports
- AWS Organizations Multi-Account Support
- Automated Cleanup Workflow

---

# Key Learnings

- AWS Lambda Development
- EventBridge Scheduling
- Amazon SNS Notifications
- IAM Permission Management
- CloudWatch Monitoring
- Python Automation with Boto3
- AWS Cost Optimization
- Serverless Architecture
- Troubleshooting Access Denied Issues

---

# Resume Highlights

- Designed and implemented a serverless AWS cost optimization solution using Lambda, EventBridge, SNS, IAM, CloudWatch, and Python (Boto3).
- Automated identification of unattached Amazon EBS volumes across the AWS account.
- Configured scheduled scans using EventBridge and automated notifications through SNS.
- Diagnosed and resolved IAM permission issues related to EC2 and SNS APIs.
- Improved cloud resource visibility and reduced manual auditing effort through automation.

---

# Project Status

✅ Completed

✅ Successfully Tested

✅ Suitable for AWS Cloud Engineer / DevOps Engineer Resume Showcase
