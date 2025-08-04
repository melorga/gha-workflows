# Shared GitHub Actions Workflows

[![Terraform](https://img.shields.io/badge/Terraform-1.5.7+-623CE4?style=for-the-badge&logo=terraform)](https://terraform.io/)
[![Terragrunt](https://img.shields.io/badge/Terragrunt-0.58.7+-623CE4?style=for-the-badge&logo=terraform)](https://terragrunt.gruntwork.io/)
[![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-CI%2FCD-2088FF?style=for-the-badge&logo=github-actions)](https://github.com/features/actions)

Reusable GitHub Actions workflows for Terraform and Terragrunt CI/CD pipelines. These workflows provide enterprise-grade automation with security scanning, testing, and deployment capabilities.

## 🚀 Available Workflows

### 1. Terraform CI/CD Workflow

Comprehensive Terraform workflow with validation, planning, security scanning, and automated deployment.

**Features:**
- ✅ Terraform format checking and validation
- ✅ Security scanning with tfsec and tflint  
- ✅ Automated plan generation with PR comments
- ✅ Integration testing with Terratest
- ✅ Conditional auto-apply on main branch
- ✅ Multi-environment support

### 2. Terragrunt CI/CD Workflow

Multi-environment Terragrunt workflow for managing complex infrastructure deployments.

**Features:**
- ✅ Terragrunt format checking and validation
- ✅ Plan-all and apply-all operations
- ✅ Environment-specific deployments
- ✅ AWS credentials management
- ✅ Non-interactive execution

## 📋 Usage Examples

### Using Terraform Workflow

Create `.github/workflows/terraform.yml` in your repository:

```yaml
name: Infrastructure CI/CD

on:
  push:
    branches: [ main, develop ]
    paths: [ 'infrastructure/**' ]
  pull_request:
    branches: [ main ]
    paths: [ 'infrastructure/**' ]

jobs:
  terraform:
    name: Terraform CI/CD
    uses: melorga/gha-workflows/.github/workflows/terraform.yml@main
    with:
      terraform_version: '1.5.7'
      working_directory: './infrastructure'
      environment: ${{ github.ref == 'refs/heads/main' && 'prod' || 'dev' }}
      auto_apply: ${{ github.ref == 'refs/heads/main' }}
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      AWS_REGION: ${{ secrets.AWS_REGION }}
```

### Using Terragrunt Workflow

Create `.github/workflows/terragrunt.yml` in your repository:

```yaml
name: Live Infrastructure CI/CD

on:
  push:
    branches: [ main ]
    paths: [ 'prod/**', 'stage/**', 'dev/**' ]
  pull_request:
    branches: [ main ]
    paths: [ 'prod/**', 'stage/**', 'dev/**' ]

jobs:
  terragrunt:
    name: Terragrunt CI/CD
    uses: melorga/gha-workflows/.github/workflows/terragrunt.yml@main
    with:
      terragrunt_version: '0.58.7'
      terraform_version: '1.5.7'
      working_directory: './prod/us-east-1'
      environment: 'production'
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      AWS_REGION: ${{ secrets.AWS_REGION }}
```

## 🔧 Workflow Parameters

### Terraform Workflow Inputs

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `terraform_version` | string | `1.5.7` | Terraform version to use |
| `working_directory` | string | `.` | Working directory for Terraform |
| `environment` | string | `dev` | Environment to deploy to |
| `auto_apply` | boolean | `false` | Auto apply changes on main branch |

### Terragrunt Workflow Inputs

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `terragrunt_version` | string | `0.58.7` | Terragrunt version to use |
| `terraform_version` | string | `1.5.7` | Terraform version to use |
| `working_directory` | string | `.` | Working directory for Terragrunt |
| `environment` | string | `dev` | Environment to deploy to |

### Required Secrets

| Secret | Description |
|--------|-------------|
| `AWS_ACCESS_KEY_ID` | AWS Access Key ID |
| `AWS_SECRET_ACCESS_KEY` | AWS Secret Access Key |
| `AWS_REGION` | AWS Region (optional, defaults to us-east-1) |

## 🛡️ Security Features

- **Security Scanning**: Automated tfsec scanning for security vulnerabilities
- **Linting**: tflint validation for best practices
- **Credential Management**: Secure AWS credentials handling
- **Environment Protection**: GitHub environment protection rules
- **Plan Review**: Required PR reviews with plan output

## 🔄 Workflow Triggers

### Terraform Workflow Jobs

1. **Validate** (Always runs)
   - Format checking
   - Terraform validation
   - Security scanning
   - Linting

2. **Plan** (Pull requests only)
   - Generate execution plan
   - Comment plan on PR

3. **Apply** (Main branch pushes only)
   - Apply infrastructure changes
   - Requires environment approval

4. **Test** (Pull requests only)
   - Run integration tests
   - Validate deployments

### Terragrunt Workflow Jobs

1. **Validate** (Always runs)
   - HCL format checking
   - Terragrunt validation

2. **Plan** (Pull requests only)
   - Plan all configurations

3. **Apply** (Main branch pushes only)
   - Apply all configurations

## 📝 Best Practices

### Repository Setup

1. **Create GitHub Environments**
   ```bash
   # Create environments: dev, stage, prod
   # Configure protection rules and required reviewers
   ```

2. **Set Repository Secrets**
   ```bash
   # Add AWS credentials to repository secrets
   # Configure environment-specific variables
   ```

3. **Configure Branch Protection**
   ```bash
   # Require PR reviews
   # Require status checks to pass
   # Restrict pushes to main branch
   ```

### Workflow Organization

- Use path filters to trigger workflows only for relevant changes
- Separate workflows for different environments or components
- Use matrix strategies for multi-environment deployments

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Test your changes with example repositories
4. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see [LICENSE](LICENSE) for details.

---

> These workflows provide production-ready CI/CD automation for Terraform and Terragrunt, following industry best practices for security, testing, and deployment.
