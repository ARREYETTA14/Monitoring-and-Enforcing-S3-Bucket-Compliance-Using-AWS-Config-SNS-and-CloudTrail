# Monitoring-and-Enforcing-S3-Bucket-Compliance-Using-AWS-Config-SNS-and-CloudTrail
This project demonstrates how to integrate AWS Config with Amazon S3, SNS, and CloudTrail to monitor changes in an S3 bucket's configuration, flag it as non-compliant if it becomes public, send an SNS notification to subscribed email addresses, and use CloudTrail to record the API that triggered the change.

1. Enable AWS Config
- Go to the AWS Config Console.
- Click Set up AWS Config.
- Select an S3 bucket to store AWS Config logs or let AWS Config create a bucket for you.
- Choose the recording options:
     - Resource types to record: Select S3 Buckets.
Enable AWS Config rules later in the process.
