# Development Summary for API Gateway to S3 with Lambda Processing

This document provides a summary of the AWS SAM translation of the Serverless Framework project, including development and testing instructions.

## Project Overview

This project implements:
1. An API Gateway that accepts PUT requests with JSON payloads
2. Direct writing of data to an S3 bucket via API Gateway
3. A Lambda function triggered by S3 object creation events
4. Processing of S3 data by the Lambda function

## Prerequisites

- AWS CLI configured with appropriate permissions
- AWS SAM CLI installed
- Docker installed (for local testing)
- Python 3.11

## Local Development and Testing

### Lambda Function Testing

The project includes one Lambda function:

1. **ProcessS3DataFunction**
   - Triggered by S3 object creation events
   - Processes data from S3 objects
   - Located at `src/process_s3_data.py`

To test this function locally:

```bash
# Create a test event file
cat > event.json << EOF
{
  "Records": [
    {
      "eventVersion": "2.1",
      "eventSource": "aws:s3",
      "awsRegion": "us-east-1",
      "eventTime": "2023-01-01T12:00:00.000Z",
      "eventName": "ObjectCreated:Put",
      "s3": {
        "s3SchemaVersion": "1.0",
        "bucket": {
          "name": "bb245-sfp",
          "arn": "arn:aws:s3:::bb245-sfp"
        },
        "object": {
          "key": "test-order.json",
          "size": 123,
          "eTag": "abcdef123456"
        }
      }
    }
  ]
}
EOF

# Invoke the function locally
sam local invoke ProcessS3DataFunction --event event.json
```

### API Gateway Testing

The API Gateway exposes a PUT endpoint that writes data directly to S3:

```bash
# Start the API locally
sam local start-api

# Test the API with curl (in a new terminal)
curl -X PUT \
  http://localhost:3000/api/orders/test-order.json \
  -H 'Content-Type: application/json' \
  -d '{
    "orderDate": "2023-01-01",
    "orderPaymentAmount": 100.50,
    "orderPaymentCurrency": "USD",
    "customerId": 12345
  }'
```

## Build and Deployment

### Build the Application

```bash
sam build
```

### Deploy the Application

```bash
# First-time deployment with guided setup
sam deploy --guided

# Subsequent deployments
sam deploy
```

### Cleanup

To remove all resources created by this project:

```bash
sam delete
```

## Resource Details

### Lambda Function
- **Name**: ProcessS3DataFunction
- **Handler**: process_s3_data.lambda_handler
- **Runtime**: Python 3.11
- **Event Source**: S3 bucket object creation

### API Gateway
- **Endpoint**: PUT /api/{order_object_path+}
- **Authentication**: API Key
- **Integration**: Direct S3 integration

### S3 Bucket
- **Name**: bb245-sfp
- **Versioning**: Suspended
- **Access Control**: Private

## Notes for Developers

1. The API Gateway uses a request validator to ensure proper JSON format
2. The Lambda function uses AWS Lambda Powertools for Python for logging and event parsing
3. API Gateway logs are sent to CloudWatch
4. The S3 bucket has retention policies to prevent accidental deletion
