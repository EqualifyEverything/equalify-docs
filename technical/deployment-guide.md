# Deployment Guide

This guide covers deploying Equalify to AWS infrastructure, including the frontend, backend API, and scanning services.

## Prerequisites

- AWS CLI v2 with SSO configured
- Node.js 18+ and Yarn
- AWS account with appropriate permissions
- Java 11+ (for PDF scanner)

## AWS SSO Setup

Configure AWS CLI for SSO access:

```bash
aws configure sso --profile equalifyuic
```

You'll be prompted for:
- **SSO session name**: `equalifyuic-sso`
- **SSO start URL**: `https://equalifyuic.awsapps.com/start`
- **SSO region**: `us-east-2`
- **CLI default region**: `us-east-2`
- **CLI output format**: `json`

To login for subsequent sessions:
```bash
aws sso login --profile equalifyuic
```

## Project Setup

Install all dependencies from the repository root:

```bash
cd equalify
yarn install
```

## Deploying the Frontend

The frontend deploys to S3 with CloudFront CDN.

### Build and Deploy to Staging
```bash
cd apps/frontend
yarn build:staging
```

This command:
1. Builds with Vite using staging environment variables
2. Syncs to S3 bucket `equalifyuic-web-staging`
3. Invalidates CloudFront distribution

### Build and Deploy to Production
```bash
yarn build:prod
```

This command:
1. Builds with Vite using production environment variables
2. Syncs to S3 bucket `equalifyuic-web`
3. Invalidates CloudFront distribution

### Deploy Both Environments
```bash
yarn build
```

## Deploying the Backend API

The backend deploys as a single Lambda function.

### Build and Deploy to Staging
```bash
cd apps/backend
yarn build:staging
```

This command:
1. Bundles with esbuild (excluding AWS SDK)
2. Creates `lambda.zip`
3. Updates Lambda function `equalifyuic-api-staging`

### Build and Deploy to Production
```bash
yarn build:prod
```

Updates Lambda function `equalifyuic-api`.

### Deploy Both Environments
```bash
yarn build
```

## Deploying Scanning Services

Each scanning service deploys independently.

### SQS Router
```bash
cd services/aws-lambda-scan-sqs-router
yarn build
```

### HTML Scanner
```bash
cd services/aws-lambda-scan-html
yarn build
```

### PDF Scanner (TypeScript)
```bash
cd services/aws-lambda-scan-pdf
yarn build
```

### PDF Scanner (Java/veraPDF)
```bash
cd services/aws-lambda-verapdf-interface
mvn package
# Deploy the resulting JAR to Lambda
```

## Environment Configuration

### Frontend Environment Variables

Create `.env.production` and `.env.staging` files:

```env
VITE_USERPOOLID=us-east-2_XXXXXXXX
VITE_USERPOOLWEBCLIENTID=XXXXXXXXXXXXXXXXXXXXXXXXXX
VITE_API_URL=https://api.equalifyapp.com
VITE_GRAPHQL_URL=https://graphql.equalifyapp.com
VITE_SSO_CLIENT_ID=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
VITE_SSO_AUTHORITY=https://login.microsoftonline.com/TENANT_ID
```

### Backend Environment Variables

Configure in AWS Lambda:

| Variable | Description |
|----------|-------------|
| `DB_USER` | PostgreSQL username |
| `DB_HOST` | PostgreSQL host (RDS endpoint) |
| `DB_NAME` | Database name |
| `DB_PASSWORD` | Database password |
| `USER_POOL_ID` | Cognito User Pool ID |
| `WEB_CLIENT_ID` | Cognito Web Client ID |
| `SSO_ENABLED` | "true" to enable SSO |
| `WEBHOOKSECRET` | Hasura webhook secret |

### Scanner Environment Variables

| Variable | Description |
|----------|-------------|
| `RESULTS_ENDPOINT` | Webhook URL for results |

## AWS Infrastructure

### Required Resources

- **Lambda Functions**: API, SQS Router, HTML Scanner, PDF Scanner, veraPDF
- **SQS Queues**: `scanHtml.fifo`, `scanPdf.fifo`
- **S3 Buckets**: Frontend hosting (staging + production)
- **CloudFront**: CDN distributions
- **RDS**: PostgreSQL instance
- **Cognito**: User Pool for authentication
- **Hasura**: GraphQL engine (can be self-hosted or Hasura Cloud)

### Lambda Layers

The HTML scanner requires a Chromium layer. Use a pre-built layer:
- `@sparticuz/chromium` compatible layer

### SQS Configuration

FIFO queues with:
- Content-based deduplication: Disabled (using MessageDeduplicationId)
- Message group ID: `auditId` for ordered processing
- Visibility timeout: 5 minutes (for HTML scanner)

## Monitoring

### CloudWatch Logs

All Lambda functions log to CloudWatch:
- `/aws/lambda/equalifyuic-api`
- `/aws/lambda/equalifyuic-api-staging`
- `/aws/lambda/aws-lambda-scan-sqs-router`
- `/aws/lambda/aws-lambda-scan-html`
- `/aws/lambda/aws-lambda-scan-pdf`

### Lambda Powertools Metrics

Scanning services emit metrics:
- `scansStarted` - Count
- `ScanDuration` - Milliseconds

### Health Checks

Monitor:
- Lambda error rates
- SQS queue depth
- Database connections
- CloudFront error rates

## Troubleshooting

### Common Issues

**SSO Token Expired**
```bash
aws sso login --profile equalifyuic
```

**Lambda Deployment Fails**
- Ensure AWS CLI is authenticated
- Check Lambda function exists
- Verify IAM permissions

**Frontend Build Errors**
- Clear `node_modules` and reinstall
- Check environment variables are set

**Scan Timeouts**
- Increase Lambda timeout
- Check if pages are very large/slow
- Review CloudWatch logs for errors

### Logs

View recent logs:
```bash
aws logs tail /aws/lambda/equalifyuic-api --follow --profile equalifyuic
```

---
*For architecture details, see the Architecture Overview.*
