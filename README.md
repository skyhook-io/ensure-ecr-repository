# Ensure ECR Repository Exists

A GitHub Action that ensures an Amazon ECR repository exists, creating it if necessary. This action is idempotent - it will create the repository only if it doesn't already exist.

## Features

- ✅ **Idempotent**: Safe to run multiple times - only creates if repository doesn't exist
- ✅ **Zero UI errors**: Uses proper error handling to avoid GitHub Actions UI noise
- ✅ **Comprehensive error handling**: Distinguishes between permission issues, missing repos, and other errors
- ✅ **Outputs**: Provides repository URI and existence status for downstream steps
- ✅ **Fast**: Uses AWS CLI directly for minimal overhead

## Usage

### Basic Example

```yaml
- name: Ensure ECR Repository Exists
  uses: skyhook-io/ensure-ecr-repository@v1
  with:
    repository-name: my-app
    aws-region: us-east-1
```

### Complete CI/CD Example

```yaml
name: Build and Push to ECR

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Ensure ECR Repository Exists
        id: ecr-repo
        uses: skyhook-io/ensure-ecr-repository@v1
        with:
          repository-name: my-app
          aws-region: us-east-1

      - name: Login to Amazon ECR
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and push Docker image
        run: |
          docker build -t ${{ steps.ecr-repo.outputs.repository-uri }}:latest .
          docker push ${{ steps.ecr-repo.outputs.repository-uri }}:latest
```

### Using Outputs

```yaml
- name: Ensure ECR Repository Exists
  id: ensure-repo
  uses: skyhook-io/ensure-ecr-repository@v1
  with:
    repository-name: my-service
    aws-region: us-west-2

- name: Use repository information
  run: |
    echo "Repository existed: ${{ steps.ensure-repo.outputs.repository-exists }}"
    echo "Repository URI: ${{ steps.ensure-repo.outputs.repository-uri }}"
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `repository-name` | The name of the ECR repository to ensure exists | ✅ | - |
| `aws-region` | The AWS region where the ECR repository should exist | ✅ | - |

## Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `repository-exists` | Whether the repository existed before this action ran | `true` or `false` |
| `repository-uri` | The full URI of the ECR repository | `123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app` |

## Prerequisites

### AWS Credentials

This action requires AWS credentials to be configured. You can use:

1. **aws-actions/configure-aws-credentials** (simple):
   ```yaml
   - uses: aws-actions/configure-aws-credentials@v4
     with:
       aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
       aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
       aws-region: us-east-1
   ```

2. **OIDC with IAM roles** (most secure):
   ```yaml
   - uses: aws-actions/configure-aws-credentials@v4
     with:
       role-to-assume: arn:aws:iam::123456789012:role/GitHubActions
       aws-region: us-east-1
   ```

### Required AWS Permissions

The AWS credentials must have the following permissions:

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

## Error Handling

The action handles various error scenarios:

- **Repository doesn't exist**: Creates the repository ✅
- **Repository already exists**: Skips creation (no error) ✅
- **Access denied**: Fails with clear error message ❌
- **Invalid region/parameters**: Fails with validation error ❌
- **Network/AWS issues**: Fails with detailed error information ❌

## Why Use This Action?

### The Problem

When pushing Docker images to Amazon ECR, you often encounter:
```
Error: Repository does not exist for repository with name 'my-app'
```

### Traditional Solutions
1. **Manual creation**: Create repositories manually in AWS console
2. **Terraform/CDK**: Requires infrastructure changes for each new service
3. **Inline scripts**: Error-prone, repetitive, clutters workflows

### This Action's Solution
- **Zero infrastructure changes** needed for new services
- **No manual steps** required
- **Clean workflows** with reusable, tested logic
- **Robust error handling** prevents cryptic failures

## Comparison with Alternatives

| Approach | Pros | Cons |
|----------|------|------|
| **This Action** | ✅ Zero setup<br>✅ Reusable<br>✅ Robust errors | ⚠️ Runtime dependency |
| **Terraform** | ✅ Infrastructure as code | ❌ PR needed per service<br>❌ More complex |
| **Manual creation** | ✅ Simple | ❌ Not automated<br>❌ Error-prone |
| **Inline scripts** | ✅ Direct control | ❌ Repetitive<br>❌ Error-prone |

## Contributing

We welcome contributions! Feel free to:
- 🐛 **Report bugs** by [creating an issue](https://github.com/skyhook-io/ensure-ecr-repository/issues)
- 💡 **Suggest features** by [creating an issue](https://github.com/skyhook-io/ensure-ecr-repository/issues)  
- 🛠️ **Submit pull requests** with improvements or fixes

No special process required - just fork, make your changes, and submit a PR!

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

- 🐛 **Bug reports**: [Create an issue](https://github.com/skyhook-io/ensure-ecr-repository/issues)
- 💡 **Feature requests**: [Create an issue](https://github.com/skyhook-io/ensure-ecr-repository/issues)

---

Made with ❤️ by [Skyhook](https://github.com/skyhook-io) 