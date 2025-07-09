# Development Summary for API Gateway to S3 with Lambda Processing

## Prerequisites

- **AWS CLI**: Version 2.0 or later, configured with appropriate credentials
- **AWS SAM CLI**: Version 1.78.0 or later
- **Python**: Version 3.11
- **Docker**: Required for local testing
- **LocalStack**: Version 4.0.0 or later, for local AWS service emulation

## Build & Deploy

### Initial Setup

1. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

2. **Create requirements.txt file**:
   ```bash
   echo "boto3==1.28.38" > requirements.txt
   echo "aws-lambda-powertools==2.22.0" >> requirements.txt
   echo "urllib3==2.2.3" >> requirements.txt
   ```

### Build Process

1. **Build the SAM application**:
   ```bash
   sam build
   ```
   This command will:
   - Install dependencies from requirements.txt
   - Package your Lambda function code
   - Prepare the application for deployment

### Deployment

1. **First-time deployment (guided)**:
   ```bash
   sam deploy --guided
   ```
   Follow the prompts to configure:
   - Stack name
   - AWS Region
   - Parameter values (Stage, BucketName)
   - Confirm IAM role creation
   - Save configuration to samconfig.toml

2. **Subsequent deployments**:
   ```bash
   sam deploy
   ```
   This will use the saved configuration from samconfig.toml.

3. **Deployment with specific parameters**:
   ```bash
   sam deploy --parameter-overrides Stage=prod BucketName=my-prod-bucket
   ```

## Local API Testing with LocalStack

1. **Start the API locally with LocalStack environment**:
   ```bash
   export AWS_ENDPOINT_URL=http://localhost:4566
   sam local start-api --env-vars env.json
   ```

2. **Test the API with curl**:
   ```bash
   curl -X PUT \
     http://localhost:3000/api/test-order \
     -H 'Content-Type: application/json' \
     -H 'x-api-key: test-api-key' \
     -d '{
       "orderId": 12345,
       "orderDate": "2023-04-28",
       "orderPaymentAmount": 100.50,
       "orderPaymentCurrency": "USD",
       "customerId": 67890
     }'
   ```

3. **Verify the file was created in LocalStack S3**:
   ```bash
   aws --endpoint-url=http://localhost:4566 s3 ls s3://bb245-sfp/
   ```

## Local Testing with LocalStack

### Setting up LocalStack

1. **Start LocalStack**:
   ```bash
   localstack start -d
   ```

2. **Create S3 bucket in LocalStack**:
   ```bash
   aws --endpoint-url=http://localhost:4566 s3 mb s3://bb245-sfp
   ```

3. **Upload a test file to the bucket**:
   ```bash
   echo '{"orderId": 12345, "orderDate": "2023-04-28", "orderPaymentAmount": 100.50, "orderPaymentCurrency": "USD", "customerId": 67890}' > test-order.json
   aws --endpoint-url=http://localhost:4566 s3 cp test-order.json s3://bb245-sfp/test-order
   ```

4. **Verify the file was uploaded**:
   ```bash
   aws --endpoint-url=http://localhost:4566 s3 ls s3://bb245-sfp/
   ```

### Local Lambda Testing with LocalStack

1. **Generate a sample S3 event**:
   ```bash
   sam local generate-event s3 put --bucket bb245-sfp --key test-order > events/s3-event.json
   ```

2. **Invoke the Lambda function locally with LocalStack**:
   ```bash
   export AWS_ENDPOINT_URL=http://localhost:4566
   sam local invoke ProcessS3DataFunction --event events/s3-event.json --env-vars env.json
   ```

   Create an env.json file with:
   ```json
   {
     "ProcessS3DataFunction": {
       "AWS_ENDPOINT_URL": "http://localhost:4566",
       "POWERTOOLS_SERVICE_NAME": "processS3Order",
       "POWERTOOLS_LOG_LEVEL": "INFO"
     }
   }
   ```

## Monitoring & Debugging

1. **View Lambda logs**:
   ```bash
   sam logs -n ProcessS3DataFunction --stack-name <stack-name>
   ```

2. **Tail Lambda logs**:
   ```bash
   sam logs -n ProcessS3DataFunction --stack-name <stack-name> --tail
   ```

3. **Access CloudWatch Logs** via AWS Console:
   - Navigate to CloudWatch > Log Groups > /aws/lambda/ProcessS3DataFunction

## Cleanup

1. **Delete the SAM application**:
   ```bash
   sam delete
   ```
   This will remove all resources created by the SAM template.

2. **Manual cleanup** (if needed):
   - Empty the S3 bucket before deletion (required due to DeletionPolicy: Retain)
   ```bash
   aws s3 rm s3://<bucket-name> --recursive
   ```

## CI/CD Integration

### GitHub Actions Example

```yaml
name: Deploy SAM Application

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - uses: aws-actions/setup-sam@v2
      - uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      - run: sam build
      - run: sam deploy --no-confirm-changeset --no-fail-on-empty-changeset
```

### AWS CodePipeline Integration

1. Create a buildspec.yml file in your repository:
   ```yaml
   version: 0.2
   phases:
     install:
       runtime-versions:
         python: 3.11
     build:
       commands:
         - sam build
         - sam deploy --no-confirm-changeset --no-fail-on-empty-changeset
   artifacts:
     files:
       - template.yaml
       - packaged.yaml
       - samconfig.toml
   ```

2. Configure AWS CodePipeline to use AWS CodeBuild with this buildspec.yml file.
