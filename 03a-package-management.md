---
layout: default
title: "03a — Package Management"
nav_order: 5
description: "Design and Implement Package Management Strategy — AZ-400 Exam Domain 3 (50–55% of exam) — package design, implementation, and management."
permalink: /03a-package-management/
mermaid: true
---

# 3a: Design and Implement a Package Management Strategy

> 📁 [← Back to Home](/az-400-study-notes/)

[← Back to Domain 3](/az-400-study-notes/03-build-and-release-pipelines/)

---

## Package Management Tools

### Azure Artifacts

Azure Artifacts is a **universal package registry** built into Azure DevOps. It supports:

| Package Type | File Extension | Ecosystem |
|-------------|--------------|-----------|
| **NuGet** | `.nupkg` | .NET |
| **npm** | `package.json` | Node.js / JavaScript |
| **Maven** | `pom.xml` | Java |
| **Python (PyPI)** | `setup.py` / `pyproject.toml` | Python |
| **Universal Packages** | `.tar.gz` | Any file/binary |
| **Cargo** | `Cargo.toml` | Rust |

**Key concepts:**
- **Feed**: A repository for hosting packages (like a private npm registry or NuGet gallery)
- **View**: A filtered subset of a feed — promotes packages through stages
- **Upstream sources**: Proxy public registries through Azure Artifacts

---

### GitHub Packages Registry

GitHub's built-in package registry — tightly integrated with GitHub repositories and Actions.

| Package Type | Registry URL |
|-------------|-------------|
| **npm** | `npm.pkg.github.com` |
| **Docker** | `ghcr.io` (GitHub Container Registry) |
| **Maven** | `maven.pkg.github.com` |
| **NuGet** | `nuget.pkg.github.com` |
| **RubyGems** | `rubygems.pkg.github.com` |

**Authenticate with GitHub Packages:**
```bash
# npm
npm login --registry=https://npm.pkg.github.com --scope=@your-org

# NuGet
dotnet nuget add source https://nuget.pkg.github.com/your-org/index.json \
  --name github --username your-username --password $GITHUB_TOKEN
```

---

### Azure Artifacts vs GitHub Packages

| Feature | Azure Artifacts | GitHub Packages |
|---------|----------------|----------------|
| **Tied to** | Azure DevOps organization | GitHub repository/organization |
| **Universal packages** | ✅ Yes | ❌ No |
| **Upstream proxying** | ✅ Yes | Limited |
| **Feed views/promotion** | ✅ Yes | ❌ No |
| **Free storage** | 2 GB free | 500 MB free |
| **Best for** | Complex enterprise scenarios | GitHub-centric workflows |

---

## Package Feeds and Views

### Feed Design

**Single feed approach:** All packages in one feed, use views to promote.

**Multiple feeds approach:** Separate feeds per team or product line.

```
Organization-level feed:  org-packages (shared libraries)
Project-level feed:       project-A-packages (project-specific)
```

**Upstream sources** — configure a feed to proxy public registries:

| Public Registry | Protocol |
|----------------|---------|
| nuget.org | NuGet |
| npmjs.com | npm |
| pypi.org | Python |
| Maven Central | Maven |

When a developer requests a package not in the feed, Azure Artifacts:
1. Checks the feed cache
2. Fetches from the upstream source
3. Saves a copy in the feed (for future use and auditing)

---

### Feed Views

Views allow you to **promote packages** through stages without moving them between feeds.

**Default views:**
| View | Symbol | Purpose |
|------|--------|---------|
| `@local` | — | All packages in the feed |
| `@prerelease` | `α` | Packages in testing |
| `@release` | `✓` | Stable, approved packages |

**Promoting a package (Azure CLI):**
```bash
az artifacts universal publish \
  --organization https://dev.azure.com/MyOrg \
  --feed MyFeed \
  --name my-package \
  --version 1.0.0 \
  --path ./dist \
  --description "Release candidate"

# Promote to @release view
az artifacts universal update \
  --organization https://dev.azure.com/MyOrg \
  --feed MyFeed \
  --name my-package \
  --version 1.0.0 \
  --views @release
```

**In a pipeline:**
```yaml
- task: NuGetCommand@2
  inputs:
    command: 'push'
    packagesToPush: '$(Build.ArtifactStagingDirectory)/*.nupkg'
    nuGetFeedType: 'internal'
    publishVstsFeed: 'MyFeed@release'  # Publish to @release view
```

---

## Versioning Strategies

### Semantic Versioning (SemVer)

**Format:** `MAJOR.MINOR.PATCH[-PRERELEASE][+BUILD]`

```
2  .  4  .  1  -  beta.1  +  build.20240115
│     │     │    │           │
│     │     │    │           └── Build metadata (ignored in precedence)
│     │     │    └───────────── Pre-release label
│     │     └────────────────── Patch: Bug fixes
│     └──────────────────────── Minor: New backwards-compatible features
└────────────────────────────── Major: Breaking changes
```

**Pre-release identifiers (precedence order):**
```
1.0.0-alpha < 1.0.0-alpha.1 < 1.0.0-beta < 1.0.0-beta.2 < 1.0.0-rc.1 < 1.0.0
```

**In pipeline YAML (Azure Pipelines):**
```yaml
variables:
  Major: 2
  Minor: 4
  Patch: $[counter(variables['Minor'], 0)]  # Auto-increment patch

name: $(Major).$(Minor).$(Patch)
```

---

### Calendar Versioning (CalVer)

**Format:** Based on the **release date** instead of semantic meaning.

| Format | Example | Used By |
|--------|---------|---------|
| `YYYY.MM.DD` | `2024.07.26` | Ubuntu (sort of) |
| `YYYY.MM` | `2024.07` | Ubuntu LTS (`24.04`) |
| `YYYY.0M.MICRO` | `2024.07.1` | pip, PyPA tools |
| `YY.MM.MICRO` | `24.7.0` | Black (Python formatter) |

**When to use CalVer:**
- Projects with regular, time-based releases
- Infrastructure or tooling where date context matters
- No clear concept of "breaking changes" (configuration, data)

**When to use SemVer:**
- Libraries with public APIs
- Packages consumed by many downstream consumers
- When backwards compatibility is a key concern

> ⭐ **Exam tip:** Libraries → SemVer. Infrastructure/tooling/date-driven releases → CalVer.

---

### Versioning Pipeline Artifacts

**Azure Pipelines — built-in variables:**

```yaml
# Access in pipeline YAML
variables:
  version: '$(Build.BuildId)'                    # Unique build number
  semver: '1.0.$(Build.BuildId)'                 # Custom SemVer
  fullVersion: '$(Build.BuildNumber)'            # Formatted build number

# Custom build number format in pipeline settings:
name: $(Date:yyyyMMdd).$(Rev:r)  # e.g., 20240726.3
```

**Publish pipeline artifact:**
```yaml
- task: PublishPipelineArtifact@1
  inputs:
    targetPath: '$(Build.ArtifactStagingDirectory)'
    artifact: 'drop'
    publishLocation: 'pipeline'
```

**Consume in a later stage:**
```yaml
- task: DownloadPipelineArtifact@2
  inputs:
    artifact: 'drop'
    path: $(Pipeline.Workspace)/drop
```

**GitHub Actions — artifact versioning:**
```yaml
- name: Upload artifact
  uses: actions/upload-artifact@v4
  with:
    name: app-${{ github.run_number }}
    path: ./dist/
    retention-days: 30

- name: Download artifact
  uses: actions/download-artifact@v4
  with:
    name: app-${{ github.run_number }}
```

---

## Dependency Versioning in Code

### npm (Node.js)

```json
{
  "dependencies": {
    "express": "4.18.2",        // Exact version
    "lodash": "~4.17.21",       // Patch updates only (~)
    "axios": "^1.6.0",          // Minor + patch updates (^)
    "react": ">=18.0.0 <19.0.0" // Range
  }
}
```

**Lock files:**
- `package-lock.json` (npm)
- `yarn.lock` (Yarn)
- `pnpm-lock.yaml` (pnpm)

> Always commit lock files. They ensure **reproducible builds**.

### NuGet (.NET)

```xml
<PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
<PackageReference Include="Microsoft.Extensions.Logging" Version="[8.0.0,9.0.0)" />
<!--                                                               ^ Range notation -->
```

**NuGet version ranges:**
| Notation | Meaning |
|----------|---------|
| `1.0.0` | Exact match |
| `[1.0.0]` | Exact match |
| `[1.0,2.0)` | ≥ 1.0 and < 2.0 |
| `(,1.0]` | ≤ 1.0 |
| `*` | Latest (avoid in production!) |

---

## 🧠 Key Exam Tips for Package Management

| Scenario | Answer |
|----------|--------|
| Host private NuGet packages for .NET teams | Azure Artifacts feed |
| Share Docker images in GitHub-centric workflow | GitHub Container Registry (ghcr.io) |
| Proxy public npm registry through internal feed | Add npmjs.com as upstream source in Azure Artifacts |
| Gradually release packages — testing → stable | Use Azure Artifacts **views** (prerelease → release) |
| Library with public API breaking changes | Bump **MAJOR** version (SemVer) |
| Infrastructure tool with monthly releases | CalVer (e.g., `2024.07`) |
| Ensure reproducible builds | Commit lock files; use exact/pinned versions in pipelines |

---

[← 03 - Build & Release Pipelines](/az-400-study-notes/03-build-release-pipelines/) | [03b - Testing Strategy →](/az-400-study-notes/03b-testing-strategy/)