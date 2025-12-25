# SetupLambda
A simple setup that uses AWS lambda , S3 and SQS services

A serverless AWS Lambda function built with TypeScript and deployed using AWS SAM (Serverless Application Model).

## Project Structure

```
SetupLambda/
├── src/
│   └── handler.ts          # Lambda function source code (TypeScript)
├── events/
│   └── event.json          # Sample event for local testing
├── template.yaml           # SAM/CloudFormation template defining AWS resources
├── samconfig.toml          # SAM CLI configuration file
├── package.json            # Node.js dependencies
├── tsconfig.json           # TypeScript configuration
└── README.md
```

## Prerequisites

Before you begin, ensure you have the following installed:

- **Node.js 18+** - [Install Node.js](https://nodejs.org/)
- **AWS SAM CLI** - [Install SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)
- **AWS CLI** - [Install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) (configured with credentials)
- **esbuild** - Installed globally: `npm install -g esbuild`
- **Docker** (optional) - For local testing with `sam local`

## Quick Start

### 1. Install Dependencies

```bash
npm install
```

### 2. Build the Application

```bash
sam build
```

### 3. Deploy to AWS

```bash
# First-time deployment (interactive)
sam deploy --guided

# Subsequent deployments (uses samconfig.toml)
sam deploy
```

## Configuration Files Explained

### template.yaml - SAM Template

The `template.yaml` defines your serverless infrastructure using AWS SAM syntax:

```yaml
AWSTemplateFormatVersion: '2010-09-09'   # CloudFormation template version
Transform: AWS::Serverless-2016-10-31    # Enables SAM resource types
Description: SAM Lambda function

Resources:
  SimpleFunction:
    Type: AWS::Serverless::Function      # SAM-specific Lambda resource
    Properties:
      Runtime: nodejs18.x                # Node.js runtime version
      CodeUri: src                       # Source code directory
      Handler: handler.handler           # File.exportedFunction
      Timeout: 30                        # Max execution time (seconds)
      MemorySize: 256                    # Memory allocation (MB)
    Metadata:
      BuildMethod: esbuild               # Use esbuild for bundling
      BuildProperties:
        Minify: true                     # Minify output for smaller bundle
        Target: es2022                   # JavaScript target version
        Sourcemap: true                  # Generate source maps for debugging
        EntryPoints:
          - handler.ts                   # TypeScript entry point
        External:
          - "@aws-sdk/*"                 # Exclude AWS SDK (provided by Lambda)
```

**Key Concepts:**

| Property | Description |
|----------|-------------|
| `Transform` | **Required** - Tells CloudFormation to process SAM resources |
| `Type: AWS::Serverless::Function` | SAM's simplified Lambda definition |
| `Handler` | Format: `filename.exportedFunction` |
| `BuildMethod: esbuild` | Bundles TypeScript without needing a separate build step |
| `External` | Dependencies to exclude (AWS SDK v3 is included in Lambda runtime) |

### samconfig.toml - SAM CLI Configuration

The `samconfig.toml` stores default parameters for SAM CLI commands:

```toml
version = 0.1

[default]                              # Environment name
[default.global.parameters]
stack_name = "SetupLambda"             # CloudFormation stack name
region = "ap-south-1"                  # AWS region

[default.build.parameters]
cached = true                          # Cache builds for faster rebuilds
parallel = true                        # Build functions in parallel

[default.deploy.parameters]
capabilities = "CAPABILITY_IAM"        # Allow IAM role creation
confirm_changeset = true               # Review changes before deploying
resolve_s3 = true                      # Auto-create S3 bucket for artifacts
s3_prefix = "SetupLambda"              # S3 prefix for uploaded artifacts

[default.local_start_api.parameters]
warm_containers = "EAGER"              # Pre-warm containers for faster local testing
```

**Configuration Sections:**

| Section | Purpose |
|---------|---------|
| `[default.global.parameters]` | Shared settings across all commands |
| `[default.build.parameters]` | Settings for `sam build` |
| `[default.deploy.parameters]` | Settings for `sam deploy` |
| `[default.local_*]` | Settings for local testing |

**Multiple Environments:**

```toml
[prod.deploy.parameters]
stack_name = "SetupLambda-Prod"
region = "us-east-1"
```

Use with: `sam deploy --config-env prod`

## Local Development

### Test Locally with SAM

```bash
# Invoke function with sample event
sam local invoke SimpleFunction -e events/event.json

# Start local API (if API Gateway is configured)
sam local start-api

# Start Lambda endpoint for testing
sam local start-lambda
```

### Validate Template

```bash
sam validate --lint
```

## Common SAM Commands

| Command | Description |
|---------|-------------|
| `sam build` | Build the application |
| `sam deploy` | Deploy to AWS |
| `sam deploy --guided` | Interactive deployment |
| `sam logs -n SimpleFunction --tail` | Stream function logs |
| `sam delete` | Delete the CloudFormation stack |
| `sam sync --watch` | Hot-reload during development |

## Troubleshooting

### "Unrecognized resource types: AWS::Serverless::Function"
**Cause:** Missing `Transform: AWS::Serverless-2016-10-31` in template.yaml

### "Cannot find esbuild"
**Solution:** Install esbuild globally:
```bash
npm install -g esbuild
```

### "Stack is in REVIEW_IN_PROGRESS state"
**Solution:** Delete the stuck managed stack:
```bash
aws cloudformation delete-stack --stack-name aws-sam-cli-managed-default
```

## AWS Resources Created

After deployment, the following resources are created:

- **Lambda Function:** `SetupLambda-SimpleFunction-*`
- **IAM Role:** Execution role with basic Lambda permissions
- **CloudWatch Log Group:** `/aws/lambda/SetupLambda-SimpleFunction-*`

## Cleanup

To delete all deployed resources:

```bash
sam delete --stack-name SetupLambda
```

## Learn More

- [AWS SAM Documentation](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/)
- [SAM CLI Configuration](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-config.html)
- [SAM Template Specification](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification.html)
- [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)

