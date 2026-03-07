---
layout: default
title: "02 — Source Control Strategy"
nav_order: 3
description: "Design and Implement Source Control Strategy — AZ-400 Exam Domain 2 (10–15% of exam) — branching strategies, repository management, and collaboration practices."
permalink: /02-source-control-strategy/
mermaid: true
---

# Domain 2: Design and Implement a Source Control Strategy

> **Exam Weight: 10–15%**
> 📁 [← Back to Home](/az-400-study-notes/)

---

## 📑 Table of Contents

- [2.1 Branching Strategies](#21-design-and-implement-branching-strategies)
- [2.2 Configure and Manage Repositories](#22-configure-and-manage-repositories)

---

## 2.1 Design and Implement Branching Strategies

### Branch Strategy Comparison

| Strategy | Long-lived Branches | Best For | Complexity |
|----------|-------------------|---------|-----------|
| **Trunk-Based Development** | `main` only | Continuous delivery, experienced teams | Low |
| **Feature Branch** | `main` + short-lived feature branches | Most teams | Low–Medium |
| **GitHub Flow** | `main` + feature branches | Web apps, continuous delivery | Low |
| **Git Flow** | `main`, `develop`, `release`, `hotfix`, `feature` | Versioned software, scheduled releases | High |
| **Release Branch** | `main` + `release/x.x` branches | Products with multiple supported versions | Medium |

---

### Trunk-Based Development

**Core principle:** All developers commit to a **single shared branch** (`main`/`trunk`) multiple times per day.

```
main ──●──●──●──●──●──●──►
       │     │
    short  short
    feature feature
    (< 2d)  (< 2d)
```

**Key practices:**
- **Feature flags** to hide incomplete features in production
- Very **small, frequent commits** — avoid long-running branches
- Strong CI gates — every commit must not break the build
- **Pair programming** or continuous code review

**When to use:**
- High-performing, experienced teams
- Continuous delivery pipelines
- Microservices architecture

---

### Feature Branch Workflow

```
main ─────────────────────────────────────────►
         ↑                         ↑
feature/login ──●──●──●── PR ──► merge
                              (squash or merge commit)
```

**Naming conventions:**
```
feature/AZ-1234-user-authentication
bugfix/AZ-5678-null-reference
hotfix/payment-gateway-timeout
release/2.4.0
```

**Benefits:**
- Isolated development — broken code doesn't affect `main`
- Natural PR review point
- Easy to abandon work (just delete the branch)

---

### Release Branching

Used when maintaining **multiple versions** in production simultaneously.

```
main ──────────────────────────────────────────►
      │                    │
   release/1.0 ──●──●──   release/2.0 ──●──●──
                 ↑                       ↑
              patch fixes             patch fixes
```

**Pattern:**
1. Branch from `main` when feature freeze begins
2. Only **bug fixes** cherry-picked onto the release branch
3. Tag the release branch at each release point
4. New features go to `main` for the next release

---

### Pull Request Workflow

**PR Lifecycle:**
```
1. Create branch → 2. Develop & commit → 3. Open PR → 4. Review →
5. CI checks pass → 6. Approve → 7. Merge → 8. Delete branch
```

**Branch Policies (Azure Repos):**

| Policy | Description |
|--------|-------------|
| **Require minimum reviewers** | e.g., 2 approvals required before merge |
| **Check for linked work items** | Enforce traceability — PR must link to a work item |
| **Check for comment resolution** | All PR comments must be resolved before merge |
| **Limit merge types** | Allow only squash merge, no merge commits |
| **Build validation** | Run specific pipeline as a required check |
| **Status checks** | Require external status checks (SonarQube, etc.) |

**Branch Protections (GitHub):**

| Protection | Description |
|-----------|-------------|
| **Require pull request reviews** | Minimum approvals required |
| **Require status checks** | CI must pass before merge |
| **Require branches to be up to date** | Branch must include latest from base |
| **Require signed commits** | GPG-signed commits only |
| **Include administrators** | Rules apply even to admins |
| **Restrict who can push** | Limit push access to specific users/teams |

> ⭐ **Exam tip:** Azure DevOps calls them **Branch Policies**; GitHub calls them **Branch Protection Rules**. Both achieve the same goals but have different UIs and feature sets.

---

### Branch Merging Restrictions

**Merge strategies:**

| Strategy | Description | Use Case |
|----------|-------------|---------|
| **Merge Commit** | Preserves full history, adds merge commit | Long-lived feature branches |
| **Squash Merge** | All commits combined into one | Cleaner main branch history |
| **Rebase and Fast-Forward** | Linear history, no merge commit | Teams preferring linear git log |
| **Semi-Linear (Rebase + Merge)** | Rebase then creates merge commit | Balance of history and clarity |

**CODEOWNERS file (GitHub/Azure DevOps):**
```
# Syntax: <pattern> <owner(s)>
/docs/          @docs-team
*.ts            @frontend-team @lead-developer
/infrastructure/ @devops-team
/src/auth/      @security-team
```
> When a PR touches a file matching a pattern, the listed owners are automatically requested for review.

---

## 2.2 Configure and Manage Repositories

### Managing Large Files

#### Git Large File Storage (LFS)

Standard Git is not designed for large binary files (videos, datasets, compiled binaries). Git LFS replaces large files with **text pointer files** and stores the actual files on a separate server.

```bash
# Install Git LFS
git lfs install

# Track file types
git lfs track "*.psd"
git lfs track "*.mp4"
git lfs track "*.zip"

# This creates/updates .gitattributes:
# *.psd filter=lfs diff=lfs merge=lfs -text

# Commit and push as normal
git add .gitattributes
git add large-file.psd
git commit -m "Add design assets"
git push origin main
```

**Azure Repos LFS:** Supported natively. Enable in repo settings.  
**GitHub LFS:** Free up to 1 GB storage + 1 GB bandwidth/month. Additional packs available.

#### git-fat

An alternative to Git LFS using an external object store (rsync, S3, etc.). Less commonly used but still referenced in the exam.

```bash
# Configure git-fat (in .gitfat file)
[rsync]
remote = your-server:/fat-store
```

---

### Scaling and Optimizing Git Repositories

#### Scalar

**Scalar** is a tool (built by Microsoft, now part of Git) that dramatically improves performance of large repositories through:

| Feature | Description |
|---------|-------------|
| **Partial clone** | Only download objects you actually access |
| **Sparse checkout** | Only check out the subdirectories you need |
| **Background maintenance** | Automatic `git fetch` and repo maintenance |
| **File system monitor** | Faster `git status` using OS file watching |
| **Commit graph** | Pre-computed graph speeds up `git log` and `git blame` |

```bash
# Clone a repo with Scalar
scalar clone https://github.com/org/large-repo

# Optimize an existing repo
scalar register
```

> Scalar was originally developed for the **Windows OS repository** (one of the largest Git repos in the world at ~300 GB).

#### Cross-Repository Sharing

| Technique | Description |
|-----------|-------------|
| **Git Submodules** | Embed a reference to another repo at a specific commit |
| **Git Subtree** | Copy another repo's history into a subdirectory |
| **NuGet/npm packages** | Share code as packages (preferred for libraries) |
| **Azure Artifacts** | Host packages internally for cross-repo sharing |
| **Sparse checkout** | Check out only certain directories of a monorepo |

**Git Submodule example:**
```bash
# Add a submodule
git submodule add https://github.com/org/shared-lib ./libs/shared

# Clone including submodules
git clone --recurse-submodules https://github.com/org/project

# Update all submodules
git submodule update --remote --merge
```

---

### Permissions in Source Control

#### Azure Repos Permissions

**Permission levels (most → least privileged):**
```
Organization Owner
  └── Project Administrator
        └── Contributor
              └── Reader
                    └── Stakeholder
```

**Repository-level permissions:**

| Permission | Description |
|-----------|-------------|
| **Contribute** | Push to non-protected branches |
| **Create branch** | Create new branches |
| **Delete or disable repository** | Destructive action |
| **Force push** | Overwrite history (dangerous — restrict this!) |
| **Manage permissions** | Change repo ACLs |
| **Read** | Clone and view |

**Branch-level permissions:**
- Override branch policies on a per-user/group basis
- Grant "Exempt from policy enforcement" sparingly

#### GitHub Repository Permissions

| Role | Permissions |
|------|-------------|
| **Read** | View and clone |
| **Triage** | Manage issues/PRs, no write access |
| **Write** | Push to non-protected branches, manage issues |
| **Maintain** | Manage repo (no destructive actions) |
| **Admin** | Full control including settings and deletion |

**GitHub Teams:** Group users and assign repo roles at the team level.

---

### Tags

Tags mark specific points in repository history — typically used for **releases**.

```bash
# Lightweight tag (just a pointer)
git tag v1.0.0

# Annotated tag (recommended — includes metadata)
git tag -a v2.1.0 -m "Release 2.1.0 - adds payment gateway"

# Tag a specific commit
git tag -a v1.9.5 abc1234 -m "Hotfix release"

# Push tags to remote
git push origin v2.1.0       # Push single tag
git push origin --tags       # Push all tags

# List tags
git tag --list "v2.*"

# Delete a tag
git tag -d v1.0.0            # Delete locally
git push origin :v1.0.0      # Delete remotely
```

**Semantic Versioning (SemVer):**
```
v MAJOR . MINOR . PATCH
  ^       ^       ^
  │       │       └── Bug fixes (backwards-compatible)
  │       └────────── New features (backwards-compatible)
  └────────────────── Breaking changes
```

---

### Recovering Data with Git Commands

| Scenario | Command |
|----------|---------|
| Undo last commit, keep changes staged | `git reset --soft HEAD~1` |
| Undo last commit, keep changes unstaged | `git reset --mixed HEAD~1` |
| Undo last commit, **discard** all changes | `git reset --hard HEAD~1` |
| Recover a deleted branch | `git checkout -b <branch> <last-commit-sha>` |
| Undo a specific commit (safe, creates new commit) | `git revert <commit-sha>` |
| Restore a deleted file | `git checkout HEAD -- path/to/file` |
| Find a lost commit | `git reflog` |
| Apply a single commit from another branch | `git cherry-pick <commit-sha>` |

**Using `git reflog` to recover:**
```bash
# Show all recent HEAD movements
git reflog

# Output:
# abc1234 HEAD@{0}: commit: WIP: new feature
# def5678 HEAD@{1}: reset: moving to HEAD~1
# ghi9012 HEAD@{2}: commit: remove old code  ← restore this?

# Restore to a previous state
git reset --hard HEAD@{2}
```

> ⭐ **Exam tip:** `git reflog` is your safety net. It keeps a local log of all changes to HEAD for ~90 days.

---

### Removing Data from Source Control

> ⚠️ **Warning:** These operations **rewrite history** and are destructive. Always coordinate with the team.

#### Remove a file from all history (e.g., accidentally committed secrets)

**Option 1: BFG Repo Cleaner (recommended)**
```bash
# Install BFG (Java tool)
java -jar bfg.jar --delete-files secret.env repo.git
java -jar bfg.jar --replace-text passwords.txt repo.git
git push --force
```

**Option 2: git filter-repo (modern alternative to filter-branch)**
```bash
# Remove a file from all history
git filter-repo --path secret.env --invert-paths

# Remove all files matching a pattern
git filter-repo --path-glob "*.pem" --invert-paths
```

**Option 3: Azure DevOps** — Contact Microsoft Support to permanently remove data from the server-side cache after a force push.

**After removing sensitive data:**
1. Force push the rewritten history
2. **Rotate all exposed secrets immediately** — assume they are compromised
3. Notify all collaborators to re-clone (their local copies still have the data)
4. Clear GitHub/Azure DevOps cached views (may require support ticket)

---

## 🧠 Key Exam Tips for Domain 2

| Scenario | Best Answer |
|----------|------------|
| Large binary files slowing down repo | Git LFS |
| Speed up a monorepo clone | Scalar with partial clone + sparse checkout |
| Require 2 approvals before merge (Azure) | Branch Policy: Minimum number of reviewers |
| Auto-assign reviewers based on file path | CODEOWNERS file |
| Released code in production; need to undo **safely** | `git revert` (creates a new commit — safe for shared branches) |
| Undo local commits before pushing | `git reset` |
| Accidentally committed a password to Git | BFG Repo Cleaner or `git filter-repo`, then rotate the secret |
| Share a library across multiple repos | Azure Artifacts or Git submodules |
| Teams maintaining multiple active versions | Release branching strategy |
| Daily deployments, experienced team | Trunk-Based Development |

---

[← 01 - Processes & Communications](/az-400-study-notes/01-processes-and-communications.md) | [03 - Build & Release Pipelines →](/az-400-study-notes/03-build-release-pipelines.md)
