# 🔄 Atlas GitHub Pipeline Workflows

## 📋 Overview

Generalized CI/CD workflows for use across multiple projects in the BLW ecosystem.

---

## 🌿 GitFlow Branching Strategy

The main workflows (`backend_workflow.yml` and `frontend_workflow.yml`) are designed with the GitFlow branching strategy in mind.

Semantic versioning is used to automatically update versions based on commit messages and branch names. The branching strategy is as follows:

| Branch | Purpose | Version Behavior | Deployment |
|---|---|---|---|
| `develop` | Prerelease branch for features, updated via PRs from feature branches | Updates prerelease suffix (e.g. `1.0.0-rc.x`) | 🚀 Deploys to `dev` |
| `main` | Production-ready code, updated via PRs from `develop` or hotfix branches | Bumps to next stable release (e.g. `1.0.0`) | 🚀 Deploys to `int` |
| `feature/*` | New features or bug fixes, merged into `develop` when complete | ➖ No version update | 🔍 Analysis & tests only |
| `hotfix/*` | Urgent fixes for production issues, merged into both `main` and `develop` | ➖ No version update | 🔍 Analysis & tests only |

---

## 🚀 Prod Releases

There is currently no automated workflow for production releases. A pipeline that can be manually triggered to deploy a specific version to any stage is planned for the future.

---

## 📚 Documentation

| Guide | Description |
|---|---|
| [🎨 Frontend Workflow](docs/frontend-workflow.md) | Setup guide for Node/pnpm frontend projects |
| [📋 Backend Workflow](docs/backend-workflow.md) | Setup guide for Java/Maven backend projects |
| [🗂️ Workflows Overview](docs/workflows-overview.md) | Reference for all individual workflows in `.github/workflows/` |