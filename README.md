# Automated-AWS-Receipt-Processing-System
An event-driven serverless application that automatically extracts data from uploaded receipt images using OCR, stores the results in a NoSQL database, and sends an email notification to the user.

## 📌 Overview
This project demonstrates a robust cloud-native architecture using AWS Lambda, Amazon Textract, and Amazon S3. It is designed to handle 

document processing workflows without the need for managing servers, offering high scalability and cost-efficiency.

## 🏗️ Architecture

1. Upload: User uploads a receipt image (JPG, PNG, or PDF) to an Amazon S3 bucket.
   
2. Trigger: The upload event triggers an AWS Lambda function.

3. Extraction: Lambda calls Amazon Textract to perform Intelligent Character Recognition (ICR) and extract structured data (vendor, date, total).

4. Storage: The extracted data is stored in Amazon DynamoDB.

5. Notification: Amazon SES sends an automated email confirmation to the user.

## 🛠️ Tech Stack

1. Cloud Provider: Amazon Web Services (AWS)
2. Compute: AWS Lambda (Python 3.x)
3. Storage: Amazon S3 (Object) & Amazon DynamoDB (NoSQL)
4. AI/ML: Amazon Textract (OCR)
5. Communication: Amazon SES (Simple Email Service)
6. SDK: Boto3 (AWS SDK for Python)

## 🚀 Setup & Deployment

**Prerequisites**

An active AWS Account.

Verified email addresses in Amazon SES (for both sender and receiver).

1. **Storage & Database Setup**
   
Create an S3 Bucket (e.g., receipt-uploads-xyz).


<img width="1367" height="480" alt="image" src="https://github.com/user-attachments/assets/5f16fb38-6c34-4c05-84ab-0e88a5dae7a1" />

Create a DynamoDB Table named ReceiptData with a Partition Key receiptId (String).


<img width="1381" height="424" alt="image" src="https://github.com/user-attachments/assets/dc8ce56b-aea0-4886-9e5c-be73fb46ba15" />


### Lambda Configuration

Create a new Lambda function using the Python 3.12 runtime.

<img width="1384" height="272" alt="image" src="https://github.com/user-attachments/assets/c841ef54-94fa-4c19-99f3-28c576bfa841" />

**Lambda Python code**


```bash
import json, boto3, uuid, urllib.parse, os
from datetime import datetime

s3 = boto3.client('s3')
textract = boto3.client('textract')
dynamodb = boto3.resource('dynamodb').Table(os.environ['TABLE_NAME'])
ses = boto3.client('ses')

def lambda_handler(event, context):
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = urllib.parse.unquote_plus(event['Records'][0]['s3']['object']['key'])
    
    # Analyze expense with Textract
    response = textract.analyze_expense(Document={'S3Object': {'Bucket': bucket, 'Name': key}})
    
    total = "0.00"
    for doc in response['ExpenseDocuments']:
        for field in doc['SummaryFields']:
            if field['Type']['Text'] == 'TOTAL':
                total = field['ValueDetection']['Text']

    # Store in DynamoDB
    dynamodb.put_item(Item={
        'receipt_id': str(uuid.uuid4()),
        'date': datetime.now().strftime('%Y-%m-%d'),
        'file_name': key,
        'total_amount': total
    })

    # Notify via SES
    ses.send_email(
        Source=os.environ['SENDER_EMAIL'],
        Destination={'ToAddresses': [os.environ['RECIPIENT_EMAIL']]},
        Message={
            'Subject': {'Data': 'Receipt Processed Successfully'},
            'Body': {'Text': {'Data': f'Processed {key}. Total: {total}'}}
        }
    )
    return {'statusCode': 200, 'body': 'Success'}

```




<img width="1341" height="505" alt="image" src="https://github.com/user-attachments/assets/3b877453-7f4f-4306-a362-3e9e29f1e458" />


📌 IAM Role: Attach a policy granting permissions for S3:GetObject, Textract:AnalyzeDocument, DynamoDB:PutItem, and SES:SendEmail.


<img width="1379" height="586" alt="image" src="https://github.com/user-attachments/assets/7a935538-4235-4cdc-b3e3-f4d2163aaed6" />


📌 Environment Variables: Add the following keys:


<img width="1360" height="486" alt="image" src="https://github.com/user-attachments/assets/c20ebfc4-14ad-4d0b-a5bd-d8e05c99beb4" />


📌 DYNAMODB_TABLE: Name of your DynamoDB table.

<img width="1353" height="407" alt="image" src="https://github.com/user-attachments/assets/cbda1200-5654-4a8f-9ce6-0bb6a3002019" />

 SENDER_EMAIL: Your verified SES sender email.

RECEIVER_EMAIL: Your verified SES recipient email.

<img width="1371" height="480" alt="image" src="https://github.com/user-attachments/assets/607590a9-0a33-40af-a368-99635a1acb6a" />


📌 








