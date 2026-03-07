# 3c: Design and Implement Pipelines

[← Back to Domain 3](./03-build-and-release-pipelines.md)

---

## Deployment Automation Solution Selection

### GitHub Actions vs Azure Pipelines — Decision Guide

```
Is your code hosted on GitHub?
├── Yes → GitHub Actions is the natural choice
│          └── Need Azure-specific integrations? → Use Azure login action + Azure CLI
└── No (Azure Repos, Bitbucket, etc.)
    └── Azure Pipelines → Supports GitHub, Bitbucket, Azure Repos, TFVC
```

**Choose GitHub Actions when:**
- Code is in GitHub
- Open-source project (generous free minutes)
- Team is familiar with GitHub ecosystem
- Using GitHub-native features (Dependabot, Advanced Security, Projects)

**Choose Azure Pipelines when:**
- Using Azure Repos or mixed source control
- Need advanced release management (approvals, gates, stages)
- Enterprise compliance requirements
- Connecting to TFVC (Team Foundation Version Control)
- Classic release pipelines with GUI management

---

## Runner / Agent Infrastructure

### Microsoft-Hosted Agents (Azure Pipelines)

| Image | OS | Pre-installed |
|-------|----|--------------|
| `ubuntu-latest` / `ubuntu-24.04` | Ubuntu | Docker, Python, Node, Java, .NET |
| `windows-latest` / `windows-2022` | Windows Server | VS Build Tools, .NET, Python |
| `macOS-latest` / `macOS-14` | macOS | Xcode, .NET, Node, Python |

**Limitations:**
- No persistent storage between runs
- Fixed software versions (updated monthly)
- Limited customization
- Public internet access only (no private network)
- Cost: Included free minutes; additional parallel jobs cost money

### Self-Hosted Agents (Azure Pipelines)

**When to use self-hosted:**
- Access to private networks / on-premises resources
- Specific software requirements not on hosted agents
- Faster builds (persistent caches, local Docker layers)
- Cost optimization for high-volume pipelines
- Compliance requirements (data never leaves your network)

**Agent installation:**
```bash
# Download agent
mkdir myagent && cd myagent
curl -O -L https://vstsagentpackage.azureedge.net/agent/3.x.x/vsts-agent-linux-x64-3.x.x.tar.gz
tar zxvf vsts-agent-linux-x64-3.x.x.tar.gz

# Configure
./config.sh --url https://dev.azure.com/MyOrg \
            --auth PAT \
            --token $(PAT_TOKEN) \
            --pool MyPool \
            --agent MyAgent

# Run as service
sudo ./svc.sh install
sudo ./svc.sh start
```

**Agent as Docker container:**
```dockerfile
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y curl libicu-dev

WORKDIR /agent
RUN curl -O -L https://vstsagentpackage.azureedge.net/agent/3.x.x/vsts-agent-linux-x64.tar.gz \
    && tar zxvf vsts-agent-linux-x64.tar.gz

COPY start.sh .
ENTRYPOINT ["./start.sh"]
```

**Agent capabilities:**
- Define custom capabilities for agent (e.g., `HasDocker=true`, `HasSpecialSoftware=true`)
- Reference in pipeline: `demands: HasDocker`

---

### GitHub-Hosted Runners

| Runner | OS | Free Minutes (private repos) |
|--------|----|------------------------------|
| `ubuntu-latest` | Ubuntu 24.04 | 2,000/month |
| `windows-latest` | Windows 2022 | 3,000/month (2x cost) |
| `macos-latest` | macOS 14 | 10,000/month (10x cost) |

### Self-Hosted GitHub Runners

```yaml
# Register runner:
# Settings → Actions → Runners → New self-hosted runner

# In workflow:
jobs:
  build:
    runs-on: self-hosted        # Use any self-hosted runner
    # OR
    runs-on: [self-hosted, linux, x64, production]  # Specific labels
```

**Runner groups:** Organize runners by team/project, control access at org level.

**Scale with ARC (Actions Runner Controller):**
- Kubernetes-based autoscaling for self-hosted runners
- Runners spin up on demand, terminate after job completes
- Ideal for bursty workloads

---

## GitHub Repositories ↔ Azure Pipelines Integration

**Method 1: Connect GitHub as a source in Azure Pipelines**
1. New Pipeline → Select **GitHub**
2. Authorize with OAuth or GitHub App
3. Select repository → Azure Pipelines will create a `azure-pipelines.yml`

**Method 2: GitHub App (recommended)**
- Install **Azure Pipelines GitHub App** in GitHub organization
- More secure than OAuth (scoped permissions, no user token)
- Status checks appear on PRs natively

**PR validation from Azure Pipelines:**
```yaml
# azure-pipelines.yml
pr:
  branches:
    include:
      - main
      - release/*
  paths:
    exclude:
      - docs/*
      - '**/*.md'
```

---

## Pipeline Trigger Rules

### Azure Pipelines Triggers

**CI Trigger:**
```yaml
trigger:
  branches:
    include:
      - main
      - release/*
    exclude:
      - experimental/*
  paths:
    include:
      - src/*
    exclude:
      - docs/*
  tags:
    include:
      - v*

# Disable CI trigger:
trigger: none
```

**PR Trigger:**
```yaml
pr:
  branches:
    include:
      - main
  autoCancel: true   # Cancel older runs when new commit pushed to PR
```

**Scheduled Trigger:**
```yaml
schedules:
  - cron: '0 2 * * 1-5'    # 2 AM UTC, weekdays
    displayName: 'Nightly build'
    branches:
      include:
        - main
    always: true             # Run even if no code changes
```

**Pipeline Trigger (trigger one pipeline from another):**
```yaml
resources:
  pipelines:
    - pipeline: BuildPipeline
      source: 'MyApp-CI'
      trigger:
        branches:
          include:
            - main
```

### GitHub Actions Triggers

```yaml
on:
  push:
    branches: [main, 'release/**']
    paths-ignore: ['docs/**', '*.md']
    tags: ['v*']

  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened]

  schedule:
    - cron: '0 2 * * 1-5'  # Weekdays at 2 AM UTC

  workflow_dispatch:        # Manual trigger
    inputs:
      environment:
        description: 'Target environment'
        required: true
        default: 'staging'
        type: choice
        options: [staging, production]

  workflow_call:            # Called by another workflow (reusable)
    inputs:
      version:
        required: true
        type: string

  repository_dispatch:      # Triggered via API (webhooks)
    types: [deploy-prod]
```

---

## Developing Pipelines with YAML

### Azure Pipelines YAML Structure

```yaml
# ─── Pipeline-level settings ───────────────────────────────
name: $(Date:yyyyMMdd).$(Rev:r)
trigger: [main]

# ─── Variables ─────────────────────────────────────────────
variables:
  - name: buildConfig
    value: 'Release'
  - group: 'Production-Variables'         # Variable group from Library
  - template: vars/common.yml             # Template file

# ─── Stages ────────────────────────────────────────────────
stages:
  - stage: Build
    displayName: 'Build & Test'
    jobs:
      - job: BuildJob
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - checkout: self
            fetchDepth: 0     # Full history (needed for SonarQube, GitVersion)

          - template: templates/build-steps.yml   # Reusable template
            parameters:
              buildConfig: $(buildConfig)

  - stage: DeployStaging
    dependsOn: Build
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: DeployStaging
        environment: 'staging'
        strategy:
          runOnce:
            deploy:
              steps:
                - script: echo "Deploy to staging"

  - stage: DeployProd
    dependsOn: DeployStaging
    jobs:
      - deployment: DeployProd
        environment: 'production'   # Has approval configured
        strategy:
          runOnce:
            deploy:
              steps:
                - script: echo "Deploy to prod"
```

### GitHub Actions Workflow Structure

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]

env:
  AZURE_WEBAPP_NAME: my-app

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.value }}

    steps:
      - uses: actions/checkout@v4

      - name: Set version
        id: version
        run: echo "value=$(date +%Y%m%d.${{ github.run_number }})" >> $GITHUB_OUTPUT

      - name: Build
        run: dotnet build --configuration Release

      - name: Test
        run: dotnet test --no-build

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: webapp
          path: ./publish/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: production

    steps:
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: webapp

      - name: Deploy to Azure Web App
        uses: azure/webapps-deploy@v3
        with:
          app-name: ${{ env.AZURE_WEBAPP_NAME }}
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
```

---

## Job Execution Order: Parallelism and Multi-Stage

### Parallel Jobs (Azure Pipelines)

```yaml
jobs:
  - job: TestLinux
    pool:
      vmImage: 'ubuntu-latest'
    steps:
      - script: dotnet test --filter OS=Linux

  - job: TestWindows
    pool:
      vmImage: 'windows-latest'
    steps:
      - script: dotnet test --filter OS=Windows

  - job: TestMac
    pool:
      vmImage: 'macOS-latest'
    steps:
      - script: dotnet test --filter OS=Mac

  - job: PublishResults
    dependsOn:
      - TestLinux
      - TestWindows
      - TestMac
    condition: always()
    steps:
      - script: echo "Publish aggregated results"
```

### Fan-out / Fan-in Pattern

```
               ┌──► Job A ──►┐
Trigger ──► Gate             ├──► Aggregate Job
               └──► Job B ──►┘
               └──► Job C ──►┘
```

### Matrix Strategy

```yaml
# Azure Pipelines
strategy:
  matrix:
    Python38:
      pythonVersion: '3.8'
    Python311:
      pythonVersion: '3.11'
    Python312:
      pythonVersion: '3.12'
  maxParallel: 2

steps:
  - task: UsePythonVersion@0
    inputs:
      versionSpec: '$(pythonVersion)'
```

```yaml
# GitHub Actions
strategy:
  matrix:
    python-version: ['3.8', '3.11', '3.12']
    os: [ubuntu-latest, windows-latest]
  fail-fast: false   # Don't cancel other matrix jobs if one fails
  max-parallel: 4
```

---

## Reusable Pipeline Elements

### YAML Templates (Azure Pipelines)

**Define a template (`templates/build-steps.yml`):**
```yaml
parameters:
  - name: buildConfig
    type: string
    default: 'Release'
  - name: runTests
    type: boolean
    default: true

steps:
  - task: DotNetCoreCLI@2
    displayName: 'Restore'
    inputs:
      command: 'restore'

  - task: DotNetCoreCLI@2
    displayName: 'Build'
    inputs:
      command: 'build'
      arguments: '--configuration ${{ parameters.buildConfig }} --no-restore'

  - ${{ if eq(parameters.runTests, true) }}:
    - task: DotNetCoreCLI@2
      displayName: 'Test'
      inputs:
        command: 'test'
        arguments: '--no-build'
```

**Use the template:**
```yaml
steps:
  - template: templates/build-steps.yml
    parameters:
      buildConfig: 'Debug'
      runTests: false
```

### Reusable Workflows (GitHub Actions)

**Define (`/.github/workflows/reusable-build.yml`):**
```yaml
on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
    secrets:
      AZURE_CREDENTIALS:
        required: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: dotnet build
```

**Call from another workflow:**
```yaml
jobs:
  call-build:
    uses: ./.github/workflows/reusable-build.yml
    with:
      environment: production
    secrets:
      AZURE_CREDENTIALS: ${{ secrets.AZURE_CREDENTIALS }}
```

### Variable Groups (Azure Pipelines)

```yaml
variables:
  - group: 'Dev-Environment'      # Library → Variable Groups

# In pipeline, access as:
# $(variableName)
```

**Link Variable Group to Key Vault:**
1. Library → Variable Groups → Link secrets from Azure Key Vault
2. Variables automatically sourced from Key Vault at runtime

### Task Groups (Azure Pipelines Classic)

- Group steps into a reusable task (like a composite template for Classic pipelines)
- Parameterisable, versioned
- Converted to YAML templates when migrating

---

## Checks and Approvals (YAML Environments)

### Configure in Azure DevOps UI

1. **Pipelines → Environments → [environment name] → Approvals and checks**

**Available check types:**
| Check | Description |
|-------|-------------|
| **Approvals** | Named users/groups must approve |
| **Branch control** | Only deploy from specific branches (e.g., `main`) |
| **Business hours** | Only deploy during defined hours |
| **Required template** | Pipeline YAML must extend a specific template |
| **Invoke Azure Function** | Custom validation via Azure Function |
| **Query Azure Monitor** | Validate no active alerts |
| **Invoke REST API** | External approval system integration |

### Approval in Pipeline

```yaml
- stage: Production
  jobs:
    - deployment: DeployProd
      environment: 'production'    # Approvals defined on environment
      strategy:
        runOnce:
          deploy:
            steps:
              - script: echo "Deploying after approval"
```

---

## 🧠 Key Exam Tips for Pipelines

| Scenario | Answer |
|----------|--------|
| Run pipeline on PR to main | `pr:` trigger (Azure Pipelines) or `pull_request:` event (Actions) |
| Share steps across multiple pipelines | YAML templates (Azure) / reusable workflows (GitHub) |
| Store common config for multiple pipelines | Variable Groups in Azure DevOps Library |
| Require manual approval before production deploy | Configure **Approvals** on the Environment |
| Run tests on Windows, Linux, macOS simultaneously | Matrix strategy |
| Deploy only from the `main` branch | Branch control check on Environment |
| Run a nightly build regardless of code changes | Schedule trigger with `always: true` |
| Build fails but you need test results published | `condition: succeededOrFailed()` on the publish step |
| Agent needs to access private subnet | Self-hosted agent in the same VNet |

---

[← Testing Strategy](./03b-testing-strategy.md) | [Next: Deployments →](./03d-deployments.md)
