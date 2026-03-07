---
layout: default
title: "03e — Infrastructure as Code"
nav_order: 9
description: "Design and Implement Infrastructure as Code — AZ-400 Exam Domain 3 (50–55% of exam) — IaC design, implementation, and management."
permalink: /03e-infrastructure-as-code/
mermaid: true
---

# 3e: Design and Implement Infrastructure as Code (IaC)

> 📁 [← Back to Home](/az-400-study-notes/)

[← Back to Domain 3](/az-400-study-notes/03-build-and-release-pipelines/)

---

## Configuration Management Technologies

| Technology | Type | Scope | Language | Idempotent |
|-----------|------|-------|----------|-----------|
| **Bicep** | Declarative | Azure resources | Bicep DSL | ✅ Yes |
| **ARM Templates** | Declarative | Azure resources | JSON | ✅ Yes |
| **Terraform** | Declarative | Multi-cloud | HCL | ✅ Yes |
| **Azure Automation DSC** | Declarative | OS configuration | PowerShell DSC | ✅ Yes |
| **Ansible** | Declarative/Imperative | Multi-cloud + OS | YAML | ✅ Mostly |
| **Chef** | Declarative | OS configuration | Ruby DSL | ✅ Yes |
| **Puppet** | Declarative | OS configuration | Puppet DSL | ✅ Yes |

**Choosing the right tool:**
- **Cloud resource provisioning** → Bicep, ARM, or Terraform
- **OS / application configuration** → Azure Automation DSC, Ansible
- **Multi-cloud** → Terraform, Pulumi
- **Azure-native, simpler syntax** → Bicep (preferred over raw ARM)

---

## IaC Strategy

### Source Control for IaC

```
infrastructure/
├── environments/
│   ├── dev/
│   │   ├── main.bicep
│   │   └── parameters.json
│   ├── staging/
│   │   ├── main.bicep
│   │   └── parameters.json
│   └── prod/
│       ├── main.bicep
│       └── parameters.json
├── modules/
│   ├── network/
│   │   └── vnet.bicep
│   ├── compute/
│   │   └── appservice.bicep
│   └── database/
│       └── sqlserver.bicep
└── scripts/
    └── validate.sh
```

**Best practices:**
- Store IaC in the **same repo as application code** (or a dedicated infra repo)
- Use **environment-specific parameter files** — never hardcode values
- **PR reviews** required for infrastructure changes (same as code)
- **Tag resources** for cost tracking and management
- Use **modules** for reusable components

### Automating IaC Testing and Deployment

```
IaC Code Change (PR)
         │
         ▼
  [Validation Pipeline]
  - az bicep build (syntax check)
  - az deployment what-if (preview changes)
  - Template linting (PSRule, checkov)
  - Security scanning (tfsec, Checkov)
         │
  PR Approved
         │
         ▼
  [Deployment Pipeline]
  - Deploy to Dev (auto)
  - Integration tests
  - Deploy to Staging (auto)
  - Smoke tests
  - Deploy to Prod (manual approval)
```

---

## Azure Bicep

Bicep is a **domain-specific language (DSL)** that compiles to ARM JSON. It's the preferred modern approach for Azure IaC.

### Basic Bicep Structure

```bicep
// main.bicep

// Parameters
@description('Environment name')
@allowed(['dev', 'staging', 'prod'])
param environment string = 'dev'

param location string = resourceGroup().location

@secure()
param sqlAdminPassword string

// Variables
var appServicePlanName = 'asp-${environment}-${uniqueString(resourceGroup().id)}'
var webAppName = 'app-${environment}-${uniqueString(resourceGroup().id)}'

// Resources
resource appServicePlan 'Microsoft.Web/serverfarms@2023-12-01' = {
  name: appServicePlanName
  location: location
  sku: {
    name: environment == 'prod' ? 'P1v3' : 'B1'
    capacity: environment == 'prod' ? 3 : 1
  }
}

resource webApp 'Microsoft.Web/sites@2023-12-01' = {
  name: webAppName
  location: location
  properties: {
    serverFarmId: appServicePlan.id
    httpsOnly: true
    siteConfig: {
      appSettings: [
        {
          name: 'ENVIRONMENT'
          value: environment
        }
      ]
    }
  }
}

// Outputs
output webAppUrl string = 'https://${webApp.properties.defaultHostName}'
output webAppName string = webApp.name
```

### Bicep Modules

```bicep
// modules/storage.bicep
param storageAccountName string
param location string = resourceGroup().location
param sku string = 'Standard_LRS'

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-04-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: sku
  }
  kind: 'StorageV2'
}

output storageAccountId string = storageAccount.id
output blobEndpoint string = storageAccount.properties.primaryEndpoints.blob
```

```bicep
// main.bicep — consume module
module storage 'modules/storage.bicep' = {
  name: 'storageDeployment'
  params: {
    storageAccountName: 'mystore${uniqueString(resourceGroup().id)}'
    sku: 'Standard_GRS'
  }
}

// Access module outputs
output blobUrl string = storage.outputs.blobEndpoint
```

### Bicep in Azure Pipelines

```yaml
- task: AzureCLI@2
  displayName: 'What-If Preview'
  inputs:
    azureSubscription: 'MyServiceConnection'
    scriptType: 'bash'
    inlineScript: |
      az deployment group what-if \
        --resource-group myRG \
        --template-file infrastructure/main.bicep \
        --parameters @infrastructure/environments/$(environment)/parameters.json

- task: AzureResourceManagerTemplateDeployment@3
  displayName: 'Deploy Bicep'
  inputs:
    deploymentScope: 'Resource Group'
    azureResourceManagerConnection: 'MyServiceConnection'
    resourceGroupName: 'myRG'
    location: 'uksouth'
    templateLocation: 'Linked artifact'
    csmFile: 'infrastructure/main.bicep'
    csmParametersFile: 'infrastructure/environments/$(environment)/parameters.json'
    deploymentMode: 'Incremental'
```

---

## ARM Templates

ARM Templates are **JSON-based** and the underlying format Bicep compiles to.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "webAppName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Azure Web App"
      }
    }
  },
  "variables": {
    "appServicePlanName": "[concat('plan-', parameters('webAppName'))]"
  },
  "resources": [
    {
      "type": "Microsoft.Web/serverfarms",
      "apiVersion": "2023-12-01",
      "name": "[variables('appServicePlanName')]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "B1"
      }
    }
  ],
  "outputs": {
    "appUrl": {
      "type": "string",
      "value": "[concat('https://', parameters('webAppName'), '.azurewebsites.net')]"
    }
  }
}
```

> ⭐ **Exam tip:** Bicep is preferred in modern deployments. ARM JSON is verbose but still tested. Know that `az bicep decompile` converts ARM JSON → Bicep.

**ARM Template deployment modes:**
| Mode | Behaviour |
|------|-----------|
| **Incremental** | Add/update resources; **does not delete** resources not in template |
| **Complete** | Add/update AND **delete** resources not in template |

> ⚠️ **Complete mode** can delete resources! Use with caution; Bicep defaults to Incremental.

---

## Desired State Configuration (DSC)

DSC is **PowerShell-based** configuration management for OS-level settings.

### Azure Automation State Configuration

**How it works:**
1. Write DSC configuration (PowerShell)
2. Compile to MOF file
3. Upload to Azure Automation
4. Register VMs/servers as DSC nodes
5. Azure Automation pulls and applies configuration continuously
6. Reports compliance status in Azure portal

```powershell
# Example DSC Configuration
Configuration WebServerConfig {
    Import-DscResource -ModuleName PSDesiredStateConfiguration

    Node 'WebServer01' {
        WindowsFeature IIS {
            Ensure = 'Present'
            Name   = 'Web-Server'
        }

        File WebContent {
            Ensure          = 'Present'
            Type            = 'Directory'
            DestinationPath = 'C:\inetpub\wwwroot\myapp'
            DependsOn       = '[WindowsFeature]IIS'
        }

        Service W3SVC {
            Name      = 'W3SVC'
            State     = 'Running'
            DependsOn = '[WindowsFeature]IIS'
        }
    }
}

# Compile to MOF
WebServerConfig -OutputPath ./MOFOutput
```

**Pipeline — upload and apply DSC:**
```yaml
- task: AzureAutomationRunbook@1
  inputs:
    azureSubscription: 'MyConnection'
    automationAccountName: 'myAutomationAccount'
    resourceGroupName: 'myRG'
    runbookName: 'ApplyDSCConfig'
```

---

### Azure Automanage Machine Configuration (formerly Guest Configuration)

- Built into **Azure Policy**
- Apply DSC-like configurations via Azure Policy assignments
- Supports **Windows** and **Linux** VMs (Azure + Arc-enabled)
- No Azure Automation account needed

```bash
# Assign a built-in guest configuration policy
az policy assignment create \
  --name 'audit-secure-baseline' \
  --policy '/providers/Microsoft.Authorization/policyDefinitions/<policy-id>' \
  --scope '/subscriptions/<sub-id>/resourceGroups/myRG'
```

---

## Azure Deployment Environments

**Azure Deployment Environments** allows developers to **self-serve** on-demand environments using pre-approved IaC templates (catalogs).

**Key components:**
| Component | Description |
|-----------|-------------|
| **Dev Center** | Central hub — defines catalogs, network connections, environment types |
| **Project** | Represents a team/product; maps environment types to subscriptions |
| **Environment Type** | Maps to a subscription/resource group (Dev, Test, Staging) |
| **Catalog** | GitHub or Azure DevOps repo containing environment definitions |
| **Environment Definition** | IaC template (ARM/Bicep) + manifest |

**Catalog structure:**
```
catalog/
├── environments/
│   ├── web-app/
│   │   ├── environment.yaml          ← Manifest
│   │   ├── main.bicep                ← IaC
│   │   └── parameters.json
│   └── data-api/
│       ├── environment.yaml
│       └── main.bicep
```

**environment.yaml manifest:**
```yaml
name: Web Application Environment
version: 1.0.0
summary: Deploys a complete web application stack
description: Includes App Service, SQL Database, and Key Vault
runner: ARM        # ARM or Terraform
templatePath: main.bicep
parameters:
  - id: appName
    name: Application Name
    description: Name for the deployed application
    type: string
    required: true
```

**Developer self-service workflow:**
1. Navigate to Azure Developer Center portal or use VS Code extension
2. Create new environment from approved catalog
3. Environment provisions automatically in the designated subscription
4. Delete environment when done → resources removed automatically

**Pipeline integration:**
```yaml
- task: AzureDeploymentEnvironments@1
  inputs:
    azureSubscription: 'MyConnection'
    action: 'Create'
    devCenterName: 'myDevCenter'
    projectName: 'myProject'
    catalogName: 'myCatalog'
    environmentDefinitionName: 'web-app'
    environmentName: 'test-env-$(Build.BuildId)'
    parameters: '{"appName": "myapp-$(Build.BuildId)"}'
```

---

## 🧠 Key Exam Tips for IaC

| Scenario | Answer |
|----------|--------|
| Azure-native IaC, simpler than ARM JSON | **Bicep** |
| Deploy to multiple cloud providers | **Terraform** |
| OS-level configuration management | **Azure Automation DSC** or Ansible |
| Preview changes before deploying | `az deployment group what-if` |
| ARM template deletes resources not in template | `deploymentMode: Complete` |
| ARM template only adds/updates, no deletes | `deploymentMode: Incremental` |
| Allow dev teams to spin up environments on-demand from approved templates | **Azure Deployment Environments** |
| Apply OS configuration via Azure Policy (no Automation account) | **Azure Automanage Machine Configuration** |
| Scan IaC for security misconfigurations | **Checkov**, **tfsec**, **PSRule for Azure** |
| Convert ARM JSON template to Bicep | `az bicep decompile --file template.json` |

---

[← 03d - Deployments](/az-400-study-notes/03d-deployments/) | [03f - Maintain Pipelines →](/az-400-study-notes/03f-maintain-pipelines/)