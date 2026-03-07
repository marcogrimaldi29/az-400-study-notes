# AZ-400: Designing and Implementing Microsoft DevOps Solutions
### Study Notes Repository

![Azure DevOps](https://img.shields.io/badge/Azure-DevOps-0078D4?style=for-the-badge&logo=azure-devops&logoColor=white)
![Certification](https://img.shields.io/badge/Exam-AZ--400-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow?style=for-the-badge)

> Personal study notes for the **Microsoft AZ-400** certification exam.  
> Based on the [official Microsoft study guide](https://learn.microsoft.com/en-gb/credentials/certifications/resources/study-guides/az-400) (updated July 26, 2024).

---

## 📋 Table of Contents

| # | Domain | Weight |
|---|--------|--------|
| [01](./01-processes-and-communications.md) | Design and Implement Processes and Communications | 10–15% |
| [02](./02-source-control-strategy.md) | Design and Implement a Source Control Strategy | 10–15% |
| [03](./03-build-and-release-pipelines.md) | Design and Implement Build and Release Pipelines | **50–55%** |
| [04](./04-security-and-compliance.md) | Develop a Security and Compliance Plan | 10–15% |
| [05](./05-instrumentation-strategy.md) | Implement an Instrumentation Strategy | 5–10% |
| [📚](./resources.md) | Resources, Tips & Quick Reference | — |

---

## 🎯 Exam at a Glance

| Item | Details |
|------|---------|
| **🪪 Exam Code** | AZ-400 |
| **📄 Full Name** | Designing and Implementing Microsoft DevOps Solutions |
| **🏅 Certification** | [DevOps Engineer Expert](https://learn.microsoft.com/en-us/credentials/certifications/devops-engineer/) |
| **🛡️ Prerequisites** | AZ-104 *or* AZ-204 (required before claiming certification) |
| **🥅 Passing Score** | 700 / 1000 |
| **⏱️ Duration** | ~120 minutes |
| **❔ Question Types** | Multiple choice, case studies, drag-and-drop, build-list, hot area |
| **🈁 Languages** | English, Japanese, Korean, Simplified Chinese, Traditional Chinese |
| **🔁 Renewal** | Annual — free online assessment on Microsoft Learn |

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
├── README.md                              ← You are here
│
├── 01-processes-and-communications.md     ← Flow of work, metrics, collaboration
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
├── 05-instrumentation-strategy.md         ← Azure Monitor, App Insights, KQL
│
└── resources.md                           ← Cheat sheet, study plan, exam tips
```

---

## 🔗 Official Microsoft Resources

| Resource | Link |
|----------|------|
| Exam Page | [AZ-400 Exam](https://learn.microsoft.com/en-us/credentials/certifications/exams/az-400/) |
| Study Guide | [Official Study Guide](https://learn.microsoft.com/en-gb/credentials/certifications/resources/study-guides/az-400) |
| Practice Assessment | [Free Practice Questions](https://learn.microsoft.com/en-us/credentials/certifications/exams/az-400/practice/assessment?assessment-type=practice&assessmentId=56) |
| Azure DevOps Docs | [azure/devops](https://learn.microsoft.com/en-us/azure/devops/?view=azure-devops) |
| DevOps Resource Center | [devops](https://learn.microsoft.com/en-us/devops/) |
| Azure Documentation | [azure](https://learn.microsoft.com/en-us/azure/?product=popular) |
| Exam Sandbox | [Try the exam UI](https://aka.ms/examdemo) |
| Microsoft Q&A | [Ask questions](https://learn.microsoft.com/en-us/answers/products/) |

---

## 📝 How to Use These Notes

1. **Start with the [Resources](./resources.md)** section for exam tips and a quick-reference cheat sheet
2. **Study domains in weight order** — pipelines first (50–55%), then everything else
3. **Use the practice assessment** regularly to gauge your readiness
4. **Each section** includes: key concepts, important distinctions, comparison tables, and gotchas
5. Sections marked with ⭐ contain **commonly tested** topics

---

## 📚 About the Study Notes

These notes are hosted on **GitHub Pages** and published as a searchable website on this URL:

👉 **[📘 AZ-400 Study Notes](https://marcogrimaldi29.com/az-400-study-notes/)**

The site includes full-text search, Mermaid diagram rendering, and mobile-friendly navigation for on-the-go review. 

These notes are designed to be a structured, exam-focused summary of the most important concepts and services baseds on the official [Microsoft AZ-400 Study Guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-400) and its criteria.

Additional resources and study notes maintained by me, such as the **[📘 AZ-305 Study Notes](https://marcogrimaldi29.com/az-305-study-notes/)** and more, are also available for those pursuing the Microsoft and Azure certifications at the following Landing Page:

👉 **[🧑‍🏫 Microsoft Study Notes: Central Hub](https://marcogrimaldi29.com/microsoft-study-notes/)**

---

## ✍️ About the Author

Maintained by **[Marco Grimaldi](https://www.linkedin.com/in/marco-grimaldi29/)** — Cloud Consultant, Language Trainer & Lifelong Learner.

🏠 Find more certification guides, study tips, and tech content at **[🌐 marcogrimaldi29.com](https://marcogrimaldi29.com)**

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

