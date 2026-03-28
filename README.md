# Serverless Architecture Using AWS Lambda and Boto3

---

# Assignment 1: Automated Instance Management Using AWS Lambda and Boto3

## Objective
The objective of this assignment is to gain hands-on experience with **AWS Lambda** and **Boto3** by automating the start and stop operations of **EC2 instances** based on instance tags.

**Technologies used:**
- Amazon EC2
- AWS Lambda
- IAM (Identity and Access Management)
- Boto3 (Python SDK for AWS)

---

## Architecture Overview

Event Trigger → AWS Lambda → Boto3 → EC2 Instance Management

Lambda checks EC2 tags and performs the following:

- Instances tagged **Auto-Stop** → Stop instance  
- Instances tagged **Auto-Start** → Start instance

---

## Step 1: EC2 Instance Setup

1. Login to AWS Management Console  
2. Navigate to **EC2 Dashboard**  
3. Launch two instances with the following configuration:

**Instance Type:** t2.micro

### Instance 1 Tag
| Key    | Value     |
|--------|-----------|
| Action | Auto-Stop |

### Instance 2 Tag
| Key    | Value     |
|--------|-----------|
| Action | Auto-Start |

---

## Step 2: Create IAM Role for Lambda

1. Open IAM Dashboard  
2. Click **Roles** → **Create Role**  
3. Choose trusted entity: AWS Service → Lambda  
4. Attach policy: **AmazonEC2FullAccess**  
5. Role name: `LambdaEC2ManagementRole`

---

## Step 3: Create Lambda Function

1. Navigate to **AWS Lambda** → **Create Function**  

**Configuration:**

| Property       | Value                     |
|----------------|---------------------------|
| Function Name  | ec2-auto-manager          |
| Runtime        | Python 3.x                |
| Execution Role | LambdaEC2ManagementRole   |

---

## Step 4: Lambda Python Code

```python
import boto3

ec2 = boto3.client('ec2')

def lambda_handler(event, context):

    stop_instances = ec2.describe_instances(
        Filters=[{'Name': 'tag:Action','Values': ['Auto-Stop']}]
    )

    stop_ids = []
    for reservation in stop_instances['Reservations']:
        for instance in reservation['Instances']:
            stop_ids.append(instance['InstanceId'])

    if stop_ids:
        ec2.stop_instances(InstanceIds=stop_ids)
        print("Stopped Instances:", stop_ids)

    start_instances = ec2.describe_instances(
        Filters=[{'Name': 'tag:Action','Values': ['Auto-Start']}]
    )

    start_ids = []
    for reservation in start_instances['Reservations']:
        for instance in reservation['Instances']:
            start_ids.append(instance['InstanceId'])

    if start_ids:
        ec2.start_instances(InstanceIds=start_ids)
        print("Started Instances:", start_ids)

    return {
        'statusCode': 200,
        'body': 'EC2 instances processed successfully'
    }
Step 5: Create Test Event

Event Name: TestEvent

Event JSON: {}

Click Test to execute the function

Step 6: Verify Results
Tag	Expected Result
Auto-Stop	Instance Stops
Auto-Start	Instance Starts
Step 7: Logs Monitoring

Logs can be viewed in CloudWatch Logs.

Example log output:

Stopped Instances: ['i-123456']
Started Instances: ['i-789012']
📸 Screenshots
EC2 Instances Created

Lambda Function Created

Lambda Test Event

EC2 Instance State Change

CloudWatch Logs

Result

The AWS Lambda function successfully automated EC2 instance management by detecting instance tags and performing start or stop operations accordingly.

Conclusion

This assignment demonstrated how serverless computing using AWS Lambda combined with Boto3 can automate infrastructure management tasks efficiently. Tag-based automation allows better control and reduces manual intervention in cloud environments.

Assignment 2: Automated S3 Bucket Cleanup Using AWS Lambda and Boto3
📌 Objective

The objective of this assignment is to automate the deletion of files older than 30 days in an Amazon S3 bucket using AWS Lambda and Boto3.

🪣 S3 Bucket Setup

An S3 bucket named my-cleanup-bucket-hero was created.

Multiple files were uploaded into the bucket, including:

Python files (.py)

Text files (.txt)

Some files were used to simulate older data for testing the cleanup process.

📸 Before Deletion

🔐 IAM Role Configuration

An IAM role was created for the Lambda function:

Role Type: AWS Service → Lambda

Permissions Policy: AmazonS3FullAccess

This role allows the Lambda function to:

List objects in the S3 bucket

Delete objects from the bucket

📸 IAM Role

⚙️ Lambda Function Implementation

A Lambda function was created using Python 3.x runtime.

🔧 Function Code
import boto3
from datetime import datetime, timezone, timedelta

s3 = boto3.client('s3')

BUCKET_NAME = 'my-cleanup-bucket-hero'
DAYS_OLD = 2  # Adjusted for testing

def lambda_handler(event, context):
    cutoff_date = datetime.now(timezone.utc) - timedelta(days=DAYS_OLD)

    response = s3.list_objects_v2(Bucket=BUCKET_NAME)

    if 'Contents' not in response:
        print("No objects found.")
        return

    deleted_files = []

    for obj in response['Contents']:
        file_name = obj['Key']
        last_modified = obj['LastModified']

        if last_modified < cutoff_date:
            s3.delete_object(Bucket=BUCKET_NAME, Key=file_name)
            deleted_files.append(file_name)
            print(f"Deleted: {file_name}")

    if not deleted_files:
        print("No files deleted.")
    else:
        print(f"Deleted {len(deleted_files)} files.")

    return {
        'statusCode': 200,
        'body': f"Deleted {len(deleted_files)} files"
    }
📸 Lambda Logs

📸 S3 Bucket After Deletion

Result

The AWS Lambda function successfully deleted files older than the defined threshold while keeping recent files intact.







# Assignment 5: Auto-Tagging EC2 Instances on Launch Using AWS Lambda and Boto3

## Objective
Learn to automate the tagging of EC2 instances as soon as they are launched, ensuring better resource tracking and management.

---

## Architecture Overview

When an EC2 instance is launched:
1. EventBridge (CloudWatch Events) detects the launch
2. It triggers AWS Lambda
3. Lambda uses Boto3 to tag the instance automatically

---

## 1. EC2 Setup

- Verified access to EC2 dashboard
- Successfully able to launch EC2 instances

📸 Screenshot:
*(Insert EC2 dashboard / instance launch screenshot here)*

---

## 2. IAM Role Creation

### Steps:
1. Navigate to IAM Dashboard
2. Click **Roles → Create Role**
3. Select **Lambda** as trusted entity
4. Attach policy:
   - `AmazonEC2FullAccess`
5. Name the role:

Conclusion

This assignment demonstrates how AWS Lambda and Boto3 can be used to automate cloud storage maintenance tasks efficiently. Automating file cleanup reduces manual effort and ensures efficient S3 bucket management.


LambdaEC2TaggingRole


📸 Screenshot:
*(Insert IAM role creation screenshot here)*

---

## 3. Lambda Function Setup

### Configuration:
- Runtime: Python 3.x
- Execution Role: LambdaEC2TaggingRole

📸 Screenshot:
*(Insert Lambda function creation screenshot here)*

---

## 4. Lambda Function Code (Boto3)

```python
import boto3
from datetime import datetime

def lambda_handler(event, context):
 ec2 = boto3.client('ec2')
 
 try:
     # Extract instance ID from event
     instance_id = event['detail']['instance-id']
     
     # Get current date
     current_date = datetime.utcnow().strftime('%Y-%m-%d')
     
     # Create tags
     tags = [
         {
             'Key': 'LaunchDate',
             'Value': current_date
         },
         {
             'Key': 'Environment',
             'Value': 'AutoTagged'
         }
     ]
     
     # Apply tags
     ec2.create_tags(
         Resources=[instance_id],
         Tags=tags
     )
     
     print(f"Successfully tagged instance {instance_id}")
 
 except Exception as e:
     print(f"Error tagging instance: {str(e)}")
     raise

📸 Screenshot:
(Insert Lambda code screenshot here)

5. EventBridge Rule Configuration
Steps:
Navigate to EventBridge (CloudWatch Events)
Click Create Rule
Choose Event Pattern
Select:
Service: EC2
Event Type: EC2 Instance State-change Notification
State: running
Event Pattern JSON:
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["running"]
  }
}
Add Target:
Select Lambda function created earlier

📸 Screenshot:
(Insert EventBridge rule screenshot here)

6. Testing
Steps:
Launch a new EC2 instance
Wait for 10–30 seconds
Navigate to EC2 → Instances → Tags
Expected Output:
LaunchDate = Current Date
Environment = AutoTagged

📸 Screenshot:
(Insert tagged EC2 instance screenshot here)

Result

The Lambda function successfully tagged EC2 instances automatically upon launch using EventBridge trigger and Boto3.

Conclusion

This assignment demonstrates:

Event-driven automation using AWS EventBridge
Serverless execution using AWS Lambda
Automated resource tagging using Boto3









📘 Assignment 7: DynamoDB Item Change Alert Using AWS Lambda, Boto3, and SNS
🎯 Objective

Automate alerts whenever an item in a DynamoDB table is updated using AWS Lambda and SNS.

🛠️ Services Used
AWS DynamoDB
AWS Lambda
AWS SNS (Simple Notification Service)
AWS IAM
Boto3 (Python SDK for AWS)
🏗️ Architecture Diagram
DynamoDB Table
     │
     │ (Stream Enabled: New & Old Images)
     ▼
DynamoDB Stream
     │
     ▼
AWS Lambda Function
     │
     ▼
SNS Topic
     │
     ▼
Email Notification 📩
📌 Implementation Steps
1️⃣ DynamoDB Setup
Navigate to DynamoDB Dashboard
Click Create Table
Configure:
Table Name: MyTable
Primary Key: id (String)
Click Create Table
➕ Add Sample Item
{
  "id": "1",
  "name": "Item1",
  "status": "active"
}
2️⃣ SNS Setup
Go to SNS Dashboard
Click Create Topic
Type: Standard
Name: DynamoDBAlerts
Create Subscription:
Protocol: Email
Endpoint: your-email@example.com
Confirm subscription via email
3️⃣ IAM Role for Lambda
Go to IAM → Roles → Create Role
Select Lambda
Attach Policies:
AmazonDynamoDBFullAccess
AmazonSNSFullAccess
AWSLambdaBasicExecutionRole
Name the role:
LambdaDynamoDBSNSRole
4️⃣ Lambda Function Setup
Go to Lambda → Create Function
Choose:
Runtime: Python 3.x
Assign IAM Role created earlier
🧠 Lambda Function Code
import json
import boto3

sns = boto3.client('sns')

SNS_TOPIC_ARN = 'arn:aws:sns:ap-south-1:902917582313:DynamoDBAlerts'

def lambda_handler(event, context):
    print("Received event:", json.dumps(event))

    for record in event['Records']:
        if record['eventName'] == 'MODIFY':
            
            old_image = record['dynamodb'].get('OldImage', {})
            new_image = record['dynamodb'].get('NewImage', {})

            message = "DynamoDB Item Updated!\n\n"
            message += "Old Value:\n"
            message += json.dumps(old_image, indent=2)
            message += "\n\nNew Value:\n"
            message += json.dumps(new_image, indent=2)

            response = sns.publish(
                TopicArn=SNS_TOPIC_ARN,
                Message=message,
                Subject="DynamoDB Item Update Alert"
            )

            print("SNS Notification sent! Message ID:", response['MessageId'])

    return {
        'statusCode': 200,
        'body': json.dumps('Processed DynamoDB update event')
    }
5️⃣ Enable DynamoDB Streams
Open your DynamoDB table
Go to Exports and Streams
Enable Stream:
View Type: New and old images
6️⃣ Connect Lambda to DynamoDB Stream
Open Lambda function
Click Add Trigger
Select:
Source: DynamoDB
Choose your table
Enable trigger
7️⃣ Testing
Go to DynamoDB table
Update an item:
{
  "id": "1",
  "name": "Item1",
  "status": "inactive"
}
Save changes
🚀 Expected Output
Lambda is triggered automatically
SNS sends an email notification
Email contains old and new values
🧪 Sample SNS Notification
DynamoDB Item Updated!

Old Value:
{
  "status": "inactive"
}

New Value:
{
  "status": "active"
}
📸 Screenshots
🔹 DynamoDBTable
![image_alt](https://github.com/bindu818309-bit/ServerlessArchitectureUsingAWSLambdaandBoto3/blob/ec2fc8948ca4b9768e1b5d0edc5c41ba3ab0274d/screenshots/DynamoDBTableBefore.png)

🔹 Lambda Function

(Add screenshot here)

🔹 SNS Topic & Subscription

(Add screenshot here)

🔹 Email Notification

(Add screenshot here)

🔹 CloudWatch Logs

(Add screenshot here)

❗ Common Issues & Fixes
🔸 Lambda Not Triggering
Ensure DynamoDB Streams is enabled
Check trigger is attached to Lambda
🔸 No SNS Email
Confirm email subscription
Check SNS Topic ARN in code
🔸 Permission Issues
Ensure IAM role has:
DynamoDB access
SNS publish permissions
✅ Conclusion

This project demonstrates how to build a real-time alerting system using AWS services. DynamoDB Streams, Lambda, and SNS work together to detect and notify changes automatically.

📎 Author

Bindu Reddy

