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


<img width="880" height="633" alt="image" src="https://github.com/user-attachments/assets/c3facd79-fff2-44ff-8a4f-d4a809bd63ad" />



---




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

---

Create a DynamoDB Table named ReceiptData with a Partition Key receiptId (String).


<img width="1381" height="424" alt="image" src="https://github.com/user-attachments/assets/dc8ce56b-aea0-4886-9e5c-be73fb46ba15" />

---


### Lambda Configuration

Create a new Lambda function using the Python 3.12 runtime.

<img width="1384" height="272" alt="image" src="https://github.com/user-attachments/assets/c841ef54-94fa-4c19-99f3-28c576bfa841" />

---

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

---

          
<img width="1341" height="505" alt="image" src="https://github.com/user-attachments/assets/3b877453-7f4f-4306-a362-3e9e29f1e458" />

---


📌 IAM Role: Attach a policy granting permissions for S3:GetObject, Textract:AnalyzeDocument, DynamoDB:PutItem, and SES:SendEmail.


<img width="1379" height="586" alt="image" src="https://github.com/user-attachments/assets/7a935538-4235-4cdc-b3e3-f4d2163aaed6" />

---


📌 Environment Variables: Add the following keys:


<img width="1360" height="486" alt="image" src="https://github.com/user-attachments/assets/c20ebfc4-14ad-4d0b-a5bd-d8e05c99beb4" />

---

📌 DYNAMODB_TABLE: Name of your DynamoDB table.

<img width="1353" height="407" alt="image" src="https://github.com/user-attachments/assets/cbda1200-5654-4a8f-9ce6-0bb6a3002019" />

---

 SENDER_EMAIL: Your verified SES sender email.

RECEIVER_EMAIL: Your verified SES recipient email.

<img width="1371" height="480" alt="image" src="https://github.com/user-attachments/assets/607590a9-0a33-40af-a368-99635a1acb6a" />

---

## 📌 Event Trigger

1. In the S3 Console, navigate to Properties > Event Notifications.
2. Create a notification for All object create events.
3. Set the Destination to your Lambda function.

<img width="1364" height="418" alt="image" src="https://github.com/user-attachments/assets/291b71b3-b43e-4228-8e8e-210b6e69c1f8" />

---

<img width="1336" height="525" alt="image" src="https://github.com/user-attachments/assets/72aaef98-f7a7-4bc6-9018-7cc96486a2a8" />

---

<img width="1347" height="531" alt="image" src="https://github.com/user-attachments/assets/31e8012e-b4ad-43d4-8c10-68943157a409" />

---


## 📌 Event Trigger Confirmation in DynamoDB

1. In the S3 Console, navigate to DynamoDB > Tables.
2. Click on Receipts. > Explore table items

<img width="1374" height="581" alt="image" src="https://github.com/user-attachments/assets/96e41b22-60c8-4ec6-b209-a58544333165" />

---

### Problem Statement

Traditional manual expense reporting is time-consuming, prone to human error, and difficult to scale. Businesses require an automated, cost-effective solution to ingest physical receipts, extract key financial data with high accuracy, and provide instant confirmation to users. This project addresses the need for a Serverless Intelligent Document Processing (IDP) pipeline that eliminates manual data entry and reduces the operational overhead of managing physical records.

### Architectural Rationale: Why These Services?

When designing this system, I prioritized **cost-efficiency**, **scalability**, and **low operational overhead**. Below is the justification for the chosen stack:


| Chosen Service | Alternative Considered | Professional Reason for Choice |
| :--- | :--- | :--- |
| **AWS Lambda** | Amazon EC2 | **Cost & Scaling:** EC2 requires paying for idle time and managing OS patches. Lambda is "Scale-to-Zero," meaning you only pay when a receipt is actually being processed. |
| **Amazon Textract** | Tesseract (Self-hosted) | **Accuracy:** Traditional OCR struggles with skewed or low-light receipt photos. Textract uses pre-trained ML models specifically optimized for forms and tables without requiring custom model training. |
| **Amazon DynamoDB** | Amazon RDS (SQL) | **Flexibility:** Receipt data formats can change. DynamoDB’s schema-less nature allows for storing varying metadata without the rigidity of a relational database. |
| **Amazon S3** | EBS/EFS Storage | **Durability:** S3 is designed for 99.999999999% durability and provides native "Event Notifications," which acts as the automatic "starter motor" for our entire pipeline. |

---


### Cost Optimization Statement

"This architecture follows a Pay-As-You-Go model to minimize Total Cost of Ownership (TCO):"
Zero Idle Costs: By using Lambda and DynamoDB (On-Demand), the system costs $0.00 when not in use.
Free Tier Utilization: For low-volume portfolios, this entire project typically fits within the AWS Free Tier (first 1 million Lambda requests and first 25GB of DynamoDB storage are free).
Textract Efficiency: Instead of running a heavy GPU instance 24/7 for OCR, Textract charges per page, making it significantly cheaper for intermittent workloads.

### Monitoring & Observability (CloudWatch)
To ensure the system is healthy, we implement Amazon CloudWatch:
Logs: Every Lambda execution is logged. If an extraction fails, we can inspect the syslog to see if it was a permissions error or a malformed image.
Metrics: We track Lambda Throttles and Errors. If the system scales suddenly, CloudWatch alerts us if we hit concurrency limits.
Alarms: A Billing Alarm is set to notify the administrator if the monthly spend exceeds a defined threshold (e.g., $5), preventing unexpected costs from high-volume uploads.

Traces (X-Ray): (Optional add-on) Used to visualize the latency between S3, Textract, and DynamoDB to identify bottlenecks in the processing chain.


## 🏁 Conclusion

This project successfully demonstrates the power of **Event-Driven Architecture (EDA)** in solving real-world business challenges. By leveraging AWS serverless technologies, I built a system that is not only highly scalable and durable but also extremely cost-effective, with a near-zero cost floor for low-volume usage.

### Key Takeaways:

- **Serverless First:** Learned how to architect solutions without managing underlying infrastructure, reducing operational complexity.
- **AI Integration:** Gained hands-on experience integrating Managed AI services (Amazon Textract) to turn unstructured image data into actionable structured data.
- **Cloud Security:** Applied industry-standard security practices using IAM roles and the Principle of Least Privilege.

This pipeline serves as a foundation for more complex document processing workflows, such as automated invoice approval systems or real-time financial auditing tools.
