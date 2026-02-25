# 🗂️ Workflows Overview

All workflows are located in `.github/workflows/`. Every workflow is a reusable `workflow_call` workflow, meaning it is designed to be called from a parent workflow — typically `backend_workflow.yml` or `frontend_workflow.yml`.

---

## 🔗 Orchestration Workflows

These are the entry points that chain all individual workflows together.

| Workflow | Description |
|---|---|
| [`backend_workflow.yml`](../.github/workflows/backend_workflow.yml) | 🏗️ Master pipeline for Java/Maven backends — runs checkstyle, secret scan, OWASP check, unit tests, Trivy scan, semantic release, Docker build/push to ECR, and infrastructure update |
| [`frontend_workflow.yml`](../.github/workflows/frontend_workflow.yml) | 🎨 Master pipeline for Node/pnpm frontends — runs secret scan, npm audit, Trivy scan, unit tests, semantic release, S3 deploy, and main→develop merge |

---

## 🔒 Security & Scanning

| Workflow | Description |
|---|---|
| [`gitleaks.yml`](../.github/workflows/gitleaks.yml) | 🔑 Scans the codebase and PRs for accidentally committed secrets using Gitleaks; posts findings as PR comments via GitHub App |
| [`trivy_scan.yml`](../.github/workflows/trivy_scan.yml) | 🛡️ Runs a Trivy filesystem scan for HIGH/CRITICAL vulnerabilities and credential leaks |
| [`frontend_npm_audit.yml`](../.github/workflows/frontend_npm_audit.yml) | 📦 Runs `pnpm audit` on production dependencies to detect high and critical severity vulnerabilities |
| [`backend_owasp_dependency_check.yml`](../.github/workflows/backend_owasp_dependency_check.yml) | 🛡️ Runs OWASP Dependency Check against Maven dependencies using the NIST vulnerability database |

---

## 🧪 Code Quality & Testing

| Workflow | Description |
|---|---|
| [`backend_unit_test_sonarqube.yml`](../.github/workflows/backend_unit_test_sonarqube.yml) | ✅ Runs Maven unit tests and SonarQube analysis with JaCoCo coverage reports |
| [`frontend_unit_test_sonarqube.yml`](../.github/workflows/frontend_unit_test_sonarqube.yml) | ✅ Lints and runs unit tests on the Node frontend, then performs SonarQube analysis |
| [`backend_checkstyle.yml`](../.github/workflows/backend_checkstyle.yml) | 🎨 Runs Maven Checkstyle to enforce code formatting and style rules on the Java backend |
| [`check_single_commit.yml`](../.github/workflows/check_single_commit.yml) | 1️⃣ Fails the pipeline if a PR contains more than one commit, enforcing commit squashing before merge |

---

## 🚀 Release & Versioning

| Workflow | Description |
|---|---|
| [`semantic_release.yml`](../.github/workflows/semantic_release.yml) | 🏷️ Automatically creates releases and version tags from semantic commit messages (`fix`/`feat`/`BREAKING`) — fully automated, no manual step required |
| [`release-please.yml`](../.github/workflows/release-please.yml) | 🔀 Alternative to semantic release — creates release PRs based on conventional commits, allowing manual review before the release is published |

---

## 🏗️ Build & Deploy

| Workflow | Description |
|---|---|
| [`backend_build_push_image.yml`](../.github/workflows/backend_build_push_image.yml) | 🐳 Builds the Java backend with Maven, packages it into a multi-platform Docker image (amd64 + arm64), and pushes it to AWS ECR |
| [`frontend_build_deploy_s3.yml`](../.github/workflows/frontend_build_deploy_s3.yml) | ☁️ Builds the Node frontend with pnpm and deploys it to an S3 bucket, then invalidates the CloudFront cache |
| [`update_infrastructure.yml`](../.github/workflows/update_infrastructure.yml) | 🔧 Updates `terraform.auto.tfvars.json` with the new Docker image tag in a separate infrastructure repository and opens a PR for review |

---

## 🔄 Repository Maintenance

| Workflow | Description |
|---|---|
| [`merge_main_develop.yml`](../.github/workflows/merge_main_develop.yml) | 🔀 Fast-forward merges `main` into `develop` after a release to keep the development branch in sync with production |
| [`trigger_scan_and_release.yml`](../.github/workflows/trigger_scan_and_release.yml) | 🔁 Used by **this repository itself** — runs Gitleaks on the workflows codebase and uses release-please to manage releases on push/PR to main |