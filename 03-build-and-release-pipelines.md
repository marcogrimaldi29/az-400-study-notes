---
layout: default
title: "03 — Build and Release Pipelines"
nav_order: 4
description: "Design and Implement Build and Release Pipelines — AZ-400 Exam Domain 3 (50–55% of exam) — pipeline design, implementation, and management."
permalink: /03-build-release-pipelines/
mermaid: true
---

# Domain 3: Design and Implement Build and Release Pipelines

> **Exam Weight: 50–55% ⚠️ HIGHEST PRIORITY DOMAIN**
> 📁 [← Back to Home](/az-400-study-notes/)

---

## 📑 Sub-Sections

| # | Topic | Key Technologies |
|---|-------|-----------------|
| [3a](/az-400-study-notes/03a-package-management/) | Package Management Strategy | Azure Artifacts, GitHub Packages, SemVer, CalVer |
| [3b](/az-400-study-notes/03b-testing-strategy/) | Testing Strategy for Pipelines | Unit, Integration, Load tests, Code Coverage, Quality Gates |
| [3c](/az-400-study-notes/03c-pipelines/) | Design and Implement Pipelines | YAML pipelines, GitHub Actions, Agents, Runners, Triggers |
| [3d](/az-400-study-notes/03d-deployments/) | Design and Implement Deployments | Blue-green, Canary, Feature Flags, Database tasks |
| [3e](/az-400-study-notes/03e-infrastructure-as-code/) | Infrastructure as Code (IaC) | Bicep, ARM, DSC, Terraform, Azure Deployment Environments |
| [3f](/az-400-study-notes/03f-maintain-pipelines/) | Maintain Pipelines | Health, Optimization, Retention, Classic → YAML migration |

---

## 🗺️ Pipeline Architecture Overview

```
Source Code (GitHub / Azure Repos)
         │
         ▼
   ┌─────────────────────────────┐
   │     CI Pipeline (Build)      │
   │  ┌────────────────────────┐  │
   │  │  Restore dependencies  │  │
   │  │  Compile / Build       │  │
   │  │  Unit Tests            │  │
   │  │  Code Coverage         │  │
   │  │  Security Scanning     │  │
   │  │  Publish Artifacts     │  │
   │  └────────────────────────┘  │
   └─────────────┬───────────────┘
                 │ Artifact
                 ▼
   ┌─────────────────────────────┐
   │    CD Pipeline (Release)     │
   │                              │
   │  Dev ──► Test ──► Staging    │
   │               ↑              │
   │          Approval Gate       │
   │               ↓              │
   │          Production          │
   └─────────────────────────────┘
```

---

## GitHub Actions vs Azure Pipelines

> ⭐ The exam tests your ability to choose between and integrate these two systems.

| Feature | GitHub Actions | Azure Pipelines |
|---------|---------------|-----------------|
| **Config format** | YAML (`.github/workflows/`) | YAML or Classic (GUI) |
| **Runner/Agent** | GitHub-hosted runners, self-hosted runners | Microsoft-hosted agents, self-hosted agents |
| **Marketplace** | GitHub Marketplace (actions) | Azure DevOps Marketplace (tasks/extensions) |
| **Triggers** | GitHub events (push, PR, schedule, workflow_dispatch) | Pipeline triggers, branch filters, path filters |
| **Environments** | Environments with protection rules | Environments with approvals and checks |
| **Secrets** | Repository/Organization/Environment secrets | Pipeline variables, Variable groups, Key Vault |
| **Reusability** | Reusable workflows, composite actions | Templates, Task groups, Variable groups |
| **Container support** | Container jobs, service containers | Container jobs, service containers |
| **Cost model** | Free minutes for public repos; minutes for private | Parallel jobs, free tier available |

**Integration between the two:**
- Use **Azure Pipelines trigger from GitHub** — connect GitHub repo to Azure Pipelines
- Use **GitHub Actions to deploy to Azure** — use Azure CLI action, Azure login action
- Share artifacts between systems via **Azure Artifacts** or **GitHub Packages**

---

## Key YAML Pipeline Concepts

```yaml
# azure-pipelines.yml — basic structure
trigger:
  branches:
    include:
      - main
      - release/*

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'
  artifactName: 'drop'

stages:
  - stage: Build
    jobs:
      - job: BuildJob
        steps:
          - task: UseDotNet@2
            inputs:
              version: '8.x'

          - script: dotnet build --configuration $(buildConfiguration)
            displayName: 'Build'

          - task: PublishBuildArtifacts@1
            inputs:
              artifactName: $(artifactName)

  - stage: Deploy
    dependsOn: Build
    condition: succeeded()
    jobs:
      - deployment: DeployProd
        environment: 'production'
        strategy:
          runOnce:
            deploy:
              steps:
                - script: echo "Deploying..."
```

---

[← 02 - Source Control Strategy](/az-400-study-notes/02-source-control-strategy/) | [03 - Build & Release Pipelines →](/az-400-study-notes/03-build-release-pipelines.md)

