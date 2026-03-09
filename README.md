# ☁️ Azure Infrastructure Automation Toolkit

> **Windows PowerShell DevOps Scripts for automated Azure infrastructure provisioning, monitoring, and cleanup.**

**Author:** Nandhagopal M

---

## 📋 Table of Contents

- [Overview](#overview)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Scripts](#scripts)
  - [01 - Setup Environment](#01---setup-environment)
  - [02 - Provision Infrastructure](#02---provision-infrastructure)
  - [03 - Health Check & Monitoring](#03---health-check--monitoring)
  - [04 - Cleanup Resources](#04---cleanup-resources)
- [Infrastructure Architecture](#infrastructure-architecture)
- [Configuration](#configuration)
- [Contributing](#contributing)

---

## Overview

This toolkit automates the full lifecycle of Azure infrastructure — from initial environment setup to provisioning, monitoring, and teardown. It supports multiple environments (`dev`, `staging`, `prod`) and follows a 3-tier architecture with Virtual Networks, Subnets, NSGs, Storage, and Log Analytics.

---

## Project Structure

```
azure-devops-automation/
├── README.md
├── scripts/
│   ├── 01-setup-environment.ps1        # Azure CLI setup & service principal creation
│   ├── 02-provision-infrastructure.ps1 # Full 3-tier infrastructure provisioning
│   ├── 03-deploy-application.ps1       # Application deployment (coming soon)
│   ├── 04-monitoring-setup.ps1         # Health checks & monitoring
│   └── 05-cleanup-resources.ps1        # Safe resource cleanup
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
└── config/
    └── config.json                     # Auto-generated after setup
```

---

## Prerequisites

| Requirement | Version | Link |
|---|---|---|
| Windows PowerShell | 5.1+ | Pre-installed on Windows |
| Azure CLI | Latest | [Install Guide](https://aka.ms/installazurecliwindows) |
| Azure Subscription | Active | [Azure Portal](https://portal.azure.com) |
| Terraform *(optional)* | 1.x+ | [Download](https://developer.hashicorp.com/terraform/downloads) |

---

## Quick Start

```powershell
# 1. Clone the repository
git clone https://github.com/<your-username>/azure-devops-automation.git
cd azure-devops-automation/scripts

# 2. Set up the environment
.\01-setup-environment.ps1 -SubscriptionId "your-subscription-id"

# 3. Provision infrastructure for dev environment
.\02-provision-infrastructure.ps1 -Environment "dev"

# 4. Run health checks
.\04-monitoring-setup.ps1 -Environment "dev"

# 5. Clean up when done
.\05-cleanup-resources.ps1 -Environment "dev" -Confirm
```

---

## Scripts

### 01 - Setup Environment

`scripts/01-setup-environment.ps1`

Validates Azure CLI installation, authenticates to Azure, sets the active subscription, creates a **Service Principal** for automation, and saves configuration to `config/config.json`.

```powershell
.\01-setup-environment.ps1 -SubscriptionId "your-subscription-id" [-Location "East US"]
```

| Parameter | Required | Default | Description |
|---|---|---|---|
| `SubscriptionId` | ✅ Yes | — | Your Azure Subscription ID |
| `Location` | ❌ No | `East US` | Azure region for resources |

**What it does:**
- ✅ Checks Azure CLI installation
- ✅ Performs device-code login
- ✅ Sets active subscription
- ✅ Creates a scoped Service Principal (Contributor role)
- ✅ Saves configuration to `config/config.json`

---

### 02 - Provision Infrastructure

`scripts/02-provision-infrastructure.ps1`

Deploys a complete **3-tier Azure infrastructure** including Resource Group, VNet, Subnets, NSG with security rules, Storage Account, and Log Analytics Workspace.

```powershell
.\02-provision-infrastructure.ps1 -Environment "dev" [-AutoApprove]
```

| Parameter | Required | Options | Description |
|---|---|---|---|
| `Environment` | ✅ Yes | `dev`, `staging`, `prod` | Target deployment environment |
| `AutoApprove` | ❌ No | Switch | Skip confirmation prompts |

**Resources Provisioned:**

| Resource | Name Pattern | Details |
|---|---|---|
| Resource Group | `devops-{env}-rg` | Container for all resources |
| Virtual Network | `devops-{env}-vnet` | Address space: `10.0.0.0/16` |
| Web Subnet | `web-subnet` | `10.0.1.0/24` |
| App Subnet | `app-subnet` | `10.0.2.0/24` |
| DB Subnet | `db-subnet` | `10.0.3.0/24` |
| NSG | `devops-{env}-nsg` | HTTP (80) & HTTPS (443) rules |
| Storage Account | `devops{env}{rand}` | Standard LRS |
| Log Analytics | `devops-{env}-logs` | Monitoring workspace |

Deployment details are saved to `config/deployment-{env}.json`.

---

### 03 - Health Check & Monitoring

`scripts/04-monitoring-setup.ps1`

Validates all deployed resources and generates a **timestamped health report** in JSON format under `reports/`.

```powershell
.\04-monitoring-setup.ps1 -Environment "dev"
```

| Parameter | Required | Description |
|---|---|---|
| `Environment` | ✅ Yes | Target environment to check |

**Checks Performed:**
- ✅ Resource Group existence
- ✅ Virtual Network status & subnet count
- ✅ Storage Account health & SKU
- ✅ Log Analytics Workspace status
- ✅ Total resource count by type

Reports are saved to: `reports/health-check-{env}-{timestamp}.json`

---

### 04 - Cleanup Resources

`scripts/05-cleanup-resources.ps1`

Safely deletes all resources in an environment's resource group. Prompts for confirmation by default.

```powershell
.\05-cleanup-resources.ps1 -Environment "dev" [-Confirm]
```

| Parameter | Required | Description |
|---|---|---|
| `Environment` | ✅ Yes | Environment to clean up |
| `Confirm` | ❌ No | Skip the interactive yes/no prompt |

> ⚠️ **Warning:** This permanently deletes all resources in the resource group. Use with caution in `prod`.

---

## Infrastructure Architecture

```
Azure Subscription
└── Resource Group: devops-{env}-rg
    ├── Virtual Network: 10.0.0.0/16
    │   ├── web-subnet:  10.0.1.0/24  ← Public-facing tier
    │   ├── app-subnet:  10.0.2.0/24  ← Application tier
    │   └── db-subnet:   10.0.3.0/24  ← Data tier
    ├── NSG: HTTP (80) + HTTPS (443) rules
    ├── Storage Account (Standard LRS)
    └── Log Analytics Workspace
```

---

## Configuration

After running `01-setup-environment.ps1`, a `config/config.json` file is created:

```json
{
  "subscription_id": "your-subscription-id",
  "location": "East US",
  "sp_client_id": "auto-generated-client-id",
  "created_date": "2024-01-01 12:00:00"
}
```

After provisioning, a per-environment file is saved to `config/deployment-{env}.json`:

```json
{
  "environment": "dev",
  "resource_group": "devops-dev-rg",
  "vnet_name": "devops-dev-vnet",
  "storage_account": "devopsdev1234",
  "workspace_name": "devops-dev-logs",
  "deployed_at": "2024-01-01 12:00:00"
}
```

> 🔒 **Security Note:** Never commit `config.json` or `deployment-*.json` to version control. Add them to `.gitignore`.

---

## .gitignore Recommendation

```gitignore
config/config.json
config/deployment-*.json
reports/
*.tfstate
*.tfstate.backup
.terraform/
```

---

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m 'Add your feature'`
4. Push to the branch: `git push origin feature/your-feature`
5. Open a Pull Request

---

## License

This project is licensed under the MIT License.

---

*Built with ❤️ by Nandhagopal M*
