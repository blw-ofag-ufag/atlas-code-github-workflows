# 📋 Backend Workflow

The backend workflow (`backend_workflow.yml`) orchestrates the full CI/CD pipeline for Java/Maven backends. It chains the following steps: checkstyle, secret scanning, OWASP dependency check, unit tests & SonarQube analysis, Trivy vulnerability scan, semantic versioning, Docker image build & push to ECR, and infrastructure update.

## 🛠️ Setup

1. 📁 Copy the contents of `.github/examples/backend` to the root of your repository.
2. 🏷️ Update the version in `.github/workflows/backend_workflow.yml` to match the latest git tag of this repo.
3. 📦 Update dependencies in package.json
4. 🤖 Create a GitHub App for semantic versioning named after your repository with the semver postfix (e.g. `ATLAS-AGATE-BACKEND-SEMVER`) with the following permissions:
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

| Variable | Description |
|---|---|
| `INFRASTRUCTURE_REPO` | Name of the infrastructure repository (e.g. `pc-core-blw-agate-dev`) |

#### 🔐 Environment-specific Secrets

| Secret                    | Description                                                                                                                                                                             |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `AWS_DEPLOYMENT_ROLE_ARN` | ARN of the IAM Role for OIDC authentication (created in Terraform in the backend blueprint after setting the repository for the application, search for "*-app-builder" in IAM > Roles) |

#### 🤖 Required GitHub Apps

- `ATLAS-AGATE-TEST-BACKEND-SEMVER` (or equivalent)
- `GH-ORG-GITLEAKS`
- `SonarQubeCloud`

#### 🏢 Organization Secrets

- `InfraRepo_DEPLOY_PRIVATE_KEY` (e.g. `PC_CORE_BLW_AGATE_DEV_DEPLOY_PRIVATE_KEY`)
- `SONAR_TOKEN`
- `NIST_OWASP_API_KEY`
- `LICENSE_KEY_GITLEAKS`
- `GH_ORG_GITLEAKS_PRIVATE_KEY`

#### 🏢 Organization Variables

- `AWS_REGION`
- `GH_ORG_GITLEAKS_APP_ID`
- `InfraRepo_DEPLOY_APP_ID` (e.g. `PC_CORE_BLW_AGATE_DEV_DEPLOY_APP_ID`)

---

## 🔬 SonarQube Configuration

1. Add your repository to the BLW SonarQube app: https://github.com/organizations/blw-ofag-ufag/settings/installations/105700821
2. Create a new project in [BLW SonarCloud](https://sonarcloud.io/organizations/bundesamt-fur-landwirtschaft-blw/projects).
3. In SonarCloud, open the project → **Administration** → **Analysis Method** and disable **Automatic Analysis**.
4. Under **Administration** → **Branches**, set the long-living branch pattern to: `^(main|develop)$`
5. Add the following plugins to your `pom.xml`:

```xml
<!-- the quarkus-jacoco dependency is only needed if you use quarkus -->
<dependencies>
    <!-- collects coverage data for @QuarkusTest and generates report -->
    <dependency>
        <groupId>io.quarkus</groupId>
        <artifactId>quarkus-jacoco</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
<plugins>
    <!-- Manages the SonarQube scanner version -->
    <plugin>
        <groupId>org.sonarsource.scanner.maven</groupId>
        <artifactId>sonar-maven-plugin</artifactId>
        <version>${sonar-maven-plugin.version}</version>
    </plugin>

    <!-- Executes unit tests. argLine is modified by the JaCoCo agent and must be passed through.
         You may omit logging-manager for non-Quarkus projects. -->
    <plugin>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>${surefire-plugin.version}</version>
        <configuration>
            <argLine>@{argLine}</argLine>
            <systemPropertyVariables>
                <java.util.logging.manager>org.jboss.logmanager.LogManager</java.util.logging.manager>
                <maven.home>${maven.home}</maven.home>
            </systemPropertyVariables>
        </configuration>
    </plugin>

    <!-- Gathers test coverage. Uses a separate destFile to avoid conflicts across modules. -->
    <plugin>
        <groupId>org.jacoco</groupId>
        <artifactId>jacoco-maven-plugin</artifactId>
        <version>${jacoco-maven-plugin.version}</version>
        <executions>
            <execution>
                <id>prepare-agent</id>
                <goals>
                    <goal>prepare-agent</goal>
                </goals>
                <configuration>
                    <!-- used to exclude all @QuarkusTest from the normal jacoco plugin, this data will be collected by the quarkus-jacoco dependency. -->
                    <exclClassLoaders>*QuarkusClassLoader</exclClassLoaders>
                    <destFile>${project.build.directory}/jacoco-quarkus.exec</destFile>
                    <append>true</append>
                </configuration>
            </execution>
            <!-- if you are not using quarkus you want to remove the exclusion for QuarkusClassLoader above and 
                 activate report execution. (Quarkus generates report with quarkus-jacoco dependency) 
            <execution>
                <id>report</id>
                <phase>test</phase>
                <goals>
                    <goal>report</goal>
                </goals>
                <configuration>
                    <dataFile>${project.build.directory}/jacoco-quarkus.exec</dataFile>
                    <outputDirectory>${project.build.directory}/jacoco-report</outputDirectory>
                </configuration>
            </execution>-->
        </executions>
    </plugin>


    <!-- Executes integration tests. Can be omitted if there are none.
         Passes the same argLine as Surefire to ensure JaCoCo coverage works for integration tests. -->
    <plugin>
        <artifactId>maven-failsafe-plugin</artifactId>
        <version>${surefire-plugin.version}</version>
        <executions>
            <execution>
                <goals>
                    <goal>integration-test</goal>
                    <goal>verify</goal>
                </goals>
            </execution>
        </executions>
        <configuration>
            <argLine>@{argLine}</argLine>
            <systemPropertyVariables>
                <native.image.path>${project.build.directory}/${project.build.finalName}-runner</native.image.path>
                <java.util.logging.manager>org.jboss.logmanager.LogManager</java.util.logging.manager>
                <maven.home>${maven.home}</maven.home>
            </systemPropertyVariables>
        </configuration>
    </plugin>
</plugins>
```

Add the following project-specific properties to your `pom.xml`:

```xml
<properties>
    <sonar-maven-plugin.version>5.5.0.6356</sonar-maven-plugin.version>
    <surefire-plugin.version>3.5.5</surefire-plugin.version>
    <jacoco-maven-plugin.version>0.8.14</jacoco-maven-plugin.version>

    <sonar.organization>bundesamt-fur-landwirtschaft-blw</sonar.organization>
    <sonar.projectKey>blw-ofag-ufag_atlas-agate-test-backend</sonar.projectKey>
</properties>
```

---

## 🎨 Checkstyle

Add the following plugin to your `pom.xml`:

```xml
<properties>
    <checkstyle.version>13.3.0</checkstyle.version>
    <maven-checkstyle-plugin.version>3.6.0</maven-checkstyle-plugin.version>
</properties>

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>${maven-checkstyle-plugin.version}</version>
    <executions>
        <execution>
            <phase>verify</phase>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <consoleOutput>true</consoleOutput>
        <configLocation>checkstyle.xml</configLocation>
        <violationSeverity>warning</violationSeverity>
        <sourceDirectories>
            <sourceDirectory>${project.build.sourceDirectory}</sourceDirectory>
        </sourceDirectories>
    </configuration>
    <dependencies>
        <dependency>
            <groupId>com.puppycrawl.tools</groupId>
            <artifactId>checkstyle</artifactId>
            <version>${checkstyle.version}</version>
        </dependency>
    </dependencies>
</plugin>
```

Copy `examples/backend/checkstyle.xml` to the root of your repository and import it in your IDE's Checkstyle plugin.

---

## 🛡️ OWASP NIST Dependency Check

Add the following plugin to your `pom.xml`:

```xml
<properties>
    <maven-checkstyle-plugin.version>3.6.0</maven-checkstyle-plugin.version>
    <dependency-check-maven.version>12.2.0</dependency-check-maven.version>
</properties>

<plugin>
    <groupId>org.owasp</groupId>
    <artifactId>dependency-check-maven</artifactId>
    <version>${dependency-check-maven.version}</version>
    <configuration>
        <failBuildOnCVSS>7</failBuildOnCVSS>
        <skipProvidedScope>true</skipProvidedScope>
        <skipTestScope>true</skipTestScope>
        <suppressionFile>.owaspignore.xml</suppressionFile>
    </configuration>
</plugin>
```

`examples/backend/.owaspignore.xml` is an example suppression file, used to temporarily ignore vulnerabilities until fixes are available. See the [OWASP Dependency Check documentation](https://jeremylong.github.io/DependencyCheck/dependency-check-maven/configuration.html#suppressionFile) for configuration details.