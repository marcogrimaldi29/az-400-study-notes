---
layout: default
title: "06 — Resources, Tips & Quick Reference"
nav_order: 13
description: "Design and Implement Build and Release Pipelines — AZ-400 Exam Domain 3 (50–55% of exam) — pipeline design, implementation, and management."
permalink: /06-resources/
mermaid: true
---

# Resources, Tips & Quick Reference

> 📁 [← Back to Home](/az-400-study-notes/)

---

## 🎯 Exam Strategy

### Study Order (by weight)

```
1. ████████████████  Domain 3: Build & Release Pipelines   (50-55%)  ← Start here!
2. ████              Domain 1: Processes & Communications  (10-15%)
3. ████              Domain 2: Source Control Strategy     (10-15%)
4. ████              Domain 4: Security & Compliance       (10-15%)
5. ██                Domain 5: Instrumentation Strategy    (5-10%)
```

### Exam Tips

- ⏱️ **Time management**: ~180 minutes for ~50–60 questions → ~3 min/question
- 📋 **Read the question twice**: Many answers hinge on a single word (e.g., "cheapest", "fastest rollback", "without code changes")
- 🎯 **"Most appropriate"**: When multiple answers seem correct, pick the Azure-native solution
- 🔄 **Case studies**: Read the requirements section first, then answer questions
- ❓ **Mark for review**: Don't spend more than 5 minutes on any single question
- 💡 **Elimination**: Remove obviously wrong answers first

### Common Traps

| Trap | Watch Out For |
|------|--------------|
| Classic vs YAML pipelines | YAML is the modern answer unless "Classic" is specified |
| Managed Identity vs Service Principal | When on Azure → Managed Identity; external → Service Principal |
| Blue-green vs Canary | Blue-green = instant rollback; Canary = gradual rollout |
| Git revert vs Git reset | `revert` = safe for shared branches; `reset` = local only |
| Lead time vs Cycle time | Lead = full journey; Cycle = active work only |
| ARM Complete vs Incremental | Complete DELETES resources not in template |

---

## 📋 Master Cheat Sheet

### GitHub Flow (6 steps)
```
1. Branch → 2. Commit → 3. PR → 4. Review → 5. Deploy → 6. Merge
```

### Deployment Strategies Summary
| Strategy | Rollback | Downtime | Traffic Split |
|----------|----------|----------|---------------|
| Blue-Green | Instant swap | None | 0% → 100% |
| Canary | Fast | None | 5% → gradually 100% |
| Rolling | Medium | None | Per-instance |
| Recreate | Redeploy | Yes | 0% → 100% |
| A/B Test | Fast | None | % based on user segment |

### DORA Metrics
| Metric | High Performer |
|--------|---------------|
| Deployment Frequency | Multiple times/day |
| Lead Time for Changes | < 1 hour |
| MTTR | < 1 hour |
| Change Failure Rate | 0–15% |

### Key Azure Services for AZ-400
| Service | Purpose |
|---------|---------|
| Azure Boards | Agile project management |
| Azure Repos | Git source control |
| Azure Pipelines | CI/CD automation |
| Azure Artifacts | Package management |
| Azure Test Plans | Manual and automated testing |
| Azure DevOps Analytics | Reporting and dashboards |
| Azure Key Vault | Secrets, keys, certificates |
| Azure App Configuration | Feature flags, config |
| Azure Monitor | Metrics and alerting |
| Log Analytics | Log aggregation and KQL queries |
| Application Insights | APM for applications |
| Azure Load Testing | Load and stress testing |
| Azure Container Registry | Docker image storage |
| Azure Deployment Environments | Self-serve dev environments |

### Versioning Quick Reference
```
SemVer:  MAJOR.MINOR.PATCH  (1.4.2)
         ^     ^     ^
         │     │     └── Bug fixes
         │     └──────── New features (backwards-compatible)
         └────────────── Breaking changes

CalVer:  YYYY.MM.PATCH  (2024.07.1)
```

### Branch Merging Strategies
| Strategy | History | Use When |
|----------|---------|----------|
| Merge commit | Non-linear | Want full context |
| Squash merge | Clean, linear | Clean main history |
| Rebase | Linear | No merge commits desired |

### Git Emergency Commands
```bash
git reflog                          # Show all HEAD history
git reset --soft HEAD~1             # Undo commit, keep changes staged
git reset --hard HEAD~1             # Undo commit, discard changes (DANGEROUS)
git revert <sha>                    # Safe undo (creates new commit)
git cherry-pick <sha>               # Apply single commit to current branch
git stash / git stash pop           # Temporarily store changes
```

### Key YAML Pipeline Concepts
```yaml
trigger: [main]                    # CI trigger
pr: [main]                         # PR trigger
schedules:                         # Scheduled trigger
  - cron: '0 2 * * 1-5'
  
dependsOn: StageA                  # Stage dependency
condition: succeeded('StageA')     # Conditional execution
concurrency:                       # Concurrency control
  group: production
  cancelInProgress: false

strategy:
  matrix: {...}                    # Matrix build
  rolling:                         # Rolling deployment
    maxParallel: 25%
```

### KQL Quick Reference
```kusto
requests                           -- Application Insights requests
| where timestamp > ago(1h)        -- Time filter
| where success == false           -- Filter failed
| summarize count() by name        -- Group by
| summarize avg(duration) by bin(timestamp, 1h)  -- Time bucket
| order by count_ desc             -- Sort
| take 20                          -- Limit
| render timechart                 -- Visualize
| project name, duration, success  -- Select columns
| extend durationSec = duration/1000  -- Computed column
```

### Managed Identity vs Service Principal
```
On Azure resource calling Azure service?
  └── YES → Managed Identity (no secret to manage!)
       ├── Same resource → System-assigned
       └── Multiple resources → User-assigned

External service / multi-tenant / outside Azure?
  └── NO → Service Principal
       ├── Modern: OIDC / Workload Identity Federation (no secret!)
       └── Legacy: Client Secret or Certificate
```

### GitHub Auth Quick Reference
```
Same repo Actions workflow  → GITHUB_TOKEN (automatic)
Cross-repo / org-wide      → GitHub App (recommended) or Fine-grained PAT
Third-party integration     → GitHub App
User-specific scripts       → Fine-grained PAT (with expiry)
```

### Security Scanning Tools
| Tool | Scans | When |
|------|-------|------|
| CodeQL | SAST — code vulnerabilities | Build / PR |
| Dependabot | SCA — dependency CVEs | On push / scheduled |
| GitHub Secret Scanning | Secrets in code | On push (blocks push) |
| Trivy | Container image CVEs | Build / deploy |
| Checkov | IaC misconfigurations | Build / PR |
| Microsoft Defender for DevOps | Multi-platform unified view | Continuous |

---

## 🔗 Official Documentation Links

### Azure DevOps
| Topic | Link |
|-------|------|
| Azure Pipelines YAML | [docs.microsoft.com/azure/devops/pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/) |
| Azure Boards | [docs.microsoft.com/azure/devops/boards](https://learn.microsoft.com/en-us/azure/devops/boards/) |
| Azure Repos | [docs.microsoft.com/azure/devops/repos](https://learn.microsoft.com/en-us/azure/devops/repos/) |
| Azure Artifacts | [docs.microsoft.com/azure/devops/artifacts](https://learn.microsoft.com/en-us/azure/devops/artifacts/) |

### GitHub
| Topic | Link |
|-------|------|
| GitHub Actions | [docs.github.com/actions](https://docs.github.com/en/actions) |
| GitHub Advanced Security | [docs.github.com/code-security](https://docs.github.com/en/code-security) |
| GitHub Packages | [docs.github.com/packages](https://docs.github.com/en/packages) |
| Dependabot | [docs.github.com/dependabot](https://docs.github.com/en/code-security/dependabot) |

### Azure Services
| Topic | Link |
|-------|------|
| Azure Key Vault | [docs.microsoft.com/azure/key-vault](https://learn.microsoft.com/en-us/azure/key-vault/) |
| Azure Monitor | [docs.microsoft.com/azure/azure-monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/) |
| Application Insights | [docs.microsoft.com/azure/azure-monitor/app](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview) |
| Azure App Configuration | [docs.microsoft.com/azure/azure-app-configuration](https://learn.microsoft.com/en-us/azure/azure-app-configuration/) |
| Bicep | [docs.microsoft.com/azure/azure-resource-manager/bicep](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/) |
| Azure Deployment Environments | [docs.microsoft.com/azure/deployment-environments](https://learn.microsoft.com/en-us/azure/deployment-environments/) |
| Scalar | [github.com/microsoft/scalar](https://github.com/microsoft/scalar) |

### Exam Resources
| Resource | Link |
|----------|------|
| Official Exam Page | [AZ-400 Exam](https://learn.microsoft.com/en-us/credentials/certifications/exams/az-400/) |
| Official Study Guide | [AZ-400 Study Guide](https://learn.microsoft.com/en-gb/credentials/certifications/resources/study-guides/az-400) |
| Free Practice Assessment | [Practice Questions](https://learn.microsoft.com/en-us/credentials/certifications/exams/az-400/practice/assessment?assessment-type=practice&assessmentId=56) |
| Exam Sandbox | [Try the exam UI](https://aka.ms/examdemo) |
| Learning Paths | [Microsoft Learn AZ-400](https://learn.microsoft.com/en-us/credentials/certifications/exams/az-400#two-ways-to-prepare) |
| Tech Community | [Azure DevOps Forum](https://techcommunity.microsoft.com/t5/azure-devops/bd-p/AzureDevOpsForum) |

---

## 📅 Suggested Study Plan (4 Weeks)

### Week 1 — Pipelines Foundations (50% of exam)
- [ ] Read [Domain 3 Overview](./03-build-and-release-pipelines.md)
- [ ] Study [3c: Pipelines (YAML)](./03c-pipelines.md)
- [ ] Study [3d: Deployments](./03d-deployments.md)
- [ ] Hands-on: Create a multi-stage YAML pipeline in Azure DevOps
- [ ] Hands-on: Set up a GitHub Actions workflow with environments and approvals

### Week 2 — Pipelines Advanced + Security
- [ ] Study [3a: Package Management](./03a-package-management.md)
- [ ] Study [3b: Testing Strategy](./03b-testing-strategy.md)
- [ ] Study [3e: Infrastructure as Code](./03e-infrastructure-as-code.md)
- [ ] Study [3f: Maintain Pipelines](./03f-maintain-pipelines.md)
- [ ] Study [Domain 4: Security & Compliance](./04-security-and-compliance.md)
- [ ] Hands-on: Create a Bicep template and deploy via pipeline
- [ ] Hands-on: Enable GHAS and run CodeQL on a repo

### Week 3 — Processes, Source Control, Instrumentation
- [ ] Study [Domain 1: Processes & Communications](./01-processes-and-communications.md)
- [ ] Study [Domain 2: Source Control Strategy](./02-source-control-strategy.md)
- [ ] Study [Domain 5: Instrumentation](./05-instrumentation-strategy.md)
- [ ] Hands-on: Set up Azure Boards with GitHub integration
- [ ] Hands-on: Write KQL queries in Log Analytics

### Week 4 — Review and Practice
- [ ] Take [free practice assessment](https://learn.microsoft.com/en-us/credentials/certifications/exams/az-400/practice/assessment?assessment-type=practice&assessmentId=56) (aim for 75%+)
- [ ] Review weak areas using these notes
- [ ] Review [Cheat Sheet](#master-cheat-sheet) above
- [ ] Take practice assessment again (aim for 85%+)
- [ ] **Book and take the exam!** 🎓

---

## ✅ Pre-Exam Checklist

**Knowledge checks:**
- [ ] Can I write a multi-stage Azure Pipelines YAML from scratch?
- [ ] Can I explain Blue-Green vs Canary vs Rolling deployments?
- [ ] Do I know when to use Managed Identity vs Service Principal?
- [ ] Can I explain the difference between Lead Time and Cycle Time?
- [ ] Do I know at least 5 basic KQL operators?
- [ ] Can I explain trunk-based development vs Git Flow?
- [ ] Do I know what GitHub Advanced Security components are?
- [ ] Can I describe 3 ways to prevent secret leakage in pipelines?

**Practical experience:**
- [ ] Created a CI/CD pipeline in Azure DevOps
- [ ] Used GitHub Actions for automated deployment
- [ ] Set up branch policies and PR workflows
- [ ] Deployed a Bicep template to Azure
- [ ] Queried Application Insights with KQL
- [ ] Configured at least one security scan in a pipeline

---

[← 05 - Instrumentation Strategy](az-400-study-notes/05-instrumentation-strategy/) | [Back to Home →](/az-400-study-notes/)

