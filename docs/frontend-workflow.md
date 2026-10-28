# 🎨 Frontend Workflow

The frontend workflow (`frontend_workflow.yml`) orchestrates the full CI/CD pipeline for Node/pnpm-based frontends. It chains the following steps: secret scanning, npm audit, Trivy vulnerability scan, unit tests & SonarQube analysis, semantic versioning, publishing a versioned release to S3, opening a PR against the infrastructure repository to bump the deployed version, and a main→develop merge.

> [!NOTE]
> Releases are published to an immutable `releases/v<version>/` path in the S3 bucket — the workflow no longer writes to the bucket root or invalidates a CloudFront cache. The release only goes live once the generated infrastructure PR (which updates `<application-name>_version` in `terraform.auto.tfvars.json`) is reviewed and merged, which re-applies Terraform with the new `origin_path`.

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
7. 🤖 Create (or reuse) a GitHub App for the infrastructure update, installed on the target infrastructure repository, with the following permissions:
   - **Contents:** Read & Write
   - **Pull requests:** Read & Write
   - **Metadata:** Read-only
8. 📦 Only if the app uses private packages from GitHub Packages (e.g. `@blw-ofag-ufag/atlas-ui`): see [Private GitHub Packages dependencies](#-private-github-packages-dependencies).

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
| `FRONTEND_S3_BUCKET` | `agate-frontend-681832766879` | S3 bucket where the frontend release is published |
| `INFRASTRUCTURE_REPO` | `pc-core-blw-agate-dev` | Name of the infrastructure repository to open the version-bump PR against |
| `DEPLOYMENT_APP_ID` | `123456` | App ID of the GitHub App used to authenticate against the infrastructure repository |

#### 🔐 Environment-specific Secrets

| Secret | Example | Description |
|---|---|---|
| `AWS_DEPLOYMENT_ROLE_ARN` | `arn:aws:iam::xxxxx:role/agate-frontend-deployment-role` | ARN of the IAM Role for deployment (created in Terraform) |
| `AWS_OIDC_ROLE_ARN` | `arn:aws:iam::xxxxx:role/agate-frontend-github-oidc-role` | ARN of the IAM Role for OIDC authentication (created in Terraform) |
| `DEPLOYMENT_APP_PRIVATE_KEY` | — | Private key of the infrastructure-update GitHub App (matching `DEPLOYMENT_APP_ID`) |

> [!NOTE]
> `CLOUDFRONT_DISTRIBUTION_ID` is no longer used — the workflow stopped invalidating CloudFront caches in favor of the versioned-release + infra-PR flow. Remove it from existing environments.

#### 🤖 Required GitHub Apps

- `ATLAS-AGATE-TEST-FRONTEND-SEMVER` (or equivalent)
- A deployment app installed on the infrastructure repository (see setup step 7), referenced by `DEPLOYMENT_APP_ID` / `DEPLOYMENT_APP_PRIVATE_KEY`
- `GH-ORG-GITLEAKS`
- `SonarQubeCloud`

#### ⚙️ Caller Workflow Inputs

In addition to the variables/secrets above, the calling workflow must pass these literal `with:` inputs when invoking `frontend_workflow.yml`:

| Input | Example | Description |
|---|---|---|
| `deployment-app-id` | `${{ vars.DEPLOYMENT_APP_ID }}` | App ID of the infrastructure deployment GitHub App |
| `application-name` | `agate_frontend` | Must match the `<application-name>_version` variable already present in the infrastructure repository's `terraform.auto.tfvars.json` |
| `github-organization` | `blw-ofag-ufag` | Organization that owns the infrastructure repository |
| `infrastructure_repo` | `${{ vars.INFRASTRUCTURE_REPO }}` | Name of the infrastructure repository |
| `infrastructure_base_branch` | `main` | Base branch for the generated infrastructure PR (optional, defaults to `main`) |

#### 🏢 Organization Secrets

- `SONAR_TOKEN`
- `LICENSE_KEY_GITLEAKS`
- `GH_ORG_GITLEAKS_PRIVATE_KEY`

#### 🏢 Organization Variables

- `AWS_REGION`
- `GH_ORG_GITLEAKS_APP_ID`

---

## 📦 Private GitHub Packages dependencies

GitHub Packages needs a token even for reading, so `pnpm install` in the pipeline cannot download private `@blw-ofag-ufag/*` packages without one. Every workflow that installs dependencies (`node_sonarqube`, `node_quality`, `semantic_release`, `frontend_build_deploy_s3`) writes the workflow's `GITHUB_TOKEN` into the runner's `~/.npmrc` before `pnpm install` (like the token a developer keeps in their user `.npmrc`). For the app this means:

1. 🗂️ Route the scope to GitHub Packages in the repository's `.npmrc` (no token in the repository):
   ```
   @blw-ofag-ufag:registry=https://npm.pkg.github.com
   ```
2. 🔐 Add `packages: read` to the `permissions` of the calling workflow (see `.github/examples/frontend`). A reusable workflow never gets more permissions than its caller, so without it the install step has no read access.
3. ✅ In the package settings (organisation → Packages → the package → **Package settings** → **Manage Actions access**) add the repository with the **Read** role.
4. ⏱️ pnpm 11 holds back freshly published versions for a day (`minimumReleaseAge`). Apps either wait or list the package in `minimumReleaseAgeExclude` in `pnpm-workspace.yaml`.

---

## 🔬 SonarQube Configuration

1. Add your repository to the BLW SonarQube app: https://github.com/organizations/blw-ofag-ufag/settings/installations/105700821
2. Create a new project in [BLW SonarCloud](https://sonarcloud.io/organizations/bundesamt-fur-landwirtschaft-blw/projects).
3. In SonarCloud, open the project → **Administration** → **Analysis Method** and disable **Automatic Analysis**.
4. Under **Administration** → **Branches**, set the long-living branch pattern to: `^(main|develop)$`