# Monitoring-and-Enforcing-S3-Bucket-Compliance-Using-AWS-Config-SNS-and-CloudTrail
This project demonstrates how to integrate AWS Config with Amazon S3, SNS, and CloudTrail to monitor changes in an S3 bucket's configuration, flag it as non-compliant if it becomes public, send an SNS notification to subscribed email addresses, and use CloudTrail to record the API that triggered the change.

**1. Enable AWS Config**
- Go to the **AWS Config Console**.
- Click Set up **AWS Config**.
- choose **Recording strategy**: **Specific Resource Type**.
- Choose the **recording options**:
     - Resource types to record: Select **S3 Buckets**. Choose either it should be **continuous** or **Daily** monitoring.
- Data Governance Section, choose **Create AWS Config service-linked role**
- Delivery Method Section, Select an S3 bucket to store AWS Config logs or let AWS Config create a bucket for you and hit **Next**.
- Enable **AWS Config rules** that flag the bucket as non-compliant if it becomes public. ```s3-bucket-public-read-prohibited``` then hit next
- Review and **confirm**.

**2. Create an SNS Topic**
- Go to the **Amazon SNS Console**.
- Click pass **topic name** and hit **next step**.
- Select **Standard** and click **create topic**.
- Once the topic is created, click **Create subscription**:
     - **Protocol**: Email.
     - **Endpoint**: Enter the email address to receive alerts.
- Click **Create subscription** and **Confirm** the subscription by clicking the confirmation link sent to the email.

**3. Enable CloudTrail**
- Go to the **CloudTrail Console**.
- Click **Create trail**.
- Enter a name for the trail, e.g., ```S3ChangeTrail```.
- It automatically creates a bucket for you to store the logs.
- Click **create trail**.

**4. Set Up Notifications for AWS Config**
- Go to **AWS Config Console**.
- Under **Settings**, click **edit** go to the **SNS Topic** section.
- Choose the SNS topic created earlier (S3-Config-Alerts) for notifications from **your account** then **save**.

**5. Testing the Intergration**
- Create an S3 bucket:
- Modify the bucket's ACL or policy to allow public access:
     - Go to Bucket Policy Section:
          - In the same Permissions tab, scroll to the Bucket Policy section.
          - Click Edit.
     - Add a Public Access Policy:
          - Replace or update the bucket policy with a policy allowing public read access. For example
            
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::your-bucket-name/*"
        }
    ]
}
```
- Wait for AWS Config to detect the change (this may take a few minutes).
- Check the AWS Config Dashboard:
     - The bucket should be flagged as **non-compliant**.
 images
- Check your email:
     - An SNS notification is sent to the subscribed email address.
images
- Check CloudTrail:
     - Go to the **CloudTrail Console**.
     - Navigate to the bucket that keeps the logs, **download** the log to query to know which user did the act.

images

**NB**: In the SNS topic console, you can select the topic, and click on **push message** to put a customized message to send when the s3 is non-compliant.
for example:
```plaintext
Subject: AWS Config Rule Violation Detected

AWS Config detected a rule violation:
- Resource: S3 Bucket `my-sensitive-bucket`
- Rule: s3-bucket-public-read-prohibited
- Status: NON_COMPLIANT

Details: The bucket was made public, violating compliance policies.

Check AWS Config for more details: https://console.aws.amazon.com/config
```


