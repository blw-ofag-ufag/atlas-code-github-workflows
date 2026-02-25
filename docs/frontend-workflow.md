# 🎨 Frontend Workflow

The frontend workflow (`frontend_workflow.yml`) orchestrates the full CI/CD pipeline for Node/pnpm-based frontends. It chains the following steps: secret scanning, npm audit, Trivy vulnerability scan, unit tests & SonarQube analysis, semantic versioning, S3 deployment, and a main→develop merge.

## 🛠️ Setup

1. 📁 Copy the contents of `.github/examples/frontend` to the root of your repository.
2. 🏷️ Update the version in `.github/workflows/frontend_workflow.yml` to match the latest git tag of this repo.
3. 📦 Update dependencies in `package.json` and add any unfixable entries to `ignoreGhsas`.
4. 🔧 Update `.releaserc.json` to match your Jira configuration.
5. 🔬 Update `sonar-project.properties` to match your SonarQube configuration.
6. 🤖 Create a GitHub App for semantic versioning named after your repository with the semver postfix (e.g. `ATLAS-AGATE-FRONTEND-SEMVER`) with the following permissions:
   - **Actions:** Read & Write
   - **Contents:** Read & Write
   - **Issues:** Read & Write
   - **Metadata:** Read-only

---

#### 📦 Repository Variables

| Variable | Description |
|---|---|
| `SEMVER_APP_ID` | GitHub App ID of the semver app you just created |

#### 🔑 Repository Secrets

| Secret | Description |
|---|---|
| `SEMVER_PRIVATE_KEY` | Private key of the semver app you just created |

#### 🌍 Environment-specific Variables

| Variable | Example | Description |
|---|---|---|
| `CLOUDFRONT_DISTRIBUTION_ID` | `E2D4KHJF4Q9SX2` | CloudFront distribution ID to invalidate after deployment |
| `FRONTEND_S3_BUCKET` | `agate-frontend-681832766879` | S3 bucket where the frontend is deployed |

#### 🔐 Environment-specific Secrets

| Secret | Example | Description |
|---|---|---|
| `AWS_DEPLOYMENT_ROLE_ARN` | `arn:aws:iam::xxxxx:role/agate-frontend-deployment-role` | ARN of the IAM Role for deployment (created in Terraform) |
| `AWS_OIDC_ROLE_ARN` | `arn:aws:iam::xxxxx:role/agate-frontend-github-oidc-role` | ARN of the IAM Role for OIDC authentication (created in Terraform) |

#### 🤖 Required GitHub Apps

- `ATLAS-AGATE-TEST-FRONTEND-SEMVER` (or equivalent)
- `GH-ORG-GITLEAKS`
- `SonarQubeCloud`

#### 🏢 Organization Secrets

- `SONAR_TOKEN`
- `LICENSE_KEY_GITLEAKS`
- `GH_ORG_GITLEAKS_PRIVATE_KEY`

#### 🏢 Organization Variables

- `AWS_REGION`
- `GH_ORG_GITLEAKS_APP_ID`

---

## 🔬 SonarQube Configuration

1. Add your repository to the BLW SonarQube app: https://github.com/organizations/blw-ofag-ufag/settings/installations/105700821
2. Create a new project in [BLW SonarCloud](https://sonarcloud.io/organizations/bundesamt-fur-landwirtschaft-blw/projects).
3. In SonarCloud, open the project → **Administration** → **Analysis Method** and disable **Automatic Analysis**.
4. Under **Administration** → **Branches**, set the long-living branch pattern to: `^(main|develop)$`