---
layout: default
title: "03d — Deployments"
nav_order: 8
description: "Design and Implement Deployments — AZ-400 Exam Domain 3 (50–55% of exam) — deployment design, implementation, and management."
permalink: /03d-deployments/
mermaid: true
---

# 3d: Design and Implement Deployments

> 📁 [← Back to Home](/az-400-study-notes/)

[← Back to Domain 3](/az-400-study-notes/03-build-and-release-pipelines/)

---

## Deployment Strategies

### Strategy Comparison Table

| Strategy | Downtime | Risk | Rollback Speed | Complexity | Use Case |
|----------|----------|------|----------------|------------|----------|
| **Recreate** | Yes | High | Slow (redeploy) | Low | Dev/Test only |
| **Rolling** | No | Medium | Medium | Medium | Most apps |
| **Blue-Green** | No | Low | Very fast (swap) | Medium-High | Critical apps |
| **Canary** | No | Low | Fast | High | High-traffic apps |
| **Ring** | No | Very low | Fast | High | Large user bases |
| **A/B Testing** | No | Low | Fast | High | Feature experiments |
| **Feature Flags** | No | Very low | Instant | Medium | Any app |

---

### Blue-Green Deployment

**How it works:**
```
                    ┌─────────────┐
                    │ Load Balancer│
                    └──────┬──────┘
                           │ 100% traffic
                           ▼
Production ──►  [BLUE - v1.0]     [GREEN - v1.1]  ← Deploy here
                 (live)                (staging/idle)

After deployment:
                    ┌─────────────┐
                    │ Load Balancer│
                    └──────┬──────┘
                           │ 100% traffic
                           ▼
                [BLUE - v1.0]     [GREEN - v1.1]
                (idle/standby)    (live) ◄─────────
```

**Azure implementation — Deployment Slots:**
```bash
# Deploy to staging slot (green)
az webapp deployment source config-zip \
  --resource-group myRG \
  --name myApp \
  --slot staging \
  --src ./package.zip

# Warm up the staging slot, run smoke tests
# ...

# Swap staging → production (near-zero downtime)
az webapp deployment slot swap \
  --resource-group myRG \
  --name myApp \
  --slot staging \
  --target-slot production
```

**Azure Pipelines task:**
```yaml
- task: AzureAppServiceManage@0
  inputs:
    azureSubscription: 'MyServiceConnection'
    Action: 'Swap Slots'
    WebAppName: 'myApp'
    ResourceGroupName: 'myRG'
    SourceSlot: 'staging'
    SwapWithProduction: true
```

> ⭐ **Key advantage of Blue-Green:** Rollback is just **swapping back** — instantaneous.

---

### Canary Deployment

**How it works — gradually shift traffic:**
```
v1.0 ──► 100% traffic
              │
         Deploy v1.1 to subset of infrastructure
              │
         5% traffic to v1.1, 95% to v1.0
              │
         Monitor metrics (error rate, latency)
              │
         10% → 25% → 50% → 100% (if healthy)
              │
         Decommission v1.0
```

**Azure Traffic Manager / Azure Front Door for canary:**
```bash
# Create weighted traffic routing
az network traffic-manager endpoint update \
  --name prod-endpoint \
  --profile-name myTrafficManager \
  --resource-group myRG \
  --type azureEndpoints \
  --weight 95    # 95% to v1.0

az network traffic-manager endpoint update \
  --name canary-endpoint \
  --profile-name myTrafficManager \
  --resource-group myRG \
  --type azureEndpoints \
  --weight 5     # 5% to v1.1
```

**Kubernetes canary with Azure:**
```yaml
# Canary deployment manifest
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-canary
spec:
  replicas: 1        # 1 canary pod vs 9 stable = 10% canary traffic
  selector:
    matchLabels:
      app: myapp
      track: canary
  template:
    metadata:
      labels:
        app: myapp
        track: canary
    spec:
      containers:
        - name: myapp
          image: myapp:v1.1
```

---

### Ring Deployment

A structured **set of progressive rollout rings**, typically used for large-scale infrastructure or software:

```
Ring 0: Internal users (developers/dogfood)
Ring 1: Early adopters / preview users (~1%)
Ring 2: Limited external rollout (~10%)
Ring 3: Broader rollout (~50%)
Ring 4: General availability (100%)
```

Each ring has automated **health gates** — only promote if metrics are healthy.

> Used extensively by Microsoft for Windows Updates, Azure services, and Microsoft 365 releases.

---

### Feature Flags (Feature Toggles)

**Types of feature flags:**
| Type | Lifetime | Example |
|------|----------|---------|
| **Release toggle** | Short-term | Hide incomplete feature until release |
| **Experiment toggle** | Medium | A/B test different UX |
| **Ops toggle** | Long-term | Circuit breaker, kill switch |
| **Permission toggle** | Permanent | Premium feature for paid users |

#### Azure App Configuration Feature Manager

```csharp
// Startup / Program.cs
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(connectionString)
           .UseFeatureFlags(featureFlagOptions =>
           {
               featureFlagOptions.CacheExpirationInterval = TimeSpan.FromMinutes(5);
           });
});

builder.Services.AddFeatureManagement();
```

```csharp
// Controller
public class HomeController : Controller
{
    private readonly IFeatureManager _featureManager;

    public HomeController(IFeatureManager featureManager)
    {
        _featureManager = featureManager;
    }

    public async Task<IActionResult> Index()
    {
        if (await _featureManager.IsEnabledAsync("NewDashboard"))
        {
            return View("NewDashboard");
        }
        return View("Dashboard");
    }
}
```

**Feature flag filters (Azure App Configuration):**
| Filter | Description |
|--------|-------------|
| `TargetingFilter` | Enable for specific users/groups/percentage |
| `TimeWindowFilter` | Enable between specific dates/times |
| `PercentageFilter` | Enable for a random X% of requests |

**Pipeline integration — toggle flag with pipeline variable:**
```yaml
- task: AzureAppConfiguration@5
  inputs:
    azureSubscription: 'MyConnection'
    AppConfigurationEndpoint: 'https://myconfig.azconfig.io'
    KeyFilter: 'App:*'
    Label: '$(Build.SourceBranchName)'  # Use branch as label
```

---

### A/B Testing

- Show **version A** to group 1, **version B** to group 2
- Measure business metrics (click-through rate, conversion, session duration)
- Statistical significance determines winner

**Azure implementation:**
- **Azure Front Door** with rules for request routing based on headers/cookies
- **Application Insights** experiments feature
- Third-party tools: LaunchDarkly, Optimizely, Split.io

---

## Minimising Downtime

### VIP Swap (Azure Cloud Services — legacy)

- "Virtual IP Swap" — swaps production and staging environments
- Effectively a blue-green swap at the network level for classic Cloud Services
- Near-instantaneous from user perspective

### Load Balancing

```
Users ──► [Azure Load Balancer / App Gateway / Front Door]
                          │
               ┌──────────┴──────────┐
               ▼                     ▼
          [Instance 1]          [Instance 2]
          (drain & update)       (serving traffic)
```

**Connection draining:** Gracefully remove an instance from rotation — wait for existing connections to complete before terminating.

### Rolling Deployments

```yaml
# Azure Pipelines — rolling strategy
jobs:
  - deployment: RollingDeploy
    strategy:
      rolling:
        maxParallel: 25%         # Deploy to 25% of targets at a time
        preDeploy:
          steps:
            - script: echo "Health check before deploy"
        deploy:
          steps:
            - script: echo "Deploy new version"
        routeTraffic:
          steps:
            - script: echo "Enable traffic to new instances"
        postRouteTraffic:
          steps:
            - script: echo "Monitor for 5 minutes"
        on:
          failure:
            steps:
              - script: echo "Rollback"
          success:
            steps:
              - script: echo "Cleanup old version"
```

### Deployment Slots (Azure App Service)

**Slot settings (sticky settings):**
- Some settings should NOT swap with slots (e.g., connection strings pointing to production DB)
- Mark as "Slot setting" in App Service configuration — they stay with the slot

```bash
# Configure a sticky slot setting
az webapp config appsettings set \
  --name myApp \
  --resource-group myRG \
  --slot-settings "ENVIRONMENT_NAME=production"
```

---

## Hotfix Path Plan

**When you need to fix a critical production bug urgently:**

```
                 main (v2.x development)
                    │
                    │
  release/1.5.0 ───┤
         │          │
         ├── hotfix/payment-fix (branch from release or tag)
         │          │
         │     Fix the bug ──► PR ──► Merge to release/1.5.0
         │                           │
         │                      Tag v1.5.1
         │                           │
         │                    Deploy to production
         │
         └──► Also cherry-pick or merge fix back to main
              (prevent regression in next major release)
```

**Pipeline for hotfixes:**
```yaml
trigger:
  branches:
    include:
      - hotfix/*

# Skip long-running tests; run only critical tests for hotfix
steps:
  - script: dotnet test --filter Category=Smoke
  - task: AzureWebApp@1
    condition: succeeded()
    inputs:
      appName: 'myApp'
      package: '$(Build.ArtifactStagingDirectory)/**/*.zip'
```

---

## Resiliency Strategy for Deployments

| Practice | Description |
|----------|-------------|
| **Health endpoints** | `/health` endpoint checked by load balancer and pipeline |
| **Readiness probes** | Kubernetes/container probe — only route traffic when ready |
| **Circuit breaker** | Stop sending requests to a degraded downstream service |
| **Retry policies** | Exponential backoff for transient failures |
| **Rollback triggers** | Auto-rollback when error rate exceeds threshold |
| **Multi-region deployment** | Deploy to secondary region if primary fails |
| **Availability Zones** | Deploy across AZs for infrastructure-level resiliency |

**Auto-rollback with Azure Monitor gate:**
```
Deploy ──► Monitor Azure Monitor Alerts (15 min window)
                │
          ┌─────┴─────┐
      No alerts    Active alerts
          │               │
      Continue         ROLLBACK (swap slots back / redeploy previous)
```

---

## Application Deployment Methods

### Containers

```yaml
# Azure Pipelines — deploy container to AKS
- task: KubernetesManifest@1
  inputs:
    action: 'deploy'
    kubernetesServiceConnection: 'MyAKSConnection'
    namespace: 'production'
    manifests: |
      k8s/deployment.yaml
      k8s/service.yaml
    containers: |
      myregistry.azurecr.io/myapp:$(Build.BuildId)
```

```yaml
# GitHub Actions — build and push to ACR, deploy to AKS
- name: Build and push image
  uses: azure/docker-login@v2
  with:
    login-server: myregistry.azurecr.io
    username: ${{ secrets.ACR_USERNAME }}
    password: ${{ secrets.ACR_PASSWORD }}

- run: |
    docker build -t myregistry.azurecr.io/myapp:${{ github.sha }} .
    docker push myregistry.azurecr.io/myapp:${{ github.sha }}

- uses: azure/k8s-deploy@v5
  with:
    namespace: production
    manifests: ./k8s/
    images: myregistry.azurecr.io/myapp:${{ github.sha }}
```

### Binaries / Scripts

```yaml
# Deploy via Azure CLI script
- task: AzureCLI@2
  inputs:
    azureSubscription: 'MyConnection'
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      az functionapp deployment source config-zip \
        --resource-group myRG \
        --name myFunctionApp \
        --src $(Build.ArtifactStagingDirectory)/function.zip
```

---

## Database Tasks in Deployments

**Challenges:**
- DB schema changes must be coordinated with app deployments
- Migrations must be backwards-compatible (for blue-green / canary)
- Rollback of DB changes is often complex

**Strategies:**

| Approach | Description |
|----------|-------------|
| **Expand-Contract** | Add new column (expand) → deploy app → remove old column (contract) |
| **Blue-Green with DB** | Separate DB for each slot (expensive but clean) |
| **Migration scripts** | Run as part of deployment pipeline |
| **Feature flags** | App works with both old/new schema until migration complete |

**Azure Pipelines — SQL migrations:**
```yaml
# Using Entity Framework Core migrations
- task: DotNetCoreCLI@2
  displayName: 'Apply EF Core Migrations'
  inputs:
    command: 'custom'
    custom: 'ef'
    arguments: >
      database update
      --project src/MyApp.Data
      --startup-project src/MyApp.Api
      --connection "$(DB_CONNECTION_STRING)"

# Using DACPAC (SQL Server)
- task: SqlAzureDacpacDeployment@1
  inputs:
    azureSubscription: 'MyConnection'
    AuthenticationType: 'servicePrincipal'
    ServerName: 'myserver.database.windows.net'
    DatabaseName: 'mydb'
    deployType: 'DacpacTask'
    DeploymentAction: 'Publish'
    DacpacFile: '$(Build.ArtifactStagingDirectory)/MyApp.dacpac'
    AdditionalArguments: '/p:BlockOnPossibleDataLoss=false'

# Using Flyway (multi-database migration tool)
- script: |
    flyway -url=jdbc:sqlserver://$(DB_SERVER) \
           -user=$(DB_USER) \
           -password=$(DB_PASSWORD) \
           -locations=filesystem:./migrations \
           migrate
```

**Ordering dependency deployments:**
```yaml
stages:
  - stage: DeployDatabase
    jobs:
      - job: MigrateDB
        steps:
          - script: flyway migrate

  - stage: DeployApp
    dependsOn: DeployDatabase      # ← App only deploys after DB is updated
    condition: succeeded('DeployDatabase')
    jobs:
      - deployment: DeployApp
        ...
```

---

## 🧠 Key Exam Tips for Deployments

| Scenario | Answer |
|----------|--------|
| Zero-downtime deployment with fast rollback | **Blue-Green** with deployment slot swap |
| Gradually expose new version to users | **Canary** or **Ring** deployment |
| Hide an incomplete feature in production code | **Feature flag** (Azure App Configuration) |
| A/B test two different checkout flows | **A/B testing** with Azure Front Door routing |
| Apply DB schema changes before app deployment | Separate DB migration stage with `dependsOn` |
| Rollback if error rate spikes post-deployment | Post-deployment monitoring gate + auto-swap back |
| Critical bug in production — fast path | Hotfix branch from release tag → short pipeline → cherry-pick to main |
| Deploy to only 25% of servers at a time | **Rolling** strategy with `maxParallel: 25%` |

---

[← 03c - Pipelines](/az-400-study-notes/03c-pipelines/) | [03e - Infrastructure as Code →](/az-400-study-notes/03e-infrastructure-as-code/)
