---
layout: home
title: AZ-400 Study Notes
nav_order: 1
description: "AZ-400 Microsoft DevOps Engineer Expert — complete study notes covering all five exam domains with Mermaid diagrams, comparison tables, and exam caveats. AZ-400 retires on February 1, 2027."
permalink: /
mermaid: true
---

# 📘 AZ-400 Study Notes
{: .no_toc }

**Microsoft Azure DevOps Engineer Expert**
{: .fs-5 .fw-300 }

[Start Studying →](/az-400-study-notes/01-processes-and-communications/){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[View on GitHub](https://github.com/marcogrimaldi29/az-400-study-notes){: .btn .fs-5 .mb-4 .mb-md-0 target="_blank" }

---

> These notes are maintained by **[Marco Grimaldi](https://www.linkedin.com/in/marco-grimaldi29/)** and based on the **[official Microsoft documentation](https://learn.microsoft.com/en-us/credentials/certifications/azure-devops-engineer-expert/)**.
> Find more certification guides, study tips, and tech content at **[🏠 marcogrimaldi29.com](https://marcogrimaldi29.com)**.
> *Not affiliated with or endorsed by Microsoft. Always verify against the latest Microsoft documentation.*

---

## 🗓️ Retirement Notice: AZ-400 retires February 1, 2027 {#retirement-notice}

{: .retirement }
Microsoft is retiring the **Microsoft Certified: DevOps Engineer Expert** certification, **Exam AZ-400**, and the certification's **renewal assessment** on **February 1, 2027**. After that date you can't take the exam, earn the certification, or renew it.

The retirement warning appears on the official [AZ-400 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-400), [exam page](https://learn.microsoft.com/en-us/credentials/certifications/exams/az-400/), and [certification page](https://learn.microsoft.com/en-us/credentials/certifications/devops-engineer/).

| Item | Details |
|------|---------|
| **📅 Retirement date** | February 1, 2027 |
| **📦 What retires** | Microsoft Certified: DevOps Engineer Expert, Exam AZ-400, and the renewal assessment |
| **🚫 After that date** | No new attempts at AZ-400, no new certifications, and no renewals |
| **🪪 Already certified?** | Your certification stays in the *Active Certifications* section of your Microsoft Learn transcript until it expires, then moves to your certification history. Renewal opens six months before your expiry date, so renew before February 1, 2027 if your window is already open. |

### Is AZ-400 being replaced?

In the [same announcement](https://techcommunity.microsoft.com/blog/skills-hub-blog/new-microsoft-certified-agentic-devops-engineer-expert/4528356) (October 15, 2026), Microsoft introduced a new expert-level DevOps certification:

| | Retiring | New |
|---|----------|-----|
| **Certification** | Microsoft Certified: DevOps Engineer Expert | [Microsoft Certified: Agentic DevOps Engineer Expert](https://learn.microsoft.com/en-us/credentials/certifications/agentic-devops-engineer-expert/) (beta) |
| **Exam** | AZ-400: Designing and Implementing Microsoft DevOps Solutions | [AZ-401: Designing and Implementing Microsoft Agentic DevOps Solutions](https://learn.microsoft.com/en-us/credentials/certifications/exams/az-401/) (beta) |
| **Prerequisite** | Azure Administrator Associate *or* Azure Developer Associate | Pass AZ-104 (Azure Administrator) *or* GH-300 (GitHub Copilot) |
| **Status** | Retires February 1, 2027 | In beta; Microsoft says general availability comes in December |

{: .note }
Microsoft doesn't call AZ-401 a "replacement" and hasn't announced an upgrade or transition path for current DevOps Engineer Expert holders. It is, however, the expert DevOps certification that the AZ-400 retirement notice links to. It keeps the same five domains, rebalances them, and adds AI-assisted and agentic DevOps.

**Domain weights compared**

| Domain | AZ-400 | AZ-401 (beta) |
|--------|--------|---------------|
| Design and implement processes and communications | 10–15% | 15–20% |
| Design and implement a source control strategy | 10–15% | 10–15% |
| Design and implement build and release pipelines | **50–55%** | **35–40%** |
| Design and implement a security and compliance plan | 10–15% | 15–20% |
| Implement an instrumentation strategy | 5–10% | 15–20% |

**New in AZ-401 (not covered by these notes)**

- AI-assisted DevOps with **GitHub Copilot**: YAML pipelines, tests, code coverage, quality gates, and project documentation
- **MCP servers**, including the Azure DevOps MCP server and GitHub MCP server, and Azure Boards ↔ GitHub integration through MCP
- Requirements and restrictions for **agentic development** (agents, skills, and loops), plus branch policies and protection rules for GitHub Copilot
- **GitHub Agentic Workflows**
- **Azure SRE Agent** and **Azure Copilot Observability Agent**, GitHub Copilot agent logs, and natural-language log queries
- **Git worktree**, **Git hooks**, and repository migration from Azure DevOps to GitHub

**No longer listed in the AZ-401 skills outline**

- Git LFS, git-fat, and Scalar
- Migrating classic pipelines to YAML, and optimizing pipeline concurrency
- Tool-specific IaC objectives: Azure Automation State Configuration, ARM, Bicep, Azure Machine Configuration, and Azure Deployment Environments (the AZ-401 audience profile still expects Bicep experience)
- Feature flags with Azure App Configuration Feature Manager
- VM, container, storage, and network insights

### What should you do?

- **Preparing for AZ-400 now?** Book and pass it before February 1, 2027, and leave time for a possible retake. Pearson VUE lets you schedule exams up to 90 days ahead.
- **Already certified?** Renew while you still can. After February 1, 2027 the certification stays on your transcript until it expires, but it can't be renewed.
- **Starting from scratch?** Consider AZ-401 instead. These notes still cover much of its foundation (pipelines, source control, security, and monitoring) but not its AI and agent topics. Use the [AZ-401 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-401) for the full skills list.

**Sources:** [AZ-400 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-400) · [AZ-400 exam page](https://learn.microsoft.com/en-us/credentials/certifications/exams/az-400/) · [Microsoft Tech Community announcement](https://techcommunity.microsoft.com/blog/skills-hub-blog/new-microsoft-certified-agentic-devops-engineer-expert/4528356) · [AZ-401 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-401) · [Exam retirement](https://learn.microsoft.com/en-us/credentials/support/retired-certification-exams) · [Credential expiration policies](https://learn.microsoft.com/en-us/credentials/support/credential-expiration-policy) · [Renewal FAQ](https://learn.microsoft.com/en-us/credentials/certifications/renew-your-microsoft-certification-faq)

---

## 📋 Table of Contents

| # | Domain | Weight |
|---|--------|--------|
| [01](/az-400-study-notes/01-processes-and-communications/) | Design and Implement Processes and Communications | 10–15% |
| [02](/az-400-study-notes/02-source-control-strategy/) | Design and Implement a Source Control Strategy | 10–15% |
| [03](/az-400-study-notes/03-build-and-release-pipelines/) | Design and Implement Build and Release Pipelines | **50–55%** |
| [04](/az-400-study-notes/04-security-and-compliance/) | Develop a Security and Compliance Plan | 10–15% |
| [05](/az-400-study-notes/05-instrumentation-strategy/) | Implement an Instrumentation Strategy | 5–10% |
| [06](/az-400-study-notes/06-resources/) | Resources, Tips & Quick Reference | — |

---

## 🎯 Exam at a Glance

| Item | Details |
|------|---------|
| **🪪 Exam Code** | AZ-400 |
| **📄 Full Name** | Designing and Implementing Microsoft DevOps Solutions |
| **🏅 Certification** | [DevOps Engineer Expert](https://learn.microsoft.com/en-us/credentials/certifications/devops-engineer/) |
| **🗓️ Retirement** | **February 1, 2027** — see the [retirement notice](#retirement-notice) |
| **🛡️ Prerequisites** | Azure Administrator Associate (AZ-104) *or* Azure Developer Associate (exam AZ-204 retired July 31, 2026), required before claiming the certification. Check the [certification page](https://learn.microsoft.com/en-us/credentials/certifications/devops-engineer/) for the current list |
| **🥅 Passing Score** | 700 / 1000 |
| **⏱️ Duration** | 100 minutes of exam time (120-minute seat time) |
| **❔ Question Types** | Multiple choice, case studies, drag-and-drop, build-list, hot area |
| **🈁 Languages** | English, Japanese, Chinese (Simplified), Korean, German, French, Spanish, Portuguese (Brazil), Chinese (Traditional), Italian |
| **🔁 Renewal** | Annual — free online assessment on Microsoft Learn, available until February 1, 2027 |

---

## 🏗️ Audience Profile

As a **DevOps Engineer**, you are a developer or infrastructure administrator with expertise in working with people, processes, and products to enable continuous delivery of value. You work on cross-functional teams that include:

- 👩‍💻 Developers
- 🛡️ Site Reliability Engineers
- ☁️ Azure Administrators
- 🔐 Security Engineers

**Core responsibilities:**
- Delivering Microsoft DevOps solutions providing continuous security, integration, testing, delivery, deployment, monitoring, and feedback
- Designing and implementing flow of work, collaboration, communication, source control, and automation
- Experience implementing **both** GitHub and Azure DevOps solutions

---

## 📊 Skills at a Glance

```
┌─────────────────────────────────────────────────────────────┐
│                     AZ-400 Exam Domains                     │
├─────────────────────────────────┬───────────────────────────┤
│ Build and Release Pipelines     │ ████████████████  50–55%  │
│ Processes and Communications    │ ████              10–15%  │
│ Source Control Strategy         │ ████              10–15%  │
│ Security and Compliance Plan    │ ████              10–15%  │
│ Instrumentation Strategy        │ ██                5–10%   │
└─────────────────────────────────┴───────────────────────────┘
```

> ⚠️ **Note:** Build and Release Pipelines makes up over half the exam — prioritise this domain!

---

## 📁 Repository Structure

```
az-400-study-notes/
│
├── README.md / index.md                   ← You are here
│
├── 01-processes-and-communications.md     ← Flow of work, metrics, collaboration
│
├── 02-source-control-strategy.md          ← Branching, repo management, Git
│
├── 03-build-and-release-pipelines.md      ← Overview & index (50–55% of exam)
├── 03a-package-management.md              ← Artifacts, feeds, SemVer, CalVer
├── 03b-testing-strategy.md                ← Gates, test types, code coverage
├── 03c-pipelines.md                       ← YAML, agents, triggers, runners
├── 03d-deployments.md                     ← Strategies, feature flags, DB tasks
├── 03e-infrastructure-as-code.md          ← Bicep, ARM, DSC, Deployment Environments
├── 03f-maintain-pipelines.md              ← Health, optimization, retention, migration
│
├── 04-security-and-compliance.md          ← AuthN/Z, Key Vault, GHAS, scanning
│
├── 05-instrumentation-strategy.md         ← Azure Monitor, App Insights, KQL
│
└── 06-resources.md                        ← Cheat sheet, study plan, exam tips
```

---

## 🔗 Official Microsoft Resources

| Resource | Link |
|----------|------|
| Exam Page | [AZ-400 Exam](https://learn.microsoft.com/en-us/credentials/certifications/exams/az-400/) |
| Study Guide | [Official Study Guide](https://learn.microsoft.com/en-gb/credentials/certifications/resources/study-guides/az-400) |
| Practice Assessment | [New practice assessment on AI Skills Navigator](https://aka.ms/AZ400-PA) (sign-in required) · [Legacy practice assessment on Learn](https://aka.ms/AZ400-LearnPA) (no longer updated) |
| Retirement Announcement | [New Microsoft Certified: Agentic DevOps Engineer Expert](https://techcommunity.microsoft.com/blog/skills-hub-blog/new-microsoft-certified-agentic-devops-engineer-expert/4528356) |
| Successor Exam (AZ-401) | [AZ-401 Exam](https://learn.microsoft.com/en-us/credentials/certifications/exams/az-401/) · [AZ-401 Study Guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-401) |
| Azure DevOps Docs | [azure/devops](https://learn.microsoft.com/en-us/azure/devops/?view=azure-devops) |
| DevOps Resource Center | [devops](https://learn.microsoft.com/en-us/devops/) |
| Azure Documentation | [azure](https://learn.microsoft.com/en-us/azure/?product=popular) |
| Exam Sandbox | [Try the exam UI](https://aka.ms/examdemo) |
| Microsoft Q&A | [Ask questions](https://learn.microsoft.com/en-us/answers/products/) |

---

## 📚 About the Study Notes

The site includes full-text search, Mermaid diagram rendering, and mobile-friendly navigation for on-the-go review. 

These notes are designed to be a structured, exam-focused summary of the most important concepts and services based on the official **[Microsoft AZ-400 Study Guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-400)** and its criteria.

Additional study notes maintained by me are also available for those pursuing Microsoft and Azure certifications at the following Landing Page:

👉 **[🛬 Landing Page: Study Notes](https://marcogrimaldi29.com/study-notes/)**

---

## ✍️ About the Author

Maintained by **[Marco Grimaldi](https://www.linkedin.com/in/marco-grimaldi29/)** — Cloud Consultant, Language Trainer & Lifelong Learner.

Find more certification guides, study tips, and tech content at **[🏠 marcogrimaldi29.com](https://marcogrimaldi29.com)**

The site is continuously updated and based on my personal study notes and experiences. If you have any feedback, suggestions, or corrections, feel free to [reach out](https://marcogrimaldi29.com/contact/)!

---

## 📈 Analytics

This site uses [Umami](https://umami.is/) for privacy-friendly analytics.

---

## ©️ Credits & Acknowledgements

The [Just the Docs](https://github.com/just-the-docs/just-the-docs) theme is used for a clean, documentation-style layout. Licensed under [MIT](https://opensource.org/license/MIT).

Created with the help of AI. Model used: [Claude Sonnet 4.6](https://www.anthropic.com/news/claude-sonnet-4-6). The content has been reviewed and edited by the author for accuracy and clarity, but may contain errors. Always verify against the latest [Microsoft documentation](https://learn.microsoft.com/en-us/azure/).

---

## ⚖️ Disclaimer

These are personal study notes intended to supplement — not replace — official Microsoft learning materials. Content is based on the official AZ-400 study guide and Microsoft documentation. Always verify against the [latest official exam guide](https://learn.microsoft.com/en-gb/credentials/certifications/resources/study-guides/az-400) as exam content may be updated.

> *Not affiliated with or endorsed by Microsoft.*

---
