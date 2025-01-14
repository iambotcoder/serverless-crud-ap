# Lambda API Gateway Integration with DynamoDB
---

## Introduction

This project demonstrates the implementation of a serverless API using **AWS services** such as **API Gateway**, **Lambda**, and **DynamoDB**. It showcases how to build a backend system without managing servers, utilizing these components to handle HTTP requests, execute backend logic, and interact with a NoSQL database. 

The key components of this project are:
- **API Gateway**: Acts as an HTTP interface for accessing Lambda functions.
- **AWS Lambda**: Runs backend logic for processing API requests.
- **DynamoDB**: Serves as the NoSQL database for storing and retrieving data.

---

## Table of Contents
- [Overview](#overview)
- [Prerequisites](#Prerequisites)
- [Architecture Diagram](#architecture-diagram)
- [Implementation Steps](#implementation-steps)
  - [Step 1: Create IAM Policy and Role](#step-1-create-iam-policy-and-role)
  - [Step 2: Deploy the Lambda Function](#step-2-deploy-the-lambda-function)
  - [Step 3: Configure API Gateway](#step-3-configure-api-gateway)
  - [Step 4: Test the Solution with Postman](#step-4-test-the-solution-with-postman)
- [Screenshots](#screenshots)
- [Conclusion](#conclusion)

---


## Prerequisites
Before you start, ensure you have the following:
- An AWS account
- Basic knowledge of AWS services: Lambda, API Gateway, and DynamoDB
- Installed tools: [Postman](https://www.postman.com/)

---

## Architecture Diagram
Provide a high-level architecture diagram of the project setup.
![Architecture Diagram](https://github.com/user-attachments/assets/f4dffa5b-22d1-4ec9-a004-5958b36215ab)

---
### Supported Operations
The API supports the following operations:
- **DynamoDB CRUD Operations**:
  - `create`, `read`, `update`, `delete`, `list`
- **Testing Operations**:
  - `echo` (returns the input as-is)
  - `ping` (returns "pong")

### Steps to Implement

### 1. Setup: IAM Role Creation
- **Create an IAM Role (lambda-apigateway-role):**
  1. Go to the AWS Management Console and navigate to the **IAM** service.
  2. Create a new policy with the following JSON
  3. Attach the following custom policy JSON to provide necessary permissions:
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Effect": "Allow",
           "Action": [
             "dynamodb:PutItem",
             "dynamodb:GetItem",
             "dynamodb:UpdateItem",
             "dynamodb:DeleteItem",
             "dynamodb:Scan"
           ],
           "Resource": "arn:aws:dynamodb:<region>:<account-id>:table/lambda-apigateway"
         }
       ]
     }
     ```
  4. Save the policy with the name **lambda-apigateway-policy**.
  5. Go to the Roles section in IAM and create a new role.
  6. Attach the **lambda-apigateway-policy** to the role.
  

### 2. Create Lambda Function
- **Lambda Function (LambdaFunctionOverHttps):**
  1. Go to the **AWS Lambda** console and click on **Create Function**.
  2. Choose the following settings:
     - **Author from scratch**: Yes.
     - **Runtime**: Python 3.13 or a newer supported version as the runtime.
     - **Execution Role**: Use the previously created `lambda-apigateway-role`.
  3. Use the following Python code snippet for the function:
     ```python
     from __future__ import print_function

      import boto3
      import json
      
      print('Loading function')
      
      def lambda_handler(event, context):
          operation = event['operation']

      if 'tableName' in event:
          dynamo = boto3.resource('dynamodb').Table(event['tableName'])
  
      operations = {
          'create': lambda x: dynamo.put_item(**x),
          'read': lambda x: dynamo.get_item(**x),
          'update': lambda x: dynamo.update_item(**x),
          'delete': lambda x: dynamo.delete_item(**x),
          'list': lambda x: dynamo.scan(**x),
          'echo': lambda x: x,
          'ping': lambda x: 'pong'
      }
  
      if operation in operations:
          return operations[operation](event.get('payload'))
      else:
          raise ValueError('Unrecognized operation "{}"'.format(operation))

     ```
  4. Save and deploy the function.

### 3. Test Lambda Function
- Test the Lambda function using the **echo** operation:
  - Sample test event JSON:
    ```json
    {
      "operation": "echo",
      "payload": {
          "somekey1": "somevalue1",
          "somekey2": "somevalue2"
      }
    }
    ```
  - Expected output:
    ```json
    {
      "somekey1": "somevalue1",
      "somekey2": "somevalue2"
    }
    ```

### 4. Create DynamoDB Table
- **DynamoDB Table (lambda-apigateway):**
  1. Go to the **DynamoDB** console and click on **Create Table**.
  2. Set the table name to `lambda-apigateway`.
  3. Define the **Primary Key**:
     - Partition Key: `id` (String).

### 5. Create API Gateway
- **API Gateway Setup:**
  1. Navigate to **API Gateway** and create a new API:
     - Name: `DynamoDBOperations`.
  2. Add a new resource:
     - Resource Name: `DynamoDBManager`.
  3. Add a method to the resource:
     - Method: `POST`.
  4. Integrate the POST method with the Lambda function:
     - Select **Lambda Function** as the integration type.
     - Specify the function name (`LambdaFunctionOverHttps`).

### 6. Deploy API
- **Deploy the API:**
  1. Go to the **Actions** menu and select **Deploy API**.
  2. Create a new stage: `prod`.
  3. Retrieve the **Invoke URL** for the deployed API.

## Running  the Solution with Postman.
  1. Open Postman and create a new request with the following configurations:
       - HTTP Method: POST
       - URL: `<API Gateway Invoke URL>/DynamoDBManager`
  2. Test different scenarios using the following payloads:
  
  - Create Operation
    ```json
    {
        "operation": "create",
        "tableName": "lambda-apigateway",
        "payload": {
            "Item": {
                "id": "123",
                "value": "sample data"
            }
        }
    }
    ```

  - Outcome
    ![Screenshot 2025-01-14 050636](https://github.com/user-attachments/assets/f86899d4-ac95-4d20-b0b6-8cddd11d45b0)


  - Read Operation
    ```json
    {
      "operation": "read",
      "tableName": "lambda-apigateway",
      "payload": {
          "Key": {
              "id": "123"
          }
      }
    }
    ```
    ![Screenshot 2025-01-14 050652](https://github.com/user-attachments/assets/fec0ba19-ff73-419f-8312-4200847d40ab)

  
  - Update Operation
  ```json
      {
        "operation": "update",
        "tableName": "lambda-apigateway",
        "payload": {
            "Key": {
                "id": "56789ABCD"
            },
            "UpdateExpression": "set #num = :val1",
            "ExpressionAttributeValues": {
                ":val1": 42
            },
            "ExpressionAttributeNames": {
                "#num": "number"
            },
            "ReturnValues": "UPDATED_NEW"
        }
    }

  ```
  - outcome
  ![Screenshot 2025-01-14 053219](https://github.com/user-attachments/assets/4f00bee4-95d8-41b2-9833-851ce6114803)

  - Delete Operation
  ```json
    {
      "operation": "delete",
      "tableName": "lambda-apigateway",
      "payload": {
          "Key": {
              "id": "56789ABCD"
          }
      }
    }
  ```
  - outcome
  ![Screenshot 2025-01-14 052910](https://github.com/user-attachments/assets/281cf0de-a4e3-4a4b-879b-a2420e7e6cf6)

- Verify the results in the DynamoDB console.

---
### Cleanup
- Delete the created resources to avoid unnecessary costs:
  1. DynamoDB table.
  2. Lambda function.
  3. API Gateway resources.

---
### Conclusion
This guide demonstrates how to integrate AWS Lambda with API Gateway and DynamoDB to perform database operations in a serverless architecture. It provides a scalable and efficient approach to manage CRUD functionalities with minimal infrastructure management.
