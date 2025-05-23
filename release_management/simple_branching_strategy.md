# 🌿 Branching Strategy

This document outlines the CodeCommit workflow and branching strategy for our development and release process.

---

## 📂 Branch Types

| Branch Type      | Purpose                                                                 |
|------------------|-------------------------------------------------------------------------|
| `master`         | Main branch containing production-ready, reviewed, and tested code.     |
| `feature/*`      | Developer-specific branches for new features or tasks (e.g., Jira IDs). |
| `bugfix/*`       | Branches to fix bugs from the `master`.                                 |
| `release/*`      | Cut from `master` for QA and UAT. Used for pre-release testing.         |

---

## 🔁 Flow Overview

```plaintext
                           +----------------+
                           |     master     |
                           +----------------+
                            ^      ^     ^
                            |      |     |
     PR & Merge             |      |     |    Cut branch
     from features          |      |     +--------------------+
                            |      |                          |
        +-------------------+      |                          |
        |                          |                          v
+----------------+          +--------------+           +----------------+
| feature/* or   |          | bugfix/*     |           |  release/*     |
| task-123       |          | fix-456      |           +----------------+
+----------------+          +--------------+                    |
                                                               QA/UAT
                                                                |
                                                                v
                                                           (Deployment)
```
## 🔧 Workflow Summary

1. Feature/Bugfix Development
    - Create feature/* or bugfix/* from master.
    - Work on your task.
    - Push to remote and raise a Pull Request (PR).
    - Once reviewed and approved, merge into master.

2. Release Preparation
    - Create a release/* branch from master.
    - Build the release and pass it to QA.
    - Same branch is used for UAT.
    - After approval, deploy to production.

3. Post-Deployment
    - Hotfixes (if needed) should also be branched from master as bugfix/* and merged back.
    - Hotfixes should also be merged in release.
    - ReBuild the release and pass it to QA for hotfix testing.
    - Move new build to UAT.
    - After approval, deploy to production.

✅ Notes
- Always pull the latest master before starting any new branch.
- Do not commit directly to master.
- PRs should be peer-reviewed before merging.
