# ARM Template Deployment and GitHub Actions CI/CD for Azure Web App

This documentation describes how to deploy the ASP.NET Core MVC web application using an Azure Resource Manager (ARM) template and automate deployments with GitHub Actions.

## 1. ARM Template Deployment

The ARM template (`ARM/deploy.json`) provisions the required Azure resources for the web app, including:
- **App Service Plan**
- **App Service (Web App)**
- **App Insights Integration**
- **Publishing Credentials Policies**
- **Web App Configuration**
- **Host Name Bindings**

### Deployment Steps

1. **Sign in to Azure:**
   ```powershell
   az login
   ```
2. **Set your subscription (if needed):**
   ```powershell
   az account set --subscription <your-subscription-id>
   ```
3. **Create a resource group (if not existing):**
   ```powershell
   az group create --name <resource-group-name> --location "West Europe"
   ```
4. **Deploy the ARM template:**
   ```powershell
   az deployment group create \
     --resource-group <resource-group-name> \
     --template-file ARM/deploy.json \
     --parameters ARM/deploy.parameters.json
   ```

## 2. GitHub Actions Workflow for Azure Deployment

A GitHub Actions workflow can automate build and deployment to Azure App Service. Place the following YAML file in `.github/workflows/azure-webapp-deploy.yml`:

```yaml
name: Build and Deploy ASP.NET Core app to Azure Web App

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'

      - name: Restore dependencies
        run: dotnet restore MyMvcApp-Contact-Databse-Application/MyMvcApp.csproj

      - name: Build
        run: dotnet build MyMvcApp-Contact-Databse-Application/MyMvcApp.csproj --configuration Release --no-restore

      - name: Publish
        run: dotnet publish MyMvcApp-Contact-Databse-Application/MyMvcApp.csproj --configuration Release --output ./publish --no-build

      - name: Deploy to Azure Web App
        uses: azure/webapps-deploy@v3
        with:
          app-name: ${{ secrets.AZURE_WEBAPP_NAME }}
          slot-name: 'production'
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
          package: ./publish
```

### Required GitHub Secrets
- `AZURE_WEBAPP_NAME`: Name of your Azure Web App (e.g., `my-app-lena`)
- `AZURE_WEBAPP_PUBLISH_PROFILE`: The publish profile XML from the Azure Portal (App Service > Get publish profile)

## 3. Additional Notes
- Update the workflow file as needed for your project structure or environment.
- The ARM template parameters can be customized in `ARM/deploy.parameters.json`.
- For more details, see [Azure documentation](https://docs.microsoft.com/azure/azure-resource-manager/templates/overview) and [GitHub Actions for Azure](https://docs.microsoft.com/azure/app-service/deploy-github-actions).
