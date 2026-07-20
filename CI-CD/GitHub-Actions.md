# GitHub Actions Cheatsheet

![](https://imgur.com/GMwRo18.png)

**1. Introduction:**

- GitHub Actions is a powerful CI/CD and automation tool integrated directly into GitHub repositories, allowing you to build, test, and deploy your code.

**2. Key Concepts:**

- **Workflow:** An automated process defined in YAML that is triggered by events like `push`, `pull_request`, etc.
- **Job:** A set of steps that runs on the same runner.
- **Step:** An individual task, such as running a script or installing a dependency.
- **Runner:** A server that runs the jobs in a workflow, can be GitHub-hosted or self-hosted.

**3. Basic Workflow Example:**

- **YAML Syntax:**

  ```yaml
  name: CI Workflow

  on:
    push:
      branches:
        - main
    pull_request:
      branches:
        - main

  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v3
        - name: Set up Node.js
          uses: actions/setup-node@v3
          with:
            node-version: '14'
        - run: npm install
        - run: npm test
  ```

**4. Common Actions:**

- **actions/checkout:** Checks out your repository under `$GITHUB_WORKSPACE`.
- **actions/setup-node:** Sets up a Node.js environment.
- **actions/upload-artifact:** Uploads build artifacts for later use.
- **actions/cache:** Caches dependencies like `node_modules` or `Maven`.

**5. Triggers:**

- **on: push:** Trigger a workflow when a push occurs.
- **on: pull_request:** Trigger a workflow when a pull request is opened.
- **on: schedule:** Schedule a workflow to run at specific times using cron syntax.

**6. Environment Variables:**

- **Set environment variables:**

  ```yaml
  env:
    NODE_ENV: production
    DEBUG: true
  ```

- **Access secrets:**

  ```yaml
  env:
    MY_SECRET: ${{ secrets.MY_SECRET }}
  ```

**7. Matrix Builds:**

- **Example:**

  ```yaml
  jobs:
    build:
      runs-on: ubuntu-latest
      strategy:
        matrix:
          node-version: [12, 14, 16]
      steps:
        - uses: actions/checkout@v3
        - name: Set up Node.js
          uses: actions/setup-node@v3
          with:
            node-version: ${{ matrix.node-version }}
        - run: npm install
        - run: npm test
  ```

**8. Artifacts and Caching:**

- **Upload Artifacts:**

  ```yaml
  - name: Upload build artifacts
    uses: actions/upload-artifact@v3
    with:
      name: my-artifact
      path: ./build
  ```

- **Caching Dependencies:**

  ```yaml
  - name: Cache Node.js modules
    uses: actions/cache@v3
    with:
      path: node_modules
      key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
      restore-keys: |
        ${{ runner.os }}-node-
  ```

**9. Reusable Workflows:**

- **Define a reusable workflow:**

  ```yaml
  name: Reusable CI Workflow

  on:
    workflow_call:
      inputs:
        node-version:
          required: true
          type: string

  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v3
        - name: Set up Node.js
          uses: actions/setup-node@v3
          with:
            node-version: ${{ inputs.node-version }}
        - run: npm install
        - run: npm test
  ```

- **Call a reusable workflow:**

  ```yaml
  jobs:
    call-workflow:
      uses: ./.github/workflows/reusable-workflow.yml
      with:
        node-version: '14'
  ```

**10. Best Practices:**

- **Modular Workflows:** Break down complex workflows into smaller, reusable pieces.
- **Use Environments:** Leverage environments in GitHub Actions for deployments with manual approvals.
- **Secret Management:** Always use GitHub Secrets for sensitive information and never hard-code them.

  # Production DevSecOps + GitOps Cheat Sheet

---

# DevSecOps Pipeline Overview

```text
Developer
    │
    ▼
Git Commit
    │
    ▼
GitHub Actions
    │
    ├── Checkout Source
    ├── Dependency Restore
    ├── Build
    ├── Unit Tests
    ├── Static Code Analysis (SAST)
    ├── Secret Scan
    ├── Dependency Scan (SCA)
    ├── IaC Scan
    ├── Build Container Image
    ├── Container Scan
    ├── Generate SBOM
    ├── Sign Image
    ├── Push Image
    │
    ▼
Update GitOps Repository
    │
    ▼
Git Commit (Manifest Updated)
    │
    ▼
ArgoCD / Flux
    │
    ▼
Kubernetes Deployment
```

---

# CI vs CD

| CI | CD |
|----|----|
| Continuous Integration | Continuous Delivery/Deployment |
| Build application | Deploy application |
| Run Tests | Release application |
| Static Analysis | Kubernetes Deployment |
| Create Artifacts | Rollback if needed |

---

# DevSecOps Stages

```
Source
 ↓
Build
 ↓
Test
 ↓
Security
 ↓
Package
 ↓
Release
 ↓
Deploy
 ↓
Monitor
```

---

# Production Pipeline

```
Git Push

↓

Checkout

↓

Install Dependencies

↓

Unit Tests

↓

Code Coverage

↓

SonarQube

↓

Secret Scan

↓

Dependency Scan

↓

IaC Scan

↓

Build Docker Image

↓

Container Scan

↓

Generate SBOM

↓

Image Signing

↓

Push Image

↓

Update GitOps Repository

↓

ArgoCD Sync

↓

Deployment

↓

Monitoring
```

---

# Security Scanning Types

| Type | Purpose | Popular Tools |
|-------|----------|---------------|
| SAST | Analyze source code | SonarQube, CodeQL, Semgrep |
| DAST | Scan running application | OWASP ZAP, Burp Suite |
| SCA | Dependency vulnerabilities | Dependency-Check, Snyk, Trivy |
| Secret Scan | Detect leaked credentials | Gitleaks, Trufflehog |
| IaC Scan | Scan Terraform/Kubernetes | Checkov, Trivy Config, Terrascan |
| Container Scan | Docker image vulnerabilities | Trivy, Grype |
| Runtime Security | Detect attacks in cluster | Falco |

---

# GitOps Workflow

```
Application Repository

↓

GitHub Actions

↓

Build Image

↓

Push Image

↓

Update GitOps Repository

↓

Git Commit

↓

ArgoCD Watches Repository

↓

Sync

↓

Deploy to Kubernetes
```

---

# Why Separate GitOps Repository?

Application Repository

```
Contains

- Source Code
- Dockerfile
- Tests
- CI Pipeline
```

GitOps Repository

```
Contains

- Kubernetes Manifests
- Helm Charts
- Kustomize
- Environment Configurations
```

Advantages

- Audit Trail
- Easy Rollback
- Separation of Concerns
- Environment Isolation

---

# Kubernetes Deployment Flow

```
Developer

↓

Git Push

↓

GitHub Actions

↓

Container Registry

↓

GitOps Repository Updated

↓

ArgoCD Detects Change

↓

Sync

↓

Kubernetes

↓

Pods Updated
```

---

# Image Lifecycle

```
Docker Build

↓

Docker Image

↓

Security Scan

↓

Generate SBOM

↓

Sign Image

↓

Push Registry

↓

Deploy
```

---

# SBOM (Software Bill of Materials)

Contains

- Packages
- Versions
- Licenses
- Dependencies
- Checksums

Popular Tools

- Syft
- Anchore
- CycloneDX

Benefits

- Supply Chain Security
- Vulnerability Tracking
- Compliance

---

# Image Signing

Purpose

Verify image authenticity.

Popular Tool

Cosign

Flow

```
Build Image

↓

Sign Image

↓

Push Registry

↓

Verify Before Deployment
```

Benefits

- Prevent image tampering
- Verify publisher
- Supply chain security

---

# Supply Chain Security

Protects

- Source Code
- Dependencies
- Build Server
- Container Images
- Deployment Artifacts

Popular Standards

- SLSA
- Sigstore
- Cosign
- SBOM

---

# GitHub Actions Concepts

Workflow

```
.github/workflows/main.yml
```

Trigger

```yaml
on:
  push:
  pull_request:
```

Job

```yaml
jobs:
  build:
```

Step

```yaml
steps:
```

Action

```yaml
uses:
```

Shell Command

```yaml
run:
```

Environment Variable

```yaml
env:
```

Secrets

```yaml
${{ secrets.MY_SECRET }}
```

Outputs

```yaml
outputs:
```

Needs

```yaml
needs:
```

Matrix

```yaml
strategy:
  matrix:
```

---

# GitHub Actions Execution Order

```
Workflow

↓

Jobs

↓

Steps

↓

Actions / Commands
```

---

# GitHub Actions Contexts

| Context | Example |
|----------|----------|
| github | github.ref |
| secrets | secrets.TOKEN |
| env | env.APP_NAME |
| vars | vars.VERSION |
| runner | runner.os |
| needs | needs.build.outputs.tag |
| matrix | matrix.java |

---

# GitHub Actions Best Practices

- Use reusable workflows
- Cache dependencies
- Pin action versions
- Never hardcode secrets
- Use environments
- Protect production deployments
- Enable branch protection
- Run scans in parallel
- Fail on Critical vulnerabilities
- Upload SARIF reports
- Generate artifacts
- Use concurrency groups
- Use OIDC authentication

---

# Docker Build Pipeline

```
Checkout

↓

Build Image

↓

Scan Image

↓

Generate SBOM

↓

Sign Image

↓

Push Registry
```

---

# Kubernetes Deployment Strategies

| Strategy | Description |
|-----------|-------------|
| Recreate | Stop old, start new |
| Rolling Update | Default strategy |
| Blue/Green | Switch traffic |
| Canary | Small percentage rollout |
| A/B Testing | Feature comparison |

---

# ArgoCD Architecture

```
Git Repository

↓

Repo Server

↓

Application Controller

↓

API Server

↓

Kubernetes Cluster
```

---

# ArgoCD Sync Policies

Manual

```
Sync manually
```

Automatic

```
Git Change

↓

Auto Sync

↓

Deploy
```

Options

- Self Heal
- Prune
- Retry

---

# Secrets Management

Avoid

```
password=admin123
```

Use

- GitHub Secrets
- Kubernetes Secrets
- HashiCorp Vault
- AWS Secrets Manager
- Azure Key Vault

---

# Production Security Checklist


# Enterprise Best Practices

- Immutable container images
- Everything as Code (IaC, Policy, Pipelines)
- GitOps for deployments
- Zero Trust authentication
- Least privilege (RBAC)
- OIDC instead of long-lived cloud credentials
- Signed commits and signed images
- Automated policy enforcement (OPA/Gatekeeper or Kyverno)
- Multi-environment promotion (dev → qa → staging → prod)
- Progressive delivery (Canary/Blue-Green)
- Automated rollback on failed health checks
- Centralized logging, metrics, and tracing
- Continuous vulnerability management
- Regular secret rotation
- Disaster recovery and backup testing
- Compliance evidence through SBOMs and attestations

## Template
```workflow
name: Production DevSecOps Pipeline

on:
  push:
    branches:
      - develop
      - release/*
      - main

permissions:
  contents: write
  packages: write
  security-events: write
  id-token: write

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: my-org/myapp

jobs:

#########################################
# Quality & Security
#########################################

  security:

    runs-on: ubuntu-latest

    outputs:
      image_tag: ${{ steps.meta.outputs.tag }}

    steps:

    - uses: actions/checkout@v4

#########################################
# Java Setup
#########################################

    - uses: actions/setup-java@v4
      with:
        distribution: temurin
        java-version: 21

#########################################
# Cache Maven
#########################################

    - uses: actions/cache@v4
      with:
        path: ~/.m2
        key: maven-${{ hashFiles('**/pom.xml') }}

#########################################
# Unit Tests
#########################################

    - name: Run Tests
      run: mvn clean verify

#########################################
# SonarQube
#########################################

    - name: Sonar Scan
      run: |
        mvn sonar:sonar \
        -Dsonar.host.url=${{ secrets.SONAR_HOST_URL }} \
        -Dsonar.login=${{ secrets.SONAR_TOKEN }}

#########################################
# Secret Scan
#########################################

    - name: Gitleaks
      uses: gitleaks/gitleaks-action@v2

#########################################
# Dependency Scan
#########################################

    - name: OWASP Dependency Check
      uses: dependency-check/Dependency-Check_Action@main
      with:
        project: myapp
        path: .

#########################################
# Build
#########################################

    - name: Build Jar
      run: mvn package -DskipTests

#########################################
# Docker Buildx
#########################################

    - uses: docker/setup-buildx-action@v3

#########################################
# Login Registry
#########################################

    - uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

#########################################
# Metadata
#########################################

    - id: meta
      run: |
        TAG=${GITHUB_SHA::8}
        echo "tag=$TAG" >> $GITHUB_OUTPUT

#########################################
# Build Image
#########################################

    - uses: docker/build-push-action@v6
      with:
        context: .
        push: false
        tags: |
          ghcr.io/my-org/myapp:${{ steps.meta.outputs.tag }}

#########################################
# Trivy Image Scan
#########################################

    - name: Trivy
      uses: aquasecurity/trivy-action@0.28.0
      with:
        image-ref: ghcr.io/my-org/myapp:${{ steps.meta.outputs.tag }}
        format: sarif
        output: trivy-results.sarif

#########################################
# Upload SARIF
#########################################

    - uses: github/codeql-action/upload-sarif@v3
      with:
        sarif_file: trivy-results.sarif

#########################################
# Generate SBOM
#########################################

    - uses: anchore/sbom-action@v0
      with:
        image: ghcr.io/my-org/myapp:${{ steps.meta.outputs.tag }}

#########################################
# Push Image
#########################################

    - uses: docker/build-push-action@v6
      with:
        context: .
        push: true
        tags: |
          ghcr.io/my-org/myapp:${{ steps.meta.outputs.tag }}

#########################################
# Cosign
#########################################

    - uses: sigstore/cosign-installer@v3

    - run: |
        cosign sign \
        --key env://COSIGN_PRIVATE_KEY \
        ghcr.io/my-org/myapp:${{ steps.meta.outputs.tag }}
      env:
        COSIGN_PRIVATE_KEY: ${{ secrets.COSIGN_PRIVATE_KEY }}
        COSIGN_PASSWORD: ${{ secrets.COSIGN_PASSWORD }}

#########################################
# GitOps Update
#########################################

  gitops:

    needs: security

    runs-on: ubuntu-latest

    steps:

    - name: Checkout GitOps Repo
      uses: actions/checkout@v4
      with:
        repository: my-org/gitops-repo
        token: ${{ secrets.GITOPS_TOKEN }}
        path: gitops

    - name: Install yq
      uses: mikefarah/yq@master

    - name: Update Manifest
      run: |
        yq -i '
        .spec.template.spec.containers[0].image =
        "ghcr.io/my-org/myapp:${{ needs.security.outputs.image_tag }}"
        ' gitops/environments/dev/deployment.yaml

    - name: Commit
      run: |
        cd gitops
        git config user.name github-actions
        git config user.email actions@github.com

        git add .
        git commit -m "Update image to ${{ needs.security.outputs.image_tag }}" || exit 0

        git push
```
