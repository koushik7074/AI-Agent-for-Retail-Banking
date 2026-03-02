# AI Agent for Retail Banking using Amazon Bedrock

An intelligent, serverless AI agent built on **Amazon Bedrock** that answers retail banking customer queries in natural language by fetching real-time account data from DynamoDB and retrieving contextual information from a RAG-powered Knowledge Base.

---

## Architecture Overview

```
Customer Query
      │
      ▼
Amazon Bedrock Agent
      │
      ├──── Action Group ──────► AWS Lambda ──► DynamoDB
      │                                        (Account Status)
      │
      └──── Knowledge Base ───► Amazon S3 Vectors (Vector Store)
                                        ▲
                                   PDF Documents
                                   (uploaded to S3)
```

---

## Tech Stack

| Service | Purpose |
|---|---|
| Amazon Bedrock | AI Agent orchestration and LLM |
| AWS Lambda | Backend logic to fetch account data |
| Amazon DynamoDB | Customer account details storage |
| Amazon S3 | OpenAPI schema + PDF document storage |
| Amazon S3 Vectors | Vector store for RAG Knowledge Base |
| IAM | Permissions and role management |
| Python + Boto3 | Lambda function implementation |

---


---

## Setup Guide

### 1. DynamoDB Table

Create a DynamoDB table to store customer account details:

- **Table Name:** `CustomerAccountStatus`
- **Primary Key:** `AccountID` (Number)

**Schema:**

| Field | Type | Description |
|---|---|---|
| AccountID | Number | Unique account identifier (Partition Key) |
| AccountName | String | Name of the customer |
| AccountStatus | String | Current status (e.g., Pending, Active) |
| Reason | String | Reason for current status |

---

### 2. Lambda Function

Deploy the Lambda function (`lambda/lambda_function.py`) which:
- Receives `AccountID` from the Bedrock Agent
- Fetches the corresponding record from DynamoDB using `boto3` `get_item`
- Returns account details back to the agent

**Required IAM permissions for Lambda execution role:**
```json
{
  "Action": [
    "dynamodb:GetItem",
    "logs:CreateLogGroup",
    "logs:CreateLogStream",
    "logs:PutLogEvents"
  ]
}
```

> **Important:** Make sure Lambda and DynamoDB are in the **same AWS region** to avoid `ResourceNotFoundException`.

---

### 3. OpenAPI Schema

The `schema/openapi_schema.json` defines the API contract for the Bedrock Agent Action Group:
- Upload this file to an **S3 bucket**
- Used by Bedrock Agent to understand when and how to invoke the Lambda function

---

### 4. Bedrock Agent Setup

1. Go to **AWS Console → Amazon Bedrock → Agents**
2. Create a new agent and select a Foundation Model (e.g., Claude)
3. Attach the auto-created **IAM service role** and give necessary permissions
4. Create an **Action Group**:
   - Select the Lambda function
   - Point to the OpenAPI schema in S3
5. **Prepare** the agent after configuration

---

### 5. Knowledge Base (RAG)

A RAG pipeline is set up to answer general banking queries from documents:

```
PDF Documents → S3 Bucket → Bedrock Knowledge Base → S3 Vectors (Vector Store)
```

1. Upload domain-specific PDF documents to an S3 bucket
2. Create a **Bedrock Knowledge Base** using **Amazon S3 Vectors** as the vector store
3. Sync the knowledge base to embed and index the documents
4. Connect the Knowledge Base to the Bedrock Agent

---

### 6. Agent Capabilities

Once fully set up, the agent can:
- **Fetch real-time account status** from DynamoDB via Lambda (e.g., *"What is the status of my account 12345?"*)
- **Answer general banking queries** from the Knowledge Base (e.g., *"Why is my account pending?"*)

---

## Sample Query & Response

**Customer:** *"Can you tell me the status of account 10023?"*

**Agent:** *"Account 10023 belongs to John Doe. The current status is Pending due to InvalidIdentification — the submitted ID document could not be verified. Please resubmit a valid government-issued ID to proceed."*

---



## Author

**Koushik Biswas**
[LinkedIn](https://linkedin.com/in/koushik-biswas-juiitm) • [GitHub](https://github.com/koushik7074)
