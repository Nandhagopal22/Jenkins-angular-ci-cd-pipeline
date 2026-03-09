# 🚀 Jenkins Angular CI/CD Pipeline

Enterprise-grade **CI/CD automation pipeline** for building, packaging, and deploying Angular applications using **Jenkins, Node.js, and PowerShell**.

This pipeline automates the complete application lifecycle from **code commit → build → artifact packaging → deployment**.

---

# 📌 Project Overview

This project demonstrates a production-style Jenkins pipeline used to:

- Install Angular dependencies
- Build environment-specific applications
- Clean old release artifacts
- Package application builds
- Execute automated deployment scripts

The pipeline supports **branch-based deployment environments** and integrates with **PowerShell deployment automation**.

---

# 🛠 Technologies Used

| Technology | Purpose |
|------------|--------|
| Jenkins | CI/CD automation |
| Git | Source control |
| Node.js / NPM | Dependency management |
| Angular | Web application |
| PowerShell | Deployment automation |
| 7-Zip | Artifact packaging |
| Windows Server | Build agent environment |

---

# ⚙️ Pipeline Architecture

---

# 🔄 Pipeline Stages

## 1️⃣ Initialize

Initializes reusable pipeline functions.

Functions included:

- Detect file changes between commits
- Skip builds when no relevant changes are detected
- Handle pipeline errors gracefully

---

## 2️⃣ Branch Configuration

Automatically configures environment variables based on Git branch.

Example mapping:

| Branch | Environment |
|------|------|
| Development | UAT |
| QA_LIMS_ENTERPRISE | QA |
| curaCloud_design | UAT |

Variables configured:

- Release folder
- Build environment
- Deployment script path
- Application path

---

## 3️⃣ Checkout

Clones the repository into the Jenkins workspace.

```groovy
checkout scm
4️⃣ Install Dependencies

Installs project dependencies using NPM.

npm install --force

This ensures the correct dependencies are installed before building the applicatio
