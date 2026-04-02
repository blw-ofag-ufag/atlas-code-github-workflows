# 🔄 Atlas GitHub Pipeline Workflows

Generalized CI/CD workflows for use across multiple projects.

---

## 📋 Overview

This repository contains reusable GitHub Actions workflows for CI/CD pipelines. Workflows that require AWS access need environment-specific variables and secrets to be configured.

### 🌍 Available Environments

- `dev`
- `int`
- `prod`

---

### 🔧 Installation

1. For deployment pipelines make sure the Github App: GH-ORG-APP-TOKEN-READ-WRITE is installed in your repository and the required org secrets are configured (see below).
2. For Gitleaks make sure that the Github App: GH-ORG-GITLEAKS is installed in your repository and the required org secrets are configured (see below).
3. Install one of the workflows as described below and adjust it to your needs.

## ⚙️ Configuration

### 🔐 AWS Environment Secrets

The following secrets must be configured for each environment that requires AWS access:

| Name                      | Description                                                                                     | Example                                                   |
|---------------------------|-------------------------------------------------------------------------------------------------|-----------------------------------------------------------|
| `AWS_DEPLOYMENT_ROLE_ARN` | ARN of the IAM Role that will be assumed for deployment (created in your Terraform blueprint)   | `arn:aws:iam::xxxxx:role/agate-frontend-deployment-role`  |
| `AWS_OIDC_ROLE_ARN`       | ARN of the IAM Role used for OIDC authentication with AWS (created in your Terraform blueprint) | `arn:aws:iam::xxxxx:role/agate-frontend-github-oidc-role` |

### 🏢 Organization Secrets

Request the Organization Admin to add your repository to the corresponding app and secrets in the organization settings. Once configured, these secrets will be available in your
repository settings.

| Name                          | Description                                                                                |
|-------------------------------|--------------------------------------------------------------------------------------------|
| `GH_ORG_PRIVATE_KEY`          | Private key for pushing to this repository or other repositories (app ID in org variables) |
| `GH_ORG_GITLEAKS_PRIVATE_KEY` | Private key for GitLeaks scans (app ID in org variables)                                   |
| `LICENSE_KEY_GITLEAKS`        | License key for GitLeaks                                                                   |

### 📝 Organization Variables

| Name                     | Description                                                         |
|--------------------------|---------------------------------------------------------------------|
| `AWS_REGION`             | AWS Region for deployment. Can be overridden by repository variable |
| `GH_ORG_APP_ID`          | App ID for pushing to other repositories (key in org secret)        |
| `GH_ORG_GITLEAKS_APP_ID` | App ID for GitLeaks scans (key in org secret)                       |

---

## 🎨 Frontend Workflow

To set up the frontend workflow,

1. copy `examples\frontend_trigger_default_workflow.yml` to your repo under`.github/workflows/trigger_default_workflow.yml` and adjust it to your needs.
2. copy `fontend.releaserc.json` to your repo under `.releaserc.json` and adjust it to your needs.

### 🗂️ Frontend Repository Configuration

#### Github App for Semantic Versioning

Create a Github App: named after your repository and semver postfix (e.g. ATLAS-AGATE-FRONTEND-SEMVER) with the following permissions:

- Repository permissions:
    - Actions Read & Write
    - Contents: Read & Write
    - Issues Read and Write
    - Metadata: Read-only
- Install the app in your repository and add app id and private key to the repository secrets and variables as described below.

#### 📦 Repository Variables

| Name                         | Description                                               | Example                       |
|------------------------------|-----------------------------------------------------------|-------------------------------|
| `CLOUDFRONT_DISTRIBUTION_ID` | CloudFront distribution ID to invalidate after deployment | `E2D4KHJF4Q9SX2`              |
| `FRONTEND_S3_BUCKET`         | S3 bucket where the frontend is deployed                  | `agate-frontend-681832766879` |

#### 🔑 Repository Secrets

| Name                      | Description                                                        | Example                                                   |
|---------------------------|--------------------------------------------------------------------|-----------------------------------------------------------|
| `AWS_DEPLOYMENT_ROLE_ARN` | ARN of the IAM Role for deployment (created in Terraform)          | `arn:aws:iam::xxxxx:role/agate-frontend-deployment-role`  |
| `AWS_OIDC_ROLE_ARN`       | ARN of the IAM Role for OIDC authentication (created in Terraform) | `arn:aws:iam::xxxxx:role/agate-frontend-github-oidc-role` |

### 🔬 SonarQube Configuration

1. Add your github repository to the BLW SonarQube app: https://github.com/organizations/blw-ofag-ufag/settings/installations/105700821
2. Create a new Project in [BLW SonarQube](https://sonarcloud.io/organizations/bundesamt-fur-landwirtschaft-blw/projects)
3. In SonarCloud select the new project then "administration" → "analysis method" and deselect "Automatic Analysis"
4. Configure SonarQube, by adding the following file to the root of your repository and fill in the correct values:

**File:** `sonar-project.properties`

```properties
sonar.organization=bundesamt-fur-landwirtschaft-blw
sonar.projectKey=blw-ofag-ufag_atlas-agate-frontend
sonar.sources=src
sonar.exclusions=**/node_modules/**,**/dist/**,**/assets/**,**/*.spec.ts
# Coverage
sonar.javascript.lcov.reportPaths=coverage/lcov.info
sonar.sourceEncoding=UTF-8
```

## Backend Workflow

Create a Github App for Semver: named after your repository and semver postfix (e.g. ATLAS-AGATE-BACKEND-SEMVER) with the following permissions:

- Repository permissions:
    - Actions Read & Write
    - Contents: Read & Write
    - Issues Read and Write
    - Metadata: Read-only
- Install the app in your repository and add app id and private key to the repository secrets and variables as described below.

Make sure your backend repository has the following `GH Apps` installed:

* Semver app eg: ATLAS-AGATE-TEST-BACKEND-SEMVER
* GH-ORG-GITLEAKS 
* Deployment app: PC-CORE-BLW-AGATE-DEV-DEPLOY
* SonarQubeCloud

Make sure the Repo has access to those `Organisation secrets`:

* InfraRepo_DEPLOY_PRIVATE_KEY (e.g: PC_CORE_BLW_AGATE_DEV_DEPLOY_PRIVATE_KEY)
* SONAR_TOKEN
* NIST_OWASP_API_KEY
* LICENSE_KEY_GITLEAKS
* GH_ORG_GITLEAKS_PRIVATE_KEY

And those `Organization variables`:
* AWS_REGION
* GH_ORG_GITLEAKS_APP_ID
* InfraRepo_DEPLOY_APP_ID (e.g: PC_CORE_BLW_AGATE_DEV_DEPLOY_APP_ID)

#### 📦 Repository Variables

| Name            | Description                            |
|-----------------|----------------------------------------|
| `SEMVER_APP_ID` | GH App ID used for semantic versioning |

#### 🔑 Repository Secrets

| Name                 | Description                                                  |
|----------------------|--------------------------------------------------------------|
| `SEMVER_PRIVATE_KEY` | Private key of the GH App ID of used for semantic versioning |

### 🔬 SonarQube Configuration

1. Add your github repository to the BLW SonarQube app: https://github.com/organizations/blw-ofag-ufag/settings/installations/105700821
2. Create a new Project in [BLW SonarQube](https://sonarcloud.io/organizations/bundesamt-fur-landwirtschaft-blw/projects)
3. In SonarCloud select the new project then "administration" → "analysis method" and deselect "Automatic Analysis"
4. Add sonar plugin to your pom.xml:

```xml

<dependency>
    <groupId>org.sonarsource.scanner.maven</groupId>
    <artifactId>sonar-maven-plugin</artifactId>
    <version>${sonar-maven-plugin.version}</version>
</dependency>
```

add project specific properties to your pom.xml:

```xml        

<properties>
    <sonar-maven-plugin.version>5.5.0.6356</sonar-maven-plugin.version>
    <sonar.organization>bundesamt-fur-landwirtschaft-blw</sonar.organization>
    <sonar.projectKey>blw-ofag-ufag_atlas-agate-test-backend</sonar.projectKey>
</properties>
```
