# API Gateway to S3 with Lambda Processing (AWS SAM)

This project demonstrates an API Gateway that writes data to S3, with Lambda processing triggered by S3 events. It has been translated from a Serverless Framework project to AWS SAM.

## Architecture

1. API Gateway accepts PUT requests with JSON payloads
2. API Gateway writes the data directly to an S3 bucket
3. S3 bucket triggers a Lambda function when objects are created
4. Lambda processes the S3 data

## Prerequisites

- [AWS CLI](https://aws.amazon.com/cli/) configured with appropriate permissions
- [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)
- [Docker](https://www.docker.com/products/docker-desktop) (for local testing)
- Python 3.11

## Deployment

### Build the application

```bash
sam build
```

### Deploy the application

```bash
sam deploy --guided
```

Follow the prompts in the deploy process to set up your deployment configuration.

## Local Testing

### Invoke Lambda function locally

Create a test event file `event.json`:

```json
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
          "name": "example-bucket-name",
          "arn": "arn:aws:s3:::example-bucket-name"
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
```

Then invoke the function:

```bash
sam local invoke ProcessS3DataFunction --event event.json
```

### Start API Gateway locally

```bash
sam local start-api
```

## API Usage

### PUT Request Example

```bash
curl -X PUT \
  https://{api-id}.execute-api.us-east-1.amazonaws.com/dev/api/orders/order123.json \
  -H 'Content-Type: application/json' \
  -H 'x-api-key: {your-api-key}' \
  -d '{
    "orderDate": "2023-01-01",
    "orderPaymentAmount": 100.50,
    "orderPaymentCurrency": "USD",
    "customerId": 12345
  }'
```

## Cleanup

To delete the application:

```bash
sam delete
```

## Additional Resources

- [AWS SAM Developer Guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)
- [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
- [API Gateway Developer Guide](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html)
- [Amazon S3 Developer Guide](https://docs.aws.amazon.com/AmazonS3/latest/dev/Welcome.html)
