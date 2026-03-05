# 🔐 DevSecOps Pipeline — Netflix Clone

A complete **DevSecOps CI/CD pipeline** that integrates security scanning at every stage of the software delivery process — from source code to Docker image to Kubernetes deployment.

---

## 🏗️ Pipeline Architecture

```
Code Push
    ↓
Clean Workspace
    ↓
Git Checkout
    ↓
SonarQube Analysis ──→ Code quality, bugs, code smells
    ↓
Quality Gate Check ──→ Pass/Fail based on SonarQube rules
    ↓
npm Install
    ↓
OWASP Dependency Check ──→ Scan dependencies for known CVEs
    ↓
Trivy Filesystem Scan ──→ Scan source code & packages
    ↓
Docker Build & Push ──→ Build image, push to DockerHub
    ↓
Trivy Image Scan ──→ Scan Docker image for vulnerabilities
    ↓
Update K8s Deployment YAML ──→ GitOps: auto-update image tag in Git
    ↓
Deploy to Container
```

---

## 🛡️ Security Stages Explained

| Stage | Tool | What It Checks |
|---|---|---|
| `SonarQube Analysis` | SonarQube | Code quality, bugs, vulnerabilities, code smells in source code |
| `Quality Gate` | SonarQube | Enforces minimum quality standards before proceeding |
| `OWASP Dependency Check` | OWASP DP-Check | Scans `node_modules` for known CVEs in third-party libraries |
| `Trivy Filesystem Scan` | Trivy | Scans source code, configs, and packages for secrets & vulnerabilities |
| `Trivy Image Scan` | Trivy | Scans the final Docker image for OS-level and library vulnerabilities |

---

## 🔄 GitOps — Auto Image Tag Update

This pipeline uses a **GitOps approach** for Kubernetes deployments:

1. Jenkins builds the Docker image and pushes to DockerHub
2. Jenkins then **automatically updates** the `deployment.yml` in the Git repo with the new image tag (using `BUILD_NUMBER`)
3. The updated manifest is committed and pushed back to GitHub
4. A GitOps tool like **ArgoCD or Flux** detects the change and syncs it to the cluster automatically

```bash
# What this sed command does:
sed -i 's#image: ahmed7860/netflix:.*#image: ahmed7860/netflix:${BUILD_NUMBER}#' deployment.yml
# Replaces whatever image tag was there with the new build number
```

---

## 🛠️ Jenkins Prerequisites

Before using this pipeline, ensure Jenkins has the following configured:

**Tools (Manage Jenkins → Tools):**
- JDK 17 installed and named `jdk17`
- Node.js 16 installed and named `node16`
- SonarQube Scanner installed and named `sonar-scanner`
- OWASP Dependency Check installed and named `DP-Check`
- Docker installed and named `docker`

**Credentials (Manage Jenkins → Credentials):**
- `Sonar-token` — SonarQube authentication token
- `docker` — DockerHub username and password
- `github-creds` — GitHub username and personal access token

**Plugins Required:**
- SonarQube Scanner Plugin
- OWASP Dependency Check Plugin
- Docker Pipeline Plugin
- NodeJS Plugin

---

## 📊 Scan Reports

After every pipeline run, two security reports are generated and archived:

- `trivyfs.txt` — Trivy filesystem vulnerability scan results
- `trivyimage.txt` — Trivy Docker image vulnerability scan results

These are automatically archived as **Jenkins build artifacts** for audit and review.

---

## 📚 Key Concepts Demonstrated

- **Shift-left security** — security checks run early in the pipeline, not just at the end
- **GitOps** — Kubernetes manifests managed in Git, auto-updated by CI pipeline
- **Image immutability** — every build gets a unique tag (`BUILD_NUMBER`) for full traceability
- **Least privilege** — credentials stored in Jenkins, never hardcoded in pipeline
- **Fail fast** — Quality Gate and scan stages catch issues before they reach production
