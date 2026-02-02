# Deploy Dev Stack

A GitHub Action for deploying development environments to AWS using CloudFormation and Helm.

## Overview

This action automates the deployment of development stacks by:
1. Validating the release name and Docker image
2. Creating AWS infrastructure via CloudFormation
3. Deploying the application to EKS using Helm

## Prerequisites

- AWS account with appropriate IAM permissions
- EKS cluster configured
- ECR registry with Docker images
- Helm chart available in ECR
- CloudFormation template for AWS resources

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `release-name` | Release name (must start with `dev-`) | Yes | - |
| `tag` | Docker image tag to deploy | Yes | - |
| `service-name` | Service name (e.g., `hello-k8s`) | Yes | - |
| `helm-release-path` | Path to HelmRelease YAML for values | Yes | - |
| `namespace` | Kubernetes namespace | No | `bisnow-apps` |
| `eks-cluster` | EKS cluster name | No | `bisnow-non-prod-eks` |
| `ecr-registry` | ECR registry URL | No | `560285300220.dkr.ecr.us-east-1.amazonaws.com` |
| `aws-account` | AWS account for role assumption | No | `bisnow` |
| `cloudformation-template` | Path to CloudFormation template | No | `./aws-resources.yaml` |
| `helm-chart-version` | Helm chart version | No | `4.1.5` |

## Outputs

| Output | Description |
|--------|-------------|
| `hostname` | The deployed application hostname |
| `release-name` | Full Helm release name |

## Usage

```yaml
- name: Deploy dev environment
  uses: bisnow/github-actions-deploy-dev-stack@v1
  with:
    release-name: dev-123
    tag: v1.2.3
    service-name: my-service
    helm-release-path: ./helm/release.yaml
```

### Complete Example

```yaml
name: Deploy to Dev

on:
  workflow_dispatch:
    inputs:
      dev-number:
        description: 'Dev environment number'
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy dev stack
        uses: bisnow/github-actions-deploy-dev-stack@v1
        with:
          release-name: dev-${{ github.event.inputs.dev-number }}
          tag: ${{ github.sha }}
          service-name: hello-k8s
          helm-release-path: ./k8s/helmrelease.yaml
          namespace: bisnow-apps
          eks-cluster: bisnow-non-prod-eks
```

## What It Does

1. **Validates Release Name**: Ensures the release name starts with `dev-`
2. **Validates Image**: Checks that the specified Docker image exists in ECR
3. **AWS Authentication**: Assumes the appropriate AWS role
4. **CloudFormation Deployment**: Creates or updates AWS resources (IAM roles, etc.)
5. **Kubectl Configuration**: Configures kubectl to connect to the EKS cluster
6. **ECR Login**: Authenticates Helm with ECR registry
7. **Helm Deployment**: Deploys the application with extracted values and dynamic configuration

## Generated Resources

The action creates the following resources:

- **Hostname**: `{release-name}-{service-name}.non-prod.bisnow.cloud`
- **IAM Role**: `arn:aws:iam::560285300220:role/{release-name}-{service-name}`
- **CloudFormation Stack**: `{release-name}-{service-name}`
- **Helm Release**: `{release-name}-{service-name}`

## Notes

- Release names must start with `dev-` (e.g., `dev-1`, `dev-23`, `dev-feature-branch`)
- The action waits up to 10 minutes for Helm deployment to complete
- Values are extracted from the HelmRelease YAML and merged with runtime overrides
- Tags are applied to CloudFormation resources for tracking (cdci, Environment, Service)

## Dependencies

This action uses the following actions:
- `bisnow/github-actions-validate-release@v1.1`
- `bisnow/github-actions-assume-role-for-environment@v2.1`
- `aws-actions/aws-cloudformation-github-deploy@v1`
