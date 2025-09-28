# AWS IAM Permissions for ensure-ecr-repository Action

This document outlines the AWS IAM permissions required when using this GitHub Action with OIDC (OpenID Connect) and assuming an IAM role.

## Required IAM Permissions

The IAM role that your GitHub Actions workflow assumes must have the following permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:DescribeRepositories",
        "ecr:CreateRepository"
      ],
      "Resource": "*"
    }
  ]
}
```

### Permission Details

- **`ecr:DescribeRepositories`** - Required to check if the ECR repository already exists
- **`ecr:CreateRepository`** - Required to create the ECR repository if it doesn't exist

## Restricting Permissions (Recommended)

For better security, you can restrict the permissions to specific repositories:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:DescribeRepositories",
        "ecr:CreateRepository"
      ],
      "Resource": "arn:aws:ecr:REGION:ACCOUNT_ID:repository/REPOSITORY_NAME"
    }
  ]
}
```

Replace:
- `REGION` with your AWS region (e.g., `us-east-1`)
- `ACCOUNT_ID` with your AWS account ID
- `REPOSITORY_NAME` with your repository name or use `*` for all repositories

## Example: Complete OIDC Setup

### 1. Create the IAM Role

Create an IAM role with a trust policy that allows GitHub Actions to assume it:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:YOUR_GITHUB_ORG/YOUR_GITHUB_REPO:*"
        }
      }
    }
  ]
}
```

### 2. Attach the Required Permissions

Attach the ECR permissions policy to the role:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:DescribeRepositories",
        "ecr:CreateRepository"
      ],
      "Resource": "*"
    }
  ]
}
```

### 3. Use in GitHub Actions Workflow

```yaml
name: Ensure ECR Repository
on:
  push:
    branches: [main]

permissions:
  id-token: write
  contents: read

jobs:
  ensure-ecr:
    runs-on: ubuntu-latest
    steps:
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::ACCOUNT_ID:role/YOUR_ROLE_NAME
          aws-region: us-east-1

      - name: Ensure ECR Repository Exists
        uses: koalaops/ensure-ecr-repository@v1
        with:
          repository-name: my-app
          aws-region: us-east-1
```

## Additional Considerations

### Using with Docker Build and Push

If you're using this action as part of a Docker build and push workflow, your IAM role will also need these additional permissions:

```json
{
  "Effect": "Allow",
  "Action": [
    "ecr:GetAuthorizationToken",
    "ecr:BatchCheckLayerAvailability",
    "ecr:GetDownloadUrlForLayer",
    "ecr:BatchGetImage",
    "ecr:PutImage",
    "ecr:InitiateLayerUpload",
    "ecr:UploadLayerPart",
    "ecr:CompleteLayerUpload"
  ],
  "Resource": "*"
}
```

Note: `ecr:GetAuthorizationToken` must have `Resource: "*"` as it doesn't support resource-level permissions.

### Troubleshooting

If you encounter an "AccessDeniedException" error, verify:

1. The IAM role has the correct permissions
2. The trust policy allows your GitHub repository to assume the role
3. The OIDC provider is correctly configured in your AWS account
4. The `aws-region` input matches the region where your ECR repository should exist