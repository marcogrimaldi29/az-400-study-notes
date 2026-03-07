---
layout: default
title: "03f — Maintain Pipelines"
nav_order: 10
description: "Design and Implement Maintain Pipelines — AZ-400 Exam Domain 3 (50–55% of exam) — pipeline design, implementation, and management."
permalink: /03f-maintain-pipelines/
mermaid: true
---

# 3f: Maintain Pipelines

> 📁 [← Back to Home](/az-400-study-notes/)

[← Back to Domain 3](/az-400-study-notes/03-build-and-release-pipelines/)

---

## Monitor Pipeline Health

### Key Pipeline Health Metrics

| Metric | Description | Target |
|--------|-------------|--------|
| **Build success rate** | % of builds that succeed | > 95% |
| **Pipeline duration** | Total time from trigger to completion | Trending down over time |
| **Test pass rate** | % of tests passing | > 99% |
| **Flaky test rate** | Tests that sometimes pass, sometimes fail | < 1% |
| **MTTR (pipeline)** | Time to fix a broken build | < 30 minutes |
| **Queue time** | Time waiting for an available agent | < 5 minutes |
| **Agent utilization** | % of time agents are busy | < 80% (leave headroom) |

### Monitoring in Azure DevOps

**Pipeline analytics (built-in):**
- **Pipelines → Analytics**: View pass rate, run duration, test results over time
- **Test Plans → Analytics**: Test pass rate trends, top failing tests
- **Dashboards**: Add Pipeline health widgets

**Pipeline dashboard widgets:**
| Widget | Shows |
|--------|-------|
| Build History | Recent build results timeline |
| Test Results Trend | Pass/fail rate over time |
| Deployment Status | Latest deployment status per environment |
| Pipeline Duration | Build time trends |
| Code Coverage | Coverage % over time |

### Identifying Flaky Tests

**Flaky test = test that produces inconsistent results without code changes.**

**Causes of flaky tests:**
- Race conditions / async issues
- Dependency on test execution order
- External service timeouts
- Date/time dependencies
- Random data without fixed seeds

**Azure DevOps flaky test detection:**
- Automatically flagged in Test Results view
- "Flaky" badge appears on identified tests
- Configure in: **Project Settings → Pipelines → Flaky test management**

**Strategies to handle:**
1. **Quarantine**: Remove from required test suite temporarily
2. **Retry**: Auto-retry failed test (mask the symptom — use cautiously)
3. **Fix root cause**: Properly async/await, mock time dependencies
4. **Mark as flaky**: Track separately, don't block CI

```yaml
# Auto-retry flaky tests (Azure Pipelines)
- task: VSTest@3
  inputs:
    rerunFailedTests: true
    rerunMaxAttempts: 3
```

---

## Optimize Pipeline Performance

### Optimize for Cost

| Strategy | Saving |
|----------|--------|
| Use Microsoft-hosted agents for short jobs | No agent maintenance cost |
| Use self-hosted agents for long/frequent jobs | No per-minute cost |
| Run tests in parallel (reduce wall clock time) | Fewer agent-minutes consumed |
| Cache dependencies between runs | Avoid re-downloading packages |
| Skip unchanged components (incremental builds) | Run only what changed |
| Adjust retention (delete old artifacts/runs) | Reduce storage costs |

**Agent cost comparison:**
- Microsoft-hosted: Billed per minute × parallel jobs
- Self-hosted: Infrastructure cost (VM, AKS) + agent management

### Optimize for Time

**Parallelism:**
```yaml
# Fan-out to multiple agents simultaneously
jobs:
  - job: Test1
    steps: [...]
  - job: Test2
    steps: [...]
  - job: Test3
    steps: [...]
  - job: Finalize
    dependsOn: [Test1, Test2, Test3]
```

**Caching:**
```yaml
# Azure Pipelines — cache NuGet packages
- task: Cache@2
  displayName: 'Cache NuGet packages'
  inputs:
    key: 'nuget | "$(Agent.OS)" | **/packages.lock.json,!**/bin/**'
    restoreKeys: |
      nuget | "$(Agent.OS)"
    path: '$(NUGET_PACKAGES)'

# GitHub Actions — cache node_modules
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

**Incremental builds:**
```yaml
# Only trigger pipeline when relevant files change
trigger:
  paths:
    include:
      - src/api/**
    exclude:
      - docs/**
      - '**/*.md'
```

**Checkout depth:**
```yaml
# Shallow clone (faster) — only get recent history
- checkout: self
  fetchDepth: 1        # Only latest commit (not full history)
```

> ⚠️ Avoid `fetchDepth: 1` if you use GitVersion, SonarQube, or git blame — these need full history.

### Optimize for Reliability

- **Retry transient failures**: Network flakiness, service unavailability
- **Timeout settings**: Fail fast rather than hanging indefinitely
- **Health gates**: Don't deploy to prod if monitoring shows issues
- **Idempotent scripts**: Scripts that can safely re-run without side effects

```yaml
# Set job timeout
jobs:
  - job: Build
    timeoutInMinutes: 30    # Fail after 30 minutes
    cancelTimeoutInMinutes: 5

# Step retry
- script: ./deploy.sh
  retryCountOnTaskFailure: 3
```

---

## Pipeline Concurrency

### Azure Pipelines Concurrency

**Organization-level parallel jobs:**
- **Microsoft-hosted**: 1 free parallel job (public projects: 10 free)
- Each additional parallel job: ~$40/month
- Controls how many pipelines run simultaneously

**Concurrency controls in YAML:**
```yaml
# Limit concurrency at pipeline level
# Only 1 instance of this pipeline runs at a time
concurrency:
  group: production-deploy
  cancelInProgress: false    # Wait for current run; don't cancel it

# OR at job/stage level:
jobs:
  - job: Deploy
    pool:
      name: 'MyPool'
      demands:
        - Agent.OS -equals Linux
```

**GitHub Actions concurrency:**
```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true   # Cancel previous run if new one starts
```

> ⭐ **Use case for `cancel-in-progress: true`:** On PRs — cancel old runs when new commit is pushed.  
> ⭐ **Use case for `cancel-in-progress: false`:** On production deployments — let current deployment finish.

---

## Retention Strategy for Pipeline Artifacts and Dependencies

### Azure Pipelines Retention

**Default retention:**
- Build runs: 30 days
- Release deployments: 60 days
- Pipeline artifacts: Tied to build retention

**Configure at organization level:**
- **Organization Settings → Pipelines → Settings → Retention policies**
- Min/Max days for builds, releases

**Configure at pipeline level:**
```yaml
# Override retention per pipeline
variables:
  Build.ArtifactStagingDirectory: $(Pipeline.Workspace)/staging

# Retention lease — prevent a specific build from being deleted
- task: PowerShell@2
  inputs:
    script: |
      az pipelines runs update \
        --id $(Build.BuildId) \
        --org $(System.TeamFoundationCollectionUri) \
        --project $(System.TeamProject) \
        --retention-leases "days=365"
```

**Retention policies — recommendations:**

| Artifact Type | Recommended Retention |
|--------------|----------------------|
| Feature branch builds | 7–14 days |
| Main branch builds | 30–90 days |
| Release builds (tagged) | 1 year or permanent |
| Test results | 30 days |
| Staging deployments | 30 days |
| Production deployments | 1 year |

### Azure Artifacts Retention

**Azure Artifacts** stores packages — manage retention per feed:
- Feed → Settings → **Retention policies**
- Delete packages older than N days
- Keep at least X latest versions

---

## Migrate Classic to YAML Pipelines

### Why Migrate?

| Classic | YAML |
|---------|------|
| GUI-based, no code | Code-defined (stored in repo) |
| Harder to review/audit | Full code review via PRs |
| No versioning | Versioned with source code |
| Limited reusability | Templates, reusable workflows |
| UI-only configuration | Infrastructure as Code approach |

### Migration Steps

1. **Audit existing classic pipeline:**
   - Document all tasks, variables, triggers, environments
   - Identify linked variable groups, service connections

2. **Export as YAML (Azure DevOps UI):**
   - Classic pipeline → **View YAML** (for build pipelines)
   - Note: Classic **release** pipelines cannot be directly exported to YAML

3. **Create YAML file:**
   ```yaml
   # Start with the exported YAML and refine
   trigger:
     branches:
       include: [main]
   
   pool:
     vmImage: 'ubuntu-latest'
   
   # Migrate tasks one-by-one
   steps:
     - task: DotNetCoreCLI@2
       ...
   ```

4. **Migrate environments:**
   - Classic deployment groups → YAML environments
   - Recreate approval gates as environment checks

5. **Migrate variables:**
   - Classic variables → YAML `variables:` section
   - Secret variables → Variable groups linked to Key Vault

6. **Test in parallel:**
   - Run classic and YAML pipelines side by side
   - Compare outputs and behaviour

7. **Switch and decommission:**
   - Update branch policies to use new YAML pipeline
   - Disable/delete classic pipeline

### Common Migration Challenges

| Challenge | Solution |
|-----------|----------|
| Release gates | YAML environments with approvals and checks |
| Manual intervention tasks | Use environment approval checks |
| Artifact links between classic build + release | Switch to pipeline artifacts in YAML |
| GUI-configured task parameters | Export YAML, capture all inputs |
| Classic agent queues | Map to YAML pool configurations |

---

## 🧠 Key Exam Tips for Maintaining Pipelines

| Scenario | Answer |
|----------|--------|
| Build takes 45 minutes — speed it up | Cache dependencies, parallelize test jobs, shallow clone |
| Too many parallel pipelines consuming agent pool | Configure concurrency group limits |
| Test randomly fails but no code changed | Flaky test — quarantine and fix root cause |
| Keep production release artifacts forever | Set retention lease or increase retention to permanent |
| Prevent two deploys to production at the same time | `concurrency: group: production-deploy, cancelInProgress: false` |
| Cancel previous PR build when new commit pushed | `concurrency: cancelInProgress: true` |
| Migrate classic release pipeline to YAML | Recreate as YAML stages with environment approvals/checks |
| Delete old pipeline artifacts automatically | Configure retention policies at org or pipeline level |

---

[← 03e - Infrastructure as Code](/az-400-study-notes/03e-infrastructure-as-code/) | [04 - Security & Compliance →](/az-400-study-notes/04-security-and-compliance/)