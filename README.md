# Shared GitHub Actions Workflows

[![Terraform](https://img.shields.io/badge/Terraform-1.15.1+-623CE4?style=for-the-badge&logo=terraform)](https://terraform.io/)
[![Terragrunt](https://img.shields.io/badge/Terragrunt-1.0.3+-623CE4?style=for-the-badge&logo=terraform)](https://terragrunt.gruntwork.io/)
[![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-CI%2FCD-2088FF?style=for-the-badge&logo=github-actions)](https://github.com/features/actions)

Reusable GitHub Actions workflows for Terraform and Terragrunt CI/CD pipelines. These workflows provide enterprise-grade automation with security scanning, testing, and deployment capabilities, using AWS OIDC for short-lived, keyless authentication.

## Available Workflows

### 1. Terraform CI/CD Workflow

Comprehensive Terraform workflow with validation, planning, security scanning, and automated deployment.

**Features:**
- Terraform format checking and validation
- Security scanning with Trivy (IaC config) and tflint
- SARIF results uploaded to GitHub code scanning
- Automated plan generation with PR comments
- Plan artifact handed off to apply (no plan/apply drift)
- Integration testing with Terratest
- Deployment gated on push-to-main
- Multi-environment support via GitHub Environments

### 2. Terragrunt CI/CD Workflow

Multi-environment Terragrunt workflow for managing complex infrastructure deployments.

**Features:**
- Terragrunt format checking and validation
- `terragrunt run-all validate|plan|apply` (non-deprecated subcommands)
- Environment-specific deployments
- AWS OIDC credentials (no long-lived secrets)
- Non-interactive execution

## Usage Examples

Both workflows authenticate to AWS using OIDC. Pre-create an IAM role in your AWS account whose trust policy permits `token.actions.githubusercontent.com` for your repository, then pass its ARN via `aws_role_arn`.

### Using Terraform Workflow

Create `.github/workflows/terraform.yml` in your repository:

```yaml
name: Infrastructure CI/CD

on:
  push:
    branches: [ main ]
    paths: [ 'infrastructure/**' ]
  pull_request:
    branches: [ main ]
    paths: [ 'infrastructure/**' ]

permissions:
  contents: read
  id-token: write
  pull-requests: write

jobs:
  terraform:
    name: Terraform CI/CD
    uses: melorga/gha-workflows/.github/workflows/terraform.yml@main
    with:
      terraform_version: '1.15.1'
      working_directory: './infrastructure'
      environment: ${{ github.ref == 'refs/heads/main' && 'prod' || 'dev' }}
      aws_role_arn: arn:aws:iam::123456789012:role/github-actions-terraform
      aws_region: us-east-1
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

permissions:
  contents: read
  id-token: write
  pull-requests: write

jobs:
  terragrunt:
    name: Terragrunt CI/CD
    uses: melorga/gha-workflows/.github/workflows/terragrunt.yml@main
    with:
      terragrunt_version: '1.0.3'
      terraform_version: '1.15.1'
      working_directory: './prod/us-east-1'
      environment: 'production'
      aws_role_arn: arn:aws:iam::123456789012:role/github-actions-terragrunt
      aws_region: us-east-1
```

## Workflow Parameters

### Terraform Workflow Inputs

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `terraform_version` | string | `1.15.1` | Terraform version to use |
| `working_directory` | string | `.` | Working directory for Terraform |
| `environment` | string | `dev` | Environment (maps to GitHub Environment) |
| `aws_role_arn` | string | _none_ | IAM role ARN to assume via OIDC |
| `aws_region` | string | `us-east-1` | AWS region |

### Terragrunt Workflow Inputs

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `terragrunt_version` | string | `1.0.3` | Terragrunt version to use |
| `terraform_version` | string | `1.15.1` | Terraform version to use |
| `working_directory` | string | `.` | Working directory for Terragrunt |
| `environment` | string | `dev` | Environment (maps to GitHub Environment) |
| `aws_role_arn` | string | _none_ | IAM role ARN to assume via OIDC |
| `aws_region` | string | `us-east-1` | AWS region |

> **Note:** Terragrunt 1.0 introduced breaking changes (CAS, `hcl fmt`, experiments). If you are upgrading from a 0.x default, review the [Terragrunt 1.0 migration notes](https://terragrunt.gruntwork.io/docs/migrate/migrating-from-0.x/) before bumping.

### Required Permissions in Caller Workflow

| Permission | Reason |
|------------|--------|
| `contents: read` | Checkout source code |
| `id-token: write` | Request OIDC token for AWS |
| `pull-requests: write` | Post plan output as PR comment |

No long-lived AWS access keys are required.

## Pinned Action Versions

These reusable workflows currently pin the following third-party actions (May 2026):

| Action | Version |
|--------|---------|
| `actions/checkout` | `v6` |
| `hashicorp/setup-terraform` | `v4` |
| `aws-actions/configure-aws-credentials` | `v6` |
| `terraform-linters/setup-tflint` | `v6` |
| `aquasecurity/trivy-action` | `v0.36.0` |
| `github/codeql-action/upload-sarif` | `v4` |
| `actions/upload-artifact` | `v7` |
| `actions/download-artifact` | `v8` |
| `reviewdog/action-actionlint` | `v1.72.0` |

## Security Features

- **Security Scanning**: Trivy IaC scanning with SARIF upload to GitHub Security tab
- **Linting**: tflint validation for best practices
- **Keyless Auth**: AWS OIDC (no `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` secrets)
- **Environment Protection**: GitHub environment protection rules
- **Plan Review**: PR comments with plan output, plan artifact reused on apply (no drift)

## Workflow Triggers

### Terraform Workflow Jobs

1. **Validate** (Always runs)
   - Format checking, validation, tflint, Trivy IaC scan
2. **Plan** (Pull requests only)
   - Generates `tfplan`, uploads as artifact, comments plan on PR
3. **Apply** (Push to `main` only)
   - Downloads `tfplan` artifact and applies it
4. **Test** (Pull requests only)
   - Runs Terratest integration tests

### Terragrunt Workflow Jobs

1. **Validate** (Always runs) — `terragrunt run-all validate`
2. **Plan** (Pull requests only) — `terragrunt run-all plan`
3. **Apply** (Push to `main` only) — `terragrunt run-all apply`

## Versioning

Once tagged, consumers should pin the workflow to a specific release rather than tracking `@main`:

```yaml
uses: melorga/gha-workflows/.github/workflows/terraform.yml@v1.0.0
```

Tag releases follow semver. Breaking changes (input renames, removed inputs, behavior changes that require caller updates) bump the major version. Non-breaking improvements bump minor or patch. Until a `v1.0.0` tag is published, callers may pin to `@main` or to a specific commit SHA for reproducibility.

## Best Practices

### Repository Setup

1. **Create an OIDC IAM Role in AWS**
   - Trust `token.actions.githubusercontent.com`
   - Restrict `sub` to `repo:<org>/<repo>:*` (or per-environment)
2. **Create GitHub Environments** (`dev`, `stage`, `prod`)
   - Configure required reviewers and protection rules
3. **Configure Branch Protection**
   - Require PR reviews
   - Require status checks to pass
   - Restrict pushes to `main`

### Workflow Organization

- Use path filters to trigger workflows only for relevant changes
- Separate workflows for different environments or components
- Use matrix strategies for multi-environment deployments

## Contributing

1. Fork the repository
2. Create a feature branch
3. Test your changes with example repositories
4. Submit a pull request

## License

This project is licensed under the MIT License - see [LICENSE](LICENSE) for details.

---

> These workflows provide production-ready CI/CD automation for Terraform and Terragrunt, following industry best practices for security, testing, and deployment.
