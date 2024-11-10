# Apigee Proxy Deployment Workflow

## Overview
This GitHub Actions workflow automates the deployment of Apigee API proxies across non-production and production environments with proper approval gates and validations.

## Workflow Structure

### 1. Input Parameters
```yaml
inputs:
  proxy_name:        # Name of the API proxy to deploy
  proxy_directory:   # Directory containing proxy files (default: 'apiproxy')
  environment_group: # Target environment group (default, edd, homerun, etc.)
  environment_type:  # Target environment (dev, test-env, test, uat, prod)
```

### 2. Workflow Jobs

#### A. Validation Phase
- **validate_deployment_inputs**
  - Validates environment configuration
  - Checks environment group and type
  - Determines if it's a production deployment
  - Status checks and validation gates

#### B. Authentication Phase
- **Setup_Auth**
  - Sets up authentication for both non-prod and prod environments
  - Manages GCP tokens for both environments
  - Handles credential management

#### C. Non-Production Deployment Phase
1. **Validate_API_Proxy**
   - Runs apigeelint validation
   - Ensures proxy meets quality standards
   - Validates proxy structure

2. **Build_And_Upload_NonProd**
   - Creates API proxy bundle
   - Uploads to non-prod organization
   - Manages artifact creation and storage
   - Gets revision number

3. **Deploy_To_NonProd**
   - Deploys to non-prod environment
   - Handles environment-specific configurations
   - Manages deployment process

4. **Verify_NonProd_Deployment**
   - Verifies successful deployment
   - Checks deployment status
   - Ensures proxy is accessible

#### D. Production Approval Phase
- **Request_Production_Approval**
  - Initiates production deployment approval
  - Manages environment protection rules
  - Waits for manual approval
  - Shows approval button in GitHub UI

#### E. Production Deployment Phase
1. **Build_And_Upload_Prod**
   - Downloads artifacts from non-prod
   - Uploads to production organization
   - Maintains version consistency

2. **Deploy_To_Production**
   - Deploys approved version to production
   - Manages production environment specifics
   - Handles production deployment verification

## Environment Setup

### Required Environments
1. Development/Test Environments
2. Production Approval Environment
   ```
   Name: production-approval
   Protection rules: Required reviewers
   ```
3. Production Environment
   ```
   Name: production
   Environment URL: Auto-generated
   ```

### Required Secrets
```yaml
APIGEE_ORG:                    # Non-prod Apigee organization
APIGEE_ORG_PROD:              # Production Apigee organization
WORKLOAD_IDENTITY_PROVIDER:    # Non-prod GCP identity provider
WORKLOAD_IDENTITY_PROVIDER_PROD: # Production GCP identity provider
SERVICE_ACCOUNT:              # Non-prod service account
SERVICE_ACCOUNT_PROD:         # Production service account
```

## Workflow Features

### 1. Status Tracking
- Detailed logging with emojis for visual feedback
- Group markers for organized logs
- Clear success/failure indicators

### 2. Error Handling
- Comprehensive error checking
- Detailed error messages
- Retry mechanisms for API calls

### 3. Security Features
- Environment-specific authentication
- Secure token management
- Protected production deployments

### 4. Artifact Management
- Maintains proxy bundle consistency
- Proper version tracking
- Secure artifact storage

## Usage Example

```yaml
name: Deploy API Proxy
on:
  workflow_dispatch:
    inputs:
      proxy_name:
        description: "API Proxy Name"
        required: true
      environment_type:
        description: "Target Environment"
        type: choice
        options:
          - dev
          - test
          - prod

jobs:
  deploy:
    uses: ./.github/workflows/Reusable-proxy-deploy.yml@main
    with:
      proxy_name: ${{ inputs.proxy_name }}
      environment_type: ${{ inputs.environment_type }}
      environment_group: "default"
    secrets: inherit
```

## Production Deployment Flow
1. Non-prod deployment succeeds
2. Production approval request created
3. Reviewers notified via GitHub
4. Approval through GitHub UI
5. Automatic production deployment after approval

## Best Practices
1. Always validate in non-prod first
2. Use meaningful proxy names
3. Keep proxy bundle structure consistent
4. Monitor deployment logs
5. Review changes before production approval

## Troubleshooting
1. Check proxy bundle structure
2. Verify environment configurations
3. Ensure proper authentication
4. Review deployment logs
5. Check GCP permissions

Would you like me to add any specific sections or provide more details about any particular aspect?
