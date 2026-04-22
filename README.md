## AWS Serverless Order Processing System

### 📌 Overview
This repository contains the architecture and implementation of a fully serverless, event-driven order processing pipeline built natively on Amazon Web Services (AWS). The system demonstrates a decoupled, fault-tolerant backend where cloud resources scale automatically to handle e-commerce traffic without the need to manage any underlying servers.


### 🏗️ System Architecture & Workflow
The system follows a strict decoupling pattern, separating frontend data ingestion from backend processing logic.

#### The Client Request (Trigger): 
A client (web or mobile app) submits an HTTP POST request containing a JSON payload with order details (OrderID, customerName, items, shippingAddress).

#### API Gateway (Entry Point): 
The API Gateway acts as the secure front door, receiving the traffic and routing the payload directly to the backend compute layer.

#### AWS Lambda (Core Logic):
The Lambda function acts as the dispatcher. It extracts the JSON payload and uses the AWS boto3 SDK to simultaneously execute three separate backend tasks.


### Parallel Output Execution:

#### Amazon SQS (Queuing): 
The order data is sent to a FIFO (First-In-First-Out) queue. This provides fault tolerance by safely holding the order in the exact sequence it was placed until downstream inventory systems are ready to process it.

#### Amazon SNS (Alerting):
A message is published to an SNS standard topic, immediately pushing a real-time "Order Received" email notification to the system administrator.

#### Amazon DynamoDB (Storage): 
The order details are permanently written into a highly scalable NoSQL database for future querying and record-keeping.


### 🛠️ AWS Services Utilized

#### Amazon API Gateway: 
Acts as the entry point for the system, safely receiving incoming HTTP POST requests and routing data to the Lambda function.

#### AWS Lambda: 
The serverless compute engine. It runs the Python logic only when triggered, automatically scaling to meet demand and coordinating data delivery to SQS, SNS, and DynamoDB.

#### Amazon SQS (Simple Queue Service):
A managed message queue used here as a FIFO queue. It ensures strict message ordering and provides a fault-tolerant buffer between frontend collection and backend processing.

#### Amazon SNS (Simple Notification Service):
A Pub/Sub messaging service used to decouple microservices and deliver real-time email alerts to administrators.

#### Amazon DynamoDB: 
A fast, serverless NoSQL database used as the permanent storage system for persisting the structured order payloads.

#### AWS IAM (Identity and Access Management): 
The security layer controlling execution roles, explicitly granting the Lambda function permission to PutItem in DynamoDB, Publish to SNS, and SendMessage to SQS.


### ⚙️ Testing the Pipeline
The system was verified using AWS CloudShell to simulate client traffic. A curl command was used to send a POST request to the API Gateway Invoke URL:

Bash
```
curl -X POST https://[your-api-id].execute-api.[region].amazonaws.com/default/LambdaSQS \
-H 'Content-Type: application/json' \
-d '{"OrderID": "23214", "customerName": "Harry Kane", "items": [{"itemId": "item456", "quantity": 22}, {"itemId": "item789", "quantity": 18}], "shippingAddress": "11 Nguyen Van Linh, Hanoi, Vietnam"}'
```

A successful test returns a 200 Status Code, triggers an email alert, populates the DynamoDB table, and drops the payload into the SQS FIFO queue.


#### 📂 Handling Large Files with Git LFS
To accommodate large architectural documentation (like project PDFs) within this repository, Git Large File Storage (LFS) was implemented. The repository utilizes a .gitattributes file containing the following instruction:

```
*.pdf filter=lfs diff=lfs merge=lfs -text
```

This line acts as the instruction manual for Git LFS, explicitly dictating how PDF files are handled:

#### *.pdf (The Target): 
Applies the rules to every file ending in .pdf.

#### filter=lfs (The Process): 
When staging the PDF, Git does not copy the file into its internal database. Instead, the heavy PDF is moved to the local LFS cache, and a tiny pointer file is committed in its place, allowing GitHub's large-file servers to handle the heavy lifting.

#### diff=lfs (The Comparison):
Because a PDF is a complex binary file, this prevents Git from attempting to read inside the file line-by-line, and instead checks if the version ID has changed.

merge=lfs (Conflict Resolution): Instructs Git to use LFS logic for merge conflicts, picking one version over another based on the file's unique ID rather than attempting a text merge.

-text (The Data Type): Explicitly declares the file is not plain text, preventing Git from attempting to format line endings (like Windows CRLF to Linux LF), which would corrupt the binary file.
