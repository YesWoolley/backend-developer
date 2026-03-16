# CI/CD and Docker Push to ACR Setup Guide

This guide explains how to set up GitHub Actions CI/CD pipeline to automatically build and push Docker images to Azure Container Registry (ACR) for different branches (dev/main).

## Overview

- **Development Environment**: For `dev` branch → Uses dev ACR, credentials, and Web App
- **Production Environment**: For `main` branch → Uses production ACR, credentials, and Web App
- **Automatic Deployment**: GitHub Actions automatically deploys to Azure Web App with dynamic SHA tags

---

## Step 1: Create GitHub Environments

1. Go to your GitHub repository
2. Click **Settings** → **Environments**
3. Click **New environment**

### Create Development Environment
- **Name**: `development`
- Click **Configure environment**
- Leave deployment branches as "No restriction" (or restrict to `dev` branch if preferred)

### Create Production Environment
- **Name**: `production`
- Click **Configure environment**
- Optionally restrict deployment branches to `main` only for extra security

---

## Step 2: Create Azure Service Principals

### For Development Environment

Run this command in Azure CLI (PowerShell):

```powershell
az ad sp create-for-rbac --name "github-actions-urban-intelligence-dev" --role "AcrPush" --scopes /subscriptions/463198b5-0215-4a6c-a34c-d7eddf85bef6/resourceGroups/urban-intelligence-dev-rg/providers/Microsoft.ContainerRegistry/registries/urbanintelligencedevacr --sdk-auth
```

**Output**: Copy the entire JSON output - this is your `AZURE_CREDENTIALS` for development.

**Important**: After creating the service principal, you need to grant it permission to deploy to the Web App. You can do this via Azure Portal UI or Azure CLI.

### Option A: Grant Permission via Azure Portal UI (Recommended)

1. **Find your service principal**:
   - Go to Azure Portal → **Microsoft Entra ID** (or **Azure Active Directory**)
   - Click **App registrations** in the left menu
   - Search for: `github-actions-urban-intelligence-dev`
   - Click on it and note the **Application (client) ID** (you'll need this to verify)

2. **Navigate to your Web App**:
   - Go to Azure Portal → **App Services**
   - Find and click: `urban-intelligence-dev-webapp-g0fcdncdb6axbafw`

3. **Grant permission**:
   - In the Web App's left menu, click **Access control (IAM)**
   - Click **"+ Add"** → **"Add role assignment"**
   - **Role tab**: Select **"Website Contributor"** → Click **"Next"**
   - **Members tab**: 
     - Select **"User, group, or service principal"**
     - Click **"+ Select members"**
     - Search for: `github-actions-urban-intelligence-dev`
     - Select it and click **"Select"**
     - Click **"Next"**
   - **Review + assign tab**: Review and click **"Review + assign"**

### Option B: Grant Permission via Azure CLI

```powershell
# Get the service principal ID (from the JSON output above, use the "clientId" value)
$SP_CLIENT_ID = "xxxx-xxxx-xxxx-xxxx"  # Replace with clientId from JSON output

# Grant Website Contributor role to the service principal on the Web App
az role assignment create `
  --assignee $SP_CLIENT_ID `
  --role "Website Contributor" `
  --scope /subscriptions/463198b5-0215-4a6c-a34c-d7eddf85bef6/resourceGroups/{dev-resource-group}/providers/Microsoft.Web/sites/urban-intelligence-dev-webapp-g0fcdncdb6axbafw
```

Replace `{dev-resource-group}` with your actual development resource group name.

### For Production Environment

**Production ACR Details:**
- **Subscription ID**: `463198b5-0215-4a6c-a34c-d7eddf85bef6`
- **Resource Group**: `urban-intelligence-prod-rg`
- **ACR Name**: `urbanintelligenceprodacr`

Run this exact command:

```powershell
az ad sp create-for-rbac --name "github-actions-urban-intelligence-prod" --role "AcrPush" --scopes /subscriptions/463198b5-0215-4a6c-a34c-d7eddf85bef6/resourceGroups/urban-intelligence-prod-rg/providers/Microsoft.ContainerRegistry/registries/urbanintelligenceprodacr --sdk-auth
```

**Output**: Copy the entire JSON output - this is your `AZURE_CREDENTIALS` for production.

**Important**: After creating the service principal, you need to grant it permission to deploy to the Web App. You can do this via Azure Portal UI or Azure CLI.

### Option A: Grant Permission via Azure Portal UI (Recommended)

1. **Find your service principal**:
   - Go to Azure Portal → **Microsoft Entra ID** (or **Azure Active Directory**)
   - Click **App registrations** in the left menu
   - Search for: `github-actions-urban-intelligence-prod`
   - Click on it and note the **Application (client) ID** (you'll need this to verify)

2. **Navigate to your Web App**:
   - Go to Azure Portal → **App Services**
   - Find and click: `urban-intelligence-prod-webapp-frdrghh9cfdmateu`

3. **Grant permission**:
   - In the Web App's left menu, click **Access control (IAM)**
   - Click **"+ Add"** → **"Add role assignment"**
   - **Role tab**: Select **"Website Contributor"** → Click **"Next"**
   - **Members tab**: 
     - Select **"User, group, or service principal"**
     - Click **"+ Select members"**
     - Search for: `github-actions-urban-intelligence-prod`
     - Select it and click **"Select"**
     - Click **"Next"**
   - **Review + assign tab**: Review and click **"Review + assign"**

### Option B: Grant Permission via Azure CLI

```powershell
# Get the service principal ID (from the JSON output above, use the "clientId" value)
$SP_CLIENT_ID = "xxxx-xxxx-xxxx-xxxx"  # Replace with clientId from JSON output

# Grant Website Contributor role to the service principal on the Web App
az role assignment create `
  --assignee $SP_CLIENT_ID `
  --role "Website Contributor" `
  --scope /subscriptions/463198b5-0215-4a6c-a34c-d7eddf85bef6/resourceGroups/da-assist-prod-rg/providers/Microsoft.Web/sites/urban-intelligence-prod-webapp-frdrghh9cfdmateu
```

**Expected JSON format:**
```json
{
  "clientId": "xxxx-xxxx-xxxx-xxxx",
  "clientSecret": "xxxx-xxxx-xxxx-xxxx",
  "subscriptionId": "463198b5-0215-4a6c-a34c-d7eddf85bef6",
  "tenantId": "65e6fd84-e889-4e52-8dc6-bde28e557b84",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
}
```

---

## Step 3: Add Secrets to GitHub Environments

### How to Find Your Azure Web App Name

The `AZURE_WEBAPP_NAME` is the name of your Azure Web App resource. You can find it by:

1. **From Azure Portal**:
   - Go to Azure Portal → **App Services**
   - Find your Web App in the list
   - The name is displayed at the top of the Overview page

2. **From Web App URL**:
   - Your Web App URL format: `{webapp-name}.azurewebsites.net`
   - Extract the part before `.azurewebsites.net`
   - Example: From `urban-intelligence-dev-webapp-g0fcdncdb6axbafw.australiaeast-01.azurewebsites.net`
   - Web App Name: `urban-intelligence-dev-webapp-g0fcdncdb6axbafw`

### Development Environment Secrets

Go to: **Settings** → **Environments** → **development** → **Add environment secret**

Add these 4 secrets:

1. **`AZURE_CREDENTIALS`**
   - Value: (Paste the entire JSON from Step 2 - Development)
   ```json
   {
     "clientId": "...",
     "clientSecret": "...",
     "subscriptionId": "463198b5-0215-4a6c-a34c-d7eddf85bef6",
     "tenantId": "...",
     ...
   }
   ```

2. **`ACR_REGISTRY_NAME`**
   - Value: `urbanintelligencedevacr` (or your dev ACR name)

3. **`IMAGE_NAME`**
   - Value: `urban-intelligence-backend`

4. **`AZURE_WEBAPP_NAME`**
   - Value: `urban-intelligence-dev-webapp-g0fcdncdb6axbafw`
   - **How to find**: Extract the Web App name from your Azure Web App URL
   - Example: From URL `urban-intelligence-dev-webapp-g0fcdncdb6axbafw.australiaeast-01.azurewebsites.net`
   - The name is the part before the first dot: `urban-intelligence-dev-webapp-g0fcdncdb6axbafw`

### Production Environment Secrets

Go to: **Settings** → **Environments** → **production** → **Add environment secret**

Add the same 4 secrets with production values:

1. **`AZURE_CREDENTIALS`**
   - Value: (Paste the entire JSON from Step 2 - Production)

2. **`ACR_REGISTRY_NAME`**
   - Value: (Your production ACR name, without `.azurecr.io`)

3. **`IMAGE_NAME`**
   - Value: `urban-intelligence-backend`

4. **`AZURE_WEBAPP_NAME`**
   - Value: `urban-intelligence-prod-webapp-frdrghh9cfdmateu`
   - **How to find**: Extract the Web App name from your Azure Web App URL
   - Example: From URL `urban-intelligence-prod-webapp-frdrghh9cfdmateu.australiaeast-01.azurewebsites.net`
   - The name is the part before the first dot: `urban-intelligence-prod-webapp-frdrghh9cfdmateu`

---

## Step 3.5: Add Test Database Connection String (Repository Secret)

Tests use the same database regardless of branch, so add this as a **repository-level secret** (not environment-specific):

1. Go to: **Settings** → **Secrets and variables** → **Actions** → **New repository secret**
2. Name: `TEST_DATABASE_CONNECTION_STRING`
3. Value: (paste your connection string **without outer quotes**)
   ```
   User Id=postgres.tiqrknobgjadsdhswgoa;Password=YOUR_PASSWORD;Server=aws-1-ap-south-1.pooler.supabase.com;Port=5432;Database=postgres
   ```

**Important**: 
- Remove the outer quotes when pasting into GitHub Secrets
- Replace `YOUR_PASSWORD` with your actual Supabase database password

---

## Step 4: Update CI/CD Workflow

The workflow file (`.github/workflows/ci-cd.yml`) should include:

```yaml
build-docker:
  name: Build and Push Docker Image to ACR
  needs: test
  runs-on: ubuntu-latest
  if: github.event_name == 'push' && (github.ref == 'refs/heads/main' || github.ref == 'refs/heads/dev' || github.ref == 'refs/heads/develop')
  environment: ${{ github.ref == 'refs/heads/main' && 'production' || 'development' }}
  
  env:
    ACR_REGISTRY_NAME: ${{ secrets.ACR_REGISTRY_NAME }}
    IMAGE_NAME: ${{ secrets.IMAGE_NAME }}
  
  steps:
    # ... rest of steps
    - name: Login to Azure
      uses: azure/login@v2
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
```

**Key point**: The `environment:` line automatically selects:
- `production` environment when pushing to `main` branch
- `development` environment when pushing to `dev` branch

---

## Quick Reference: Azure CLI Commands

### Development
```powershell
az ad sp create-for-rbac --name "github-actions-urban-intelligence-dev" --role "AcrPush" --scopes /subscriptions/463198b5-0215-4a6c-a34c-d7eddf85bef6/resourceGroups/da-assist-dev-rg/providers/Microsoft.ContainerRegistry/registries/daassistdevacr --sdk-auth
```

### Production
```powershell
az ad sp create-for-rbac --name "github-actions-urban-intelligence-prod" --role "AcrPush" --scopes /subscriptions/463198b5-0215-4a6c-a34c-d7eddf85bef6/resourceGroups/da-assist-prod-rg/providers/Microsoft.ContainerRegistry/registries/daassistprodacr --sdk-auth
```

---

## Troubleshooting

### Azure CLI not found
- Install: `winget install -e --id Microsoft.AzureCLI`
- Restart PowerShell after installation

### Service Principal already exists
- **For Development**: 
  ```powershell
  az ad sp list --display-name "github-actions-urban-intelligence-dev"
  az ad sp delete --id {appId}
  ```
- **For Production**:
  ```powershell
  az ad sp list --display-name "github-actions-urban-intelligence-prod"
  az ad sp delete --id {appId}
  ```
- Or use a different name: `--name "github-actions-urban-intelligence-dev-v2"`

### Permission denied
- Ensure you have "Owner" or "User Access Administrator" role on the subscription/resource group

### Web App deployment fails: "Resource doesn't exist"
This error means the service principal doesn't have permission to access the Web App. Fix it by granting permissions:

#### Option A: Via Azure Portal UI (Easiest)

1. **Go to your Web App**:
   - Azure Portal → **App Services** → Find your Web App
   - Development: `urban-intelligence-dev-webapp-g0fcdncdb6axbafw`
   - Production: `urban-intelligence-prod-webapp-frdrghh9cfdmateu`

2. **Grant permission**:
   - Click **Access control (IAM)** in the left menu
   - Click **"+ Add"** → **"Add role assignment"**
   - **Role tab**: Select **"Website Contributor"** → Click **"Next"**
   - **Members tab**: 
     - Select **"User, group, or service principal"**
     - Click **"+ Select members"**
     - Search for your service principal:
       - Development: `github-actions-urban-intelligence-dev`
       - Production: `github-actions-urban-intelligence-prod`
     - Select it and click **"Select"**
     - Click **"Next"**
   - **Review + assign tab**: Review and click **"Review + assign"**

#### Option B: Via Azure CLI

1. **Get the service principal client ID** from your `AZURE_CREDENTIALS` JSON (the `clientId` field)

2. **Grant Website Contributor role**:
   ```powershell
   # For Development
   az role assignment create `
     --assignee {clientId-from-json} `
     --role "Website Contributor" `
     --scope /subscriptions/463198b5-0215-4a6c-a34c-d7eddf85bef6/resourceGroups/{dev-rg}/providers/Microsoft.Web/sites/urban-intelligence-dev-webapp-g0fcdncdb6axbafw
   
   # For Production
   az role assignment create `
     --assignee {clientId-from-json} `
     --role "Website Contributor" `
     --scope /subscriptions/463198b5-0215-4a6c-a34c-d7eddf85bef6/resourceGroups/da-assist-prod-rg/providers/Microsoft.Web/sites/urban-intelligence-prod-webapp-frdrghh9cfdmateu
   ```

3. **Alternative**: Grant "Contributor" role on the entire resource group (broader permission):
   ```powershell
   az role assignment create `
     --assignee {clientId-from-json} `
     --role "Contributor" `
     --scope /subscriptions/463198b5-0215-4a6c-a34c-d7eddf85bef6/resourceGroups/{resource-group-name}
   ```

### Can't find production ACR
- Go to Azure Portal → Container registries
- Find your production ACR
- Copy the name (without `.azurecr.io`)
- Copy the resource group name
- Copy the subscription ID

---

## Image Tagging Strategy

The workflow automatically creates **multiple tags** for each Docker image pushed to ACR:

### Tag Types

1. **Branch-based tag** (primary tag):
   - `main` branch → `latest`
   - `dev` or `develop` branch → `dev`
   - Other branches → branch name (e.g., `feature-xyz`)

2. **Short SHA tag** (7 characters):
   - Example: `a1b2c3d`
   - Useful for referencing specific commits

3. **Full commit SHA tag**:
   - Example: `a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0`
   - Unique identifier for exact commit

### Example Tags

When pushing to `dev` branch with commit SHA `a1b2c3d4e5f6...`, the image will be tagged as:
- `daassistdevacr.azurecr.io/urban-intelligence-backend:dev`
- `daassistdevacr.azurecr.io/urban-intelligence-backend:a1b2c3d`
- `daassistdevacr.azurecr.io/urban-intelligence-backend:a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0`

When pushing to `main` branch, the image will be tagged as:
- `daassistprodacr.azurecr.io/urban-intelligence-backend:latest`
- `daassistprodacr.azurecr.io/urban-intelligence-backend:a1b2c3d`
- `daassistprodacr.azurecr.io/urban-intelligence-backend:a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0`

### Pulling Images

```bash
# Pull latest dev image
docker pull daassistdevacr.azurecr.io/urban-intelligence-backend:dev

# Pull specific commit
docker pull daassistdevacr.azurecr.io/urban-intelligence-backend:a1b2c3d

# Pull production latest
docker pull daassistprodacr.azurecr.io/urban-intelligence-backend:latest
```

---

## How It Works

### Development Branch (`dev`)
1. GitHub Actions workflow runs
2. The `environment: ${{ github.ref == 'refs/heads/main' && 'production' || 'development' }}` line selects `development`
3. Workflow uses secrets from the `development` environment
4. Docker image is pushed to `daassistdevacr.azurecr.io` with tag `dev`

### Production Branch (`main`)
1. GitHub Actions workflow runs
2. The `environment:` line selects `production`
3. Workflow uses secrets from the `production` environment
4. Docker image is pushed to `daassistprodacr.azurecr.io` with tag `latest`

---

## Setup Checklist

### Development Environment
- [ ] Created `development` environment in GitHub
- [ ] Created Azure Service Principal for dev
- [ ] **Granted "Website Contributor" role to service principal on Web App** (via Portal UI or CLI)
- [ ] Added `AZURE_CREDENTIALS` secret (full JSON)
- [ ] Added `ACR_REGISTRY_NAME` secret (your dev ACR name)
- [ ] Added `IMAGE_NAME` secret (`urban-intelligence-backend`)
- [ ] Added `AZURE_WEBAPP_NAME` secret (`urban-intelligence-dev-webapp-g0fcdncdb6axbafw`)

### Production Environment
- [ ] Created `production` environment in GitHub
- [ ] Created Azure Service Principal for production
- [ ] **Granted "Website Contributor" role to service principal on Web App** (via Portal UI or CLI)
- [ ] Added `AZURE_CREDENTIALS` secret (full JSON)
- [ ] Added `ACR_REGISTRY_NAME` secret (your prod ACR name)
- [ ] Added `IMAGE_NAME` secret (`urban-intelligence-backend`)
- [ ] Added `AZURE_WEBAPP_NAME` secret (`urban-intelligence-prod-webapp-frdrghh9cfdmateu`)

### Test Database
- [ ] Added `TEST_DATABASE_CONNECTION_STRING` repository secret (without quotes)

### Verification
- [ ] Tested by pushing to `dev` branch
- [ ] Tested by pushing to `main` branch

---

## Notes

- **Secrets are encrypted** and only accessible during workflow runs
- **Each environment has separate secrets** - dev and prod are completely isolated
- **Service Principal secrets expire** - set expiration to 24 months or create a reminder to rotate
- **Multiple tags per image** - Each push creates multiple tags (latest, SHA, timestamp) for flexibility
- **Tag format**: `{ACR_NAME}.azurecr.io/{IMAGE_NAME}:{TAG}`
- **Test database connection** is repository-level (shared across all branches)
- **Automatic deployment** - GitHub Actions automatically deploys to Azure Web App with dynamic SHA tags after each build
- **Web App name** - Extract from Azure Portal or Web App URL (part before `.azurewebsites.net`)

