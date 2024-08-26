
# AWS Lambda S3 to SES Notification

## Overview

This project is an AWS Lambda function designed to send an email notification via Amazon Simple Email Service (SES) whenever a new object is uploaded to a specified S3 bucket. The notification is sent to a Gmail account.

## Prerequisites

Before setting up this project, ensure you have the following:

- **AWS Account**: You need an AWS account to create and configure Lambda functions, S3 buckets, and SES.
- **AWS CLI**: Installed and configured with appropriate IAM permissions.
- **SES Verified Email Address**: Ensure your Gmail address is verified in SES.

## Architecture

1. **S3 Bucket**: Stores the uploaded files.
2. **Lambda Function**: Triggered when an object is uploaded to the S3 bucket.
3. **SES**: Used to send an email notification to a Gmail address.

## Setup Instructions

### Step 1: Create the S3 Bucket

1. Go to the S3 service in your AWS Console.
2. Click **Create Bucket**.
3. Name your bucket (must be unique across all AWS accounts).
4. Configure bucket settings as needed, then click **Create Bucket**.

### Step 2: Verify Email in SES

1. Go to the SES service in your AWS Console.
2. In the **Email Addresses** section, click **Verify a New Email Address**.
3. Enter your Gmail address and verify it via the confirmation email you receive.

### Step 3: Create the Lambda Function

1. Go to the Lambda service in your AWS Console.
2. Click **Create Function**.
3. Choose **Author from scratch**:
   - Name: `S3ToSESNotification`
   - Runtime: Choose the appropriate runtime (e.g., Python, Node.js)
4. Create a new role with the required permissions:
   - **S3 Read**: To access the S3 bucket.
   - **SES Send**: To send emails via SES.
5. Click **Create Function**.

### Step 4: Write the Lambda Function Code

Replace the placeholder code in the Lambda editor with the following code (adapt it based on your runtime):

```python
import json
import boto3

def lambda_handler(event, context):
    s3_client = boto3.client('s3')
    ses_client = boto3.client('ses')
    
    bucket_name = event['Records'][0]['s3']['bucket']['name']
    object_key = event['Records'][0]['s3']['object']['key']
    
    email_from = "your-verified-email@gmail.com"
    email_to = "recipient-email@gmail.com"
    email_subject = f"New Object Uploaded: {object_key}"
    email_body = f"A new object has been uploaded to {bucket_name}:

Key: {object_key}"

    ses_client.send_email(
        Source=email_from,
        Destination={'ToAddresses': [email_to]},
        Message={
            'Subject': {'Data': email_subject},
            'Body': {'Text': {'Data': email_body}}
        }
    )
    
    return {
        'statusCode': 200,
        'body': json.dumps('Email sent successfully!')
    }
```

### Step 5: Set Up S3 Bucket Trigger

1. In your Lambda function, go to the **Configuration** tab and click on **Triggers**.
2. Add a new trigger, select **S3**, and choose your bucket.
3. Under **Event type**, choose **All object create events**.
4. Click **Add**.

### Step 6: Deploy the Lambda Function

1. Review your function and click **Deploy**.
2. Test the function by uploading an object to your S3 bucket and verifying the email notification.

## Testing

- Upload a file to the S3 bucket and check your Gmail for the notification.
- Ensure the email contains the correct details (bucket name, object key).

## Troubleshooting

- **Permission Errors**: Ensure your Lambda execution role has the correct permissions.
- **SES Quota Limits**: Check if your SES is in the sandbox environment; you may need to request production access.
- **Email Not Received**: Verify the SES email address and check your spam folder.

## Conclusion

This project demonstrates how to integrate AWS Lambda with S3 and SES to send email notifications on S3 object uploads. It can be expanded to include more complex use cases such as filtering specific objects or sending notifications to multiple recipients.
