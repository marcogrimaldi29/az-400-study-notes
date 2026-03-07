# Domain 4: Develop a Security and Compliance Plan

**Exam Weight: 10–15%**

[← Back to Main](./README.md)

---

## 📑 Table of Contents

- [4.1 Authentication and Authorization](#41-design-and-implement-authentication-and-authorization-methods)
- [4.2 Managing Sensitive Information in Automation](#42-managing-sensitive-information-in-automation)
- [4.3 Automate Security and Compliance Scanning](#43-automate-security-and-compliance-scanning)

---

## 4.1 Design and Implement Authentication and Authorization Methods

### Service Principal vs Managed Identity

> ⭐ This is a **very commonly tested** comparison!

| | Service Principal | Managed Identity |
|--|------------------|-----------------|
| **What is it** | App/service identity in Azure AD | Azure-managed identity for Azure resources |
| **Credentials** | Client ID + Secret or Certificate | No credentials to manage |
| **Secret rotation** | Manual (your responsibility) | Automatic (Azure handles it) |
| **Best for** | Cross-tenant, non-Azure resources, CI/CD external to Azure | Azure resources calling other Azure services |
| **Types** | Single type | System-assigned, User-assigned |
| **Supports federated identity** | ✅ (OIDC) | ✅ (system-assigned) |

**System-assigned Managed Identity:**
- Created with the Azure resource (VM, App Service, Function App, etc.)
- Lifecycle tied to the resource — deleted when resource is deleted
- One-to-one: one identity per resource

**User-assigned Managed Identity:**
- Created independently as a standalone Azure resource
- Can be assigned to **multiple** Azure resources
- Lifecycle managed independently — persists when resources are deleted

**When to use User-assigned vs System-assigned:**
| Use Case | Recommendation |
|----------|---------------|
| Multiple resources need same permissions | User-assigned (share one identity) |
| Resource is unique and self-contained | System-assigned |
| Pre-provision identity before resource creation | User-assigned |
| Simplest setup for a single resource | System-assigned |

---

### GitHub Authentication

#### GitHub Apps
- **Recommended** for automation and integrations
- Act as their own identity (not tied to a user)
- Fine-grained permissions at repo/org level
- OAuth 2.0 flow with JWT for auth
- Higher rate limits than PATs

#### GITHUB_TOKEN
- Automatically generated **per-workflow-run** by GitHub Actions
- Scoped to the current repository
- Permissions configurable in workflow YAML:
  ```yaml
  permissions:
    contents: read
    pull-requests: write
    issues: write
  ```
- Expires when workflow completes
- **Best for:** Actions that work within the same repo

#### Personal Access Tokens (PATs)
- **Classic PAT**: Scoped to user account, coarse permissions
- **Fine-grained PAT**: Scoped to specific repos, expiry date required
- **Avoid** where possible — tied to an individual, not a service
- Store as **encrypted secrets**, never in code

**Choosing GitHub auth method:**
```
Need to call GitHub API from Actions workflow?
├── Same repo → Use GITHUB_TOKEN (automatic, no setup)
├── Cross-repo, same org → Fine-grained PAT or GitHub App
└── Third-party integration → GitHub App (preferred)
```

---

### Azure DevOps Authentication

#### Service Connections
- Store credentials for external services (Azure, AWS, Docker, SonarQube, etc.)
- Types:
  | Type | Use Case |
  |------|---------|
  | **Azure Resource Manager** | Deploy to Azure subscriptions |
  | **Docker Registry** | Push images to ACR or Docker Hub |
  | **Kubernetes** | Deploy to AKS or any Kubernetes cluster |
  | **Generic** | REST APIs, custom services |
  | **GitHub** | Access GitHub repositories |
  | **SSH** | SSH-based deployments |

**Creating Azure RM Service Connection:**
- **Recommended**: Use **Workload Identity Federation** (no secret/certificate needed)
- Traditional: Service Principal with Client Secret or Certificate
- Managed Identity: When agent runs in Azure (self-hosted)

**Workload Identity Federation (OIDC) — modern approach:**
```yaml
# No stored credentials! Azure AD trusts the OIDC token from GitHub Actions
- name: Azure Login (OIDC)
  uses: azure/login@v2
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
    # No client secret needed!
```

#### Azure DevOps PATs
- Used to authenticate Azure DevOps REST API calls and CLI
- Scoped to organization-level permissions
- Set expiry dates (maximum 1 year)
- Store in Key Vault or pipeline secrets — never in code

---

### Permissions and Roles

#### GitHub Repository Roles

| Role | Permissions |
|------|-------------|
| **Read** | View and clone repository |
| **Triage** | Manage issues and PRs, no write |
| **Write** | Push to non-protected branches, manage issues |
| **Maintain** | Manage repo without destructive actions |
| **Admin** | Full control including settings, delete repo |

**GitHub Teams:**
- Group users and assign roles at team level to multiple repos
- Nested teams supported (inherit from parent)
- CODEOWNERS file references teams: `@org/team-name`

#### Azure DevOps Permission Groups

| Group | Scope |
|-------|-------|
| **Project Administrators** | Full control within project |
| **Build Administrators** | Manage pipelines |
| **Release Administrators** | Manage releases |
| **Contributors** | Read/write access to most project resources |
| **Readers** | Read-only access |
| **Stakeholders** | Free license — limited to Boards and wiki |

**Azure DevOps — Stakeholder Access:**
- **Free** — for non-technical stakeholders
- Can view/update work items, view pipelines, view dashboards
- Cannot: access repos, run pipelines, access Test Plans

**Outside Collaborators (GitHub):**
- Individuals outside the organization given access to specific repos
- Subject to organizational policies (e.g., require 2FA)
- Best practice: Use teams over individual outside collaborator access

---

## 4.2 Managing Sensitive Information in Automation

### Azure Key Vault

**Key Vault capabilities:**
| Feature | Description |
|---------|-------------|
| **Secrets** | Passwords, connection strings, API keys |
| **Keys** | Cryptographic keys (RSA, EC) for encryption/signing |
| **Certificates** | TLS/SSL certificates with auto-renewal |

**SKUs:**
| | Standard | Premium |
|--|----------|---------|
| Secrets/Certs | ✅ | ✅ |
| Software keys | ✅ | ✅ |
| HSM-backed keys | ❌ | ✅ |

**Access models:**
- **Vault access policy** (legacy): Grant users/apps access to all secrets
- **Azure RBAC** (recommended): Fine-grained role assignments per secret

**Key RBAC roles:**
| Role | Permissions |
|------|-------------|
| Key Vault Administrator | Full management |
| Key Vault Secrets Officer | Manage secrets (not keys/certs) |
| Key Vault Secrets User | Read secrets |
| Key Vault Crypto Officer | Manage keys |
| Key Vault Certificate Officer | Manage certificates |

**Use Key Vault in Azure Pipelines:**
```yaml
variables:
  - group: 'KeyVault-LinkedGroup'   # Variable group linked to Key Vault

# OR using the AzureKeyVault task:
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'MyServiceConnection'
    KeyVaultName: 'my-keyvault'
    SecretsFilter: 'ConnectionString,ApiKey'
    RunAsPreJob: true    # Make secrets available to all steps
```

**Use Key Vault in GitHub Actions:**
```yaml
- name: Get secrets from Key Vault
  uses: azure/get-keyvault-secrets@v1
  with:
    keyvault: my-keyvault
    secrets: 'ConnectionString, ApiKey'
  id: myGetSecretAction

- name: Use the secret
  run: echo "Connection=${{ steps.myGetSecretAction.outputs.ConnectionString }}"
```

---

### Secrets in GitHub Actions

**Types of GitHub secrets:**
| Type | Scope |
|------|-------|
| **Repository secrets** | Available to all workflows in the repo |
| **Environment secrets** | Only available when deploying to a specific environment |
| **Organization secrets** | Shared across multiple repos in the org |

**Secret access in workflows:**
```yaml
env:
  API_KEY: ${{ secrets.API_KEY }}

steps:
  - name: Use secret
    run: |
      curl -H "Authorization: Bearer ${{ secrets.API_KEY }}" \
           https://api.example.com/data
```

**Best practices:**
- Never `echo` secrets — GitHub will redact `***` but logs may be shared
- Use environment secrets for production deployments (extra protection)
- Rotate secrets regularly
- Audit secret access via org audit logs

---

### Secrets in Azure Pipelines

```yaml
variables:
  - name: regularVar
    value: 'hello'
  - name: secretVar
    value: $(MY_SECRET)   # Reference a pipeline secret variable

# Secret variables:
# 1. Set in Pipeline UI → Variables → Secret (lock icon)
# 2. Or in Variable Group → Mark as secret
# 3. Or from linked Key Vault

steps:
  - script: echo "$(regularVar)"   # Prints: hello
  - script: echo "$(secretVar)"    # Prints: *** (masked in logs)

  # Pass secret to script as environment variable
  - script: ./myscript.sh
    env:
      MY_SECRET: $(secretVar)      # ✅ Correct: pass via env
```

---

### Azure Pipelines Secure Files

**Secure Files** store files (certificates, .p12, provisioning profiles) that shouldn't be committed to source control.

```yaml
- task: DownloadSecureFile@1
  name: mySecureFile
  inputs:
    secureFile: 'signing-certificate.p12'

- script: |
    echo "Certificate path: $(mySecureFile.secureFilePath)"
    security import $(mySecureFile.secureFilePath) -k ~/Library/Keychains/login.keychain
```

---

### Preventing Secret Leakage

**Common leakage vectors and mitigations:**

| Risk | Mitigation |
|------|-----------|
| Secret in commit | Pre-commit hooks (git-secrets, detect-secrets) |
| Secret printed in logs | Azure Pipelines auto-masks; GitHub masks registered secrets |
| Secret in PR diff | GitHub Secret Scanning blocks push; Advanced Security alerts |
| Secret in artifact | Scan artifacts before publishing |
| Secret in environment variable dump | Limit env var logging; secure steps |

**Secret scanning tools:**
- **GitHub Advanced Security**: Blocks push if known secret pattern detected
- **Azure DevOps Advanced Security**: Same scanning for Azure Repos
- **Microsoft Defender for DevOps**: Cross-platform scanning
- **Trufflehog**, **gitleaks**: Open-source alternatives

---

## 4.3 Automate Security and Compliance Scanning

### Security Scanning Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│                    Shift-Left Security                           │
│                                                                  │
│  IDE          PR/CI          Build          Deploy       Runtime │
│  ────         ─────          ─────          ──────       ─────── │
│  SAST         SAST           DAST           Image        CSPM    │
│  Secrets      SCA            SCA            Scan         WAF     │
│  Linting      Secret         License        SBOM         SIEM    │
│               Scanning       Check                               │
└─────────────────────────────────────────────────────────────────┘
```

**Scanning types:**
| Type | Acronym | What it finds | When |
|------|---------|--------------|------|
| Static Analysis | SAST | Code vulnerabilities, misconfigurations | Build time |
| Dynamic Analysis | DAST | Runtime vulnerabilities | Against running app |
| Software Composition Analysis | SCA | Vulnerable/unlicensed dependencies | Build time |
| Secret Scanning | — | Committed secrets/credentials | Push time |
| Container Scanning | — | Vulnerable base images, OS packages | Build/deploy |

---

### Microsoft Defender for Cloud — DevOps Security

Provides a **unified security posture** across Azure DevOps, GitHub, and GitLab.

**Capabilities:**
- Connects to GitHub, Azure DevOps, GitLab repositories
- Surfaces security recommendations in Defender for Cloud portal
- Shows **Infrastructure as Code** security findings
- **Pull Request annotations**: Security findings commented on PRs
- Generates **SBOMs** (Software Bill of Materials)

**Setup (Azure DevOps):**
1. Defender for Cloud → **DevOps Security → Add DevOps**
2. Select **Azure DevOps** → Authorize
3. Choose organizations/projects to onboard
4. Results appear in **Defender for Cloud → DevOps Security**

**Key metrics surfaced:**
- Exposed secrets in code
- IaC misconfigurations
- Container image vulnerabilities
- Open-source dependency vulnerabilities
- Code scanning findings (via CodeQL)

---

### GitHub Advanced Security (GHAS)

**Components:**

| Feature | What it does |
|---------|-------------|
| **Code Scanning (CodeQL)** | SAST — finds bugs and security vulnerabilities |
| **Secret Scanning** | Detects secrets committed to code |
| **Dependency Review** | Shows vulnerable dependencies in PRs |
| **Dependabot Alerts** | Notifies about vulnerable dependencies |
| **Dependabot Security Updates** | Auto-PRs to update vulnerable packages |
| **Dependabot Version Updates** | Auto-PRs to keep dependencies current |

**Enable Code Scanning in GitHub Actions:**
```yaml
name: CodeQL Analysis

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 2 * * 1'   # Weekly on Monday

jobs:
  analyze:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      contents: read

    strategy:
      matrix:
        language: [javascript, python, csharp]

    steps:
      - uses: actions/checkout@v4

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
          queries: security-extended   # Extended security query suite

      - name: Autobuild
        uses: github/codeql-action/autobuild@v3

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3
        with:
          category: '/language:${{ matrix.language }}'
```

**Secret Scanning push protection:**
- Enabled per repository or organization
- Blocks a `git push` if it contains a known secret pattern (AWS keys, GitHub tokens, Azure SAS tokens, etc.)
- Developers must confirm the "secret" is a false positive to bypass

**GHAS for Azure DevOps:**
- Available as **Microsoft Security DevOps** extension
- Adds CodeQL scanning, secret scanning, and dependency scanning to Azure Pipelines

```yaml
- task: MicrosoftSecurityDevOps@1
  inputs:
    categories: 'secrets,code,IaC,containers'
```

---

### Integrate GHAS with Microsoft Defender for Cloud

**Integration flow:**
1. Enable GHAS on GitHub repositories
2. Connect GitHub org to Microsoft Defender for Cloud
3. GHAS findings (Code Scanning, Secret Scanning, Dependabot alerts) **flow into** Defender for Cloud
4. Security posture managed in a single pane of glass
5. Azure Security Center recommendations include GitHub findings

---

### Container Scanning

**Scan container images for:**
- OS package vulnerabilities (CVEs)
- Application dependency vulnerabilities
- Misconfigurations in Dockerfile
- Secrets embedded in image layers

**Microsoft Defender for Containers:**
```bash
# Enable on ACR
az security pricing create \
  --name Containers \
  --tier Standard

# Scan on push to ACR (automatic when Defender enabled)
# Results appear in Security Center
```

**Trivy (open-source, popular in GitHub Actions):**
```yaml
- name: Build Docker image
  run: docker build -t myapp:${{ github.sha }} .

- name: Scan image with Trivy
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'myapp:${{ github.sha }}'
    format: 'sarif'
    output: 'trivy-results.sarif'
    severity: 'CRITICAL,HIGH'
    exit-code: '1'      # Fail the workflow if critical vulns found

- name: Upload Trivy scan results to GitHub Security
  uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: 'trivy-results.sarif'
```

**CodeQL analysis in a container:**
```yaml
- name: Initialize CodeQL
  uses: github/codeql-action/init@v3
  with:
    languages: javascript

- name: Build in container
  uses: github/codeql-action/autobuild@v3   # CodeQL traces the container build

- name: Analyze
  uses: github/codeql-action/analyze@v3
```

---

### Dependabot — Open-Source Component Analysis

**Dependabot automates:**
- **Vulnerability alerts**: Notify when a dependency has a known CVE
- **Security updates**: Auto-create PRs to patch vulnerable dependencies
- **Version updates**: Auto-create PRs to keep dependencies up to date

**Configure `.github/dependabot.yml`:**
```yaml
version: 2
updates:
  # npm packages
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
    reviewers:
      - "security-team"
    labels:
      - "dependencies"
    open-pull-requests-limit: 10
    groups:
      dev-dependencies:
        patterns: ["*"]
        dependency-type: "development"

  # NuGet packages
  - package-ecosystem: "nuget"
    directory: "/src"
    schedule:
      interval: "daily"

  # Docker base images
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"

  # GitHub Actions
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

**Dependabot alert severities:**
| CVSS Score | Severity |
|-----------|---------|
| 0.1–3.9 | Low |
| 4.0–6.9 | Medium |
| 7.0–8.9 | High |
| 9.0–10.0 | Critical |

---

## 🧠 Key Exam Tips for Security and Compliance

| Scenario | Answer |
|----------|--------|
| No credentials to manage, Azure resource calling Azure service | **Managed Identity** |
| Multiple resources need same permissions | **User-assigned Managed Identity** |
| GitHub Actions workflow needs to deploy to Azure without a secret | **OIDC / Workload Identity Federation** |
| Store API keys and connection strings securely for pipelines | **Azure Key Vault** → link to Variable Group |
| Block push if secrets accidentally committed | **GitHub Advanced Security — Secret Scanning push protection** |
| SAST scanning integrated into PR reviews on GitHub | **CodeQL (GitHub Advanced Security)** |
| Automatically update vulnerable npm packages | **Dependabot security updates** |
| Scan Docker images for CVEs in CI pipeline | **Trivy** or **Microsoft Defender for Containers** |
| Unified security view across GitHub + Azure DevOps | **Microsoft Defender for Cloud — DevOps Security** |
| Store a code-signing certificate securely in Azure Pipelines | **Azure Pipelines Secure Files** |
| GHAS finding not supported natively | Upload **SARIF file** to `github/codeql-action/upload-sarif` |
| Free access for project stakeholders in Azure DevOps | **Stakeholder access level** |

---

[← Build & Release Pipelines](./03-build-and-release-pipelines.md) | [Back to Main](./README.md) | [Next: Instrumentation →](./05-instrumentation-strategy.md)
