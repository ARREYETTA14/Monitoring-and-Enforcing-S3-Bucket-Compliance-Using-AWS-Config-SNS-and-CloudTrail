# Monitoring-and-Enforcing-S3-Bucket-Compliance-Using-AWS-Config-SNS-and-CloudTrail
This project demonstrates how to integrate AWS Config with Amazon S3, SNS, and CloudTrail to monitor changes in an S3 bucket's configuration, flag it as non-compliant if it becomes public, send an SNS notification to subscribed email addresses, and use CloudTrail to record the API that triggered the change.

**1. Enable AWS Config**
- Go to the **AWS Config Console**.
- Click Set up **AWS Config**.
- choose **Recording strategy**: **Specific Resource Type**.
- Choose the **recording options**:
     - Resource types to record: Select **S3 Buckets**. Choose either it should be **continuous** or **Daily** monitoring.
 
![Image](https://github.com/user-attachments/assets/80a6bcd7-5641-444f-ae56-dd69c0a2b15b)

- Data Governance Section, choose **Create AWS Config service-linked role**

![Image](https://github.com/user-attachments/assets/8d064cd6-bd84-4e72-96d8-1fd5ee74e321)

- Delivery Method Section, Select an S3 bucket to store AWS Config logs or let AWS Config create a bucket for you and hit **Next**.

![Image](https://github.com/user-attachments/assets/93c0814a-2751-45f0-b581-1c57678aff43)

- Enable **AWS Config rules** that flag the bucket as non-compliant if it becomes public. ```s3-bucket-public-read-prohibited``` then hit next

![Image](https://github.com/user-attachments/assets/aa8858f6-1a47-4ba6-9cf3-23c06efa054b)

- Review and **confirm**.

**2. Create an SNS Topic**
- Go to the **Amazon SNS Console**.
- Click pass **topic name** and hit **next step**.
- Select **Standard** and click **create topic**.
- Once the topic is created, click **Create subscription**:
     - **Protocol**: Email.
     - **Endpoint**: Enter the email address to receive alerts.

- Click **Create subscription** and **Confirm** the subscription by clicking the confirmation link sent to the email.

![Image](https://github.com/user-attachments/assets/06982bee-e117-4c2a-baac-dae2ff2d8a75)


**NB**: In the **SNS topic console**, select the **topic menu on the left**, and click on **edit**. At the level of the **Access Policy**, paste the following script. Replace your current policy with this (adjust the ```region```, ```account-id```,```topic-name``` and ```SourceOwner``` accordingly):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "config.amazonaws.com"
      },
      "Action": "sns:Publish",
      "Resource": "arn:aws:sns:sa-east-1:211125453684:tester"
    },
    {
      "Sid": "__default_statement_ID",
      "Effect": "Allow",
      "Principal": {
        "AWS": "*"
      },
      "Action": [
        "SNS:GetTopicAttributes",
        "SNS:SetTopicAttributes",
        "SNS:AddPermission",
        "SNS:RemovePermission",
        "SNS:DeleteTopic",
        "SNS:Subscribe",
        "SNS:ListSubscriptionsByTopic",
        "SNS:Publish"
      ],
      "Resource": "arn:aws:sns:sa-east-1:211125453684:tester",
      "Condition": {
        "StringEquals": {
          "AWS:SourceOwner": "211125453684"
        }
      }
    }
  ]
}
```


**3. Enable CloudTrail**
- Go to the **CloudTrail Console**.
- Click **Create trail**.
- Enter a name for the trail, e.g., ```S3ChangeTrail```.
- It automatically creates a bucket for you to store the logs.
- Click **create trail**.

![Image](https://github.com/user-attachments/assets/eaa3c51f-aaa7-493f-8653-655f1edeca1c)

**4. Set Up Notifications for AWS Config**
- Go to **AWS Config Console**.
- Under **Settings**, click **edit** go to the **SNS Topic** section.
- Choose the SNS topic created earlier (S3-Config-Alerts) for notifications from **your account** then **save**.

**5. Testing the Integration**
- Create an S3 bucket:
- Modify the bucket's ACL or policy to allow public access:
     - Go to Bucket Policy Section:
          - In the same Permissions tab, scroll to the Bucket Policy section.
          - Click Edit.
     - Add a Public Access Policy:
          - Pass a bucket policy allowing public read access. For example
            
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

![Image](https://github.com/user-attachments/assets/08418e95-f79b-426d-8c62-5494ef4fd298)
 
- Check your email:
     - An SNS notification is sent to the subscribed email address.

![Image](https://github.com/user-attachments/assets/ac31c51a-d369-44ed-bad8-c861e52260cf)

- Check CloudTrail:
     - Go to the **CloudTrail Console**.
     - Navigate to the **trail** you created to store the logs, click on the **s3 bucket**, navigate to the log you desire and **download** the log to query to know which user did the act.

![Image](https://github.com/user-attachments/assets/e62599fb-5d40-4674-9af6-dabf99a070ac)


