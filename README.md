# 🏗️ devops-terraform-services-teamcity

[![Build status](https://badge.buildkite.com/your-org/your-pipeline.svg?branch=main)](https://buildkite.com/your-org/your-pipeline)
[![OpenTofu](https://img.shields.io/badge/OpenTofu-%3E%3D1.6.0-blue?logo=opentofu)](https://opentofu.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Renovate](https://img.shields.io/badge/renovate-enabled-brightgreen?logo=renovatebot)](https://renovatebot.com/)
[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg)](https://conventionalcommits.org)

Infrastructure as Code for TeamCity resources using OpenTofu/Terraform.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Prerequisites](#-prerequisites)
- [Repository Structure](#-repository-structure)
- [Quick Start](#-quick-start)
- [Environments](#-environments)
- [CI/CD Pipeline](#-cicd-pipeline)
- [Configuration](#-configuration)
- [Contributing](#-contributing)

---

## 🎯 Overview

This repository manages TeamCity infrastructure including:

| Resource | Description |
|----------|-------------|
| 📁 Projects | Root and sub-projects |
| 🔧 Build Configurations | Build pipelines and steps |
| 🎨 Build Templates | Reusable build templates |
| 🔗 VCS Roots | Git/VCS connections |
| 👥 Groups | User groups and permissions |
| 🔑 Roles | Role definitions and assignments |
| ⚙️ Parameters | Project and build parameters |
| 🧹 Cleanup Rules | Artifact and history cleanup |
| 🏃 Agents | Agent pools and configurations |
| 🔌 Connections | External service connections |

---

## ✅ Prerequisites

| Tool | Version | Description |
|------|---------|-------------|
| [OpenTofu](https://opentofu.org/) | >= 1.6.0 | Infrastructure provisioning |
| [TFLint](https://github.com/terraform-linters/tflint) | >= 0.50.0 | Terraform linter |
| [Vault](https://www.vaultproject.io/) | >= 1.15.0 | Secrets management |

### Required Access

- 🔑 TeamCity access token with admin permissions
- 🔐 Vault access for secrets retrieval
- 🏗️ Buildkite agent access

---

## 📁 Repository Structure

```
.
├── 📂 .buildkite/
│   ├── pipeline.yml              # Main CI/CD pipeline
│   └── scripts/                  # PowerShell automation scripts
│       ├── setup/                # Vault auth, tool installation
│       ├── validation/           # Format, validate, lint
│       ├── security/             # tfsec, checkov, trivy
│       ├── documentation/        # terraform-docs
│       ├── versioning/           # git-cliff, semantic versioning
│       ├── terraform/            # Init, plan, apply
│       └── post-deploy/          # Smoke tests, notifications
│
├── 📂 environments/
│   ├── dev/                      # Development environment
│   │   └── terraform.tfvars.example
│   └── prd/                      # Production environment
│       └── terraform.tfvars.example
│
├── 📂 modules/                   # Local reusable modules
│   ├── teamcity-project/
│   ├── teamcity-build-config/
│   ├── teamcity-vcs-root/
│   └── teamcity-agent-pool/
│
├── 📄 main.tf                    # Root module
├── 📄 variables.tf               # Input variables
├── 📄 outputs.tf                 # Output values
├── 📄 providers.tf               # Provider configuration
├── 📄 versions.tf                # Version constraints
├── 📄 environments.yml           # CI/CD environment config
├── 📄 cliff.toml                 # Changelog configuration
├── 📄 renovate.json              # Dependency updates
└── 📄 README.md                  # This file
```

---

## 🚀 Quick Start

### 1️⃣ Clone the Repository

```bash
git clone https://github.com/your-org/teamcity-terraform.git
cd teamcity-terraform
```

### 2️⃣ Set Up Environment Variables

```bash
export TEAMCITY_URL="https://teamcity.example.com"
export TEAMCITY_TOKEN="your-access-token"
```

### 3️⃣ Initialize OpenTofu

```bash
tofu init
```

### 4️⃣ Plan Changes

```bash
tofu plan -var-file=environments/dev/terraform.tfvars
```

### 5️⃣ Apply Changes

```bash
tofu apply -var-file=environments/dev/terraform.tfvars
```

---

## 🌍 Environments

| Environment | Auto Apply | Approval Required | Branch |
|-------------|------------|-------------------|--------|
| 🟢 Dev | ✅ Yes | ❌ No | All |
| 🔴 Prd | ❌ No | ✅ Yes (platform-team) | `main` |

### Deployment Flow

```
┌─────────┐     ┌──────────┐
│   Dev   │ ──► │   Prd    │
└─────────┘     └──────────┘
 auto-apply       approval
                (platform-team)
```

---

## 🔄 CI/CD Pipeline

### Pipeline Phases

| Phase | Tools | Description |
|-------|-------|-------------|
| 1️⃣ Setup | Vault | Authenticate and install tools |
| 2️⃣ Validation | OpenTofu, TFLint | Format check, validate, lint |
| 3️⃣ Security | tfsec, checkov, trivy | Security scanning (parallel) |
| 4️⃣ Documentation | terraform-docs | Generate and check docs |
| 5️⃣ Versioning | git-cliff | Changelog and version calculation |
| 6️⃣ AI Review | fabric | AI-powered code review |
| 7️⃣ Deploy | OpenTofu | Plan and apply per environment |

### Triggering Deployments

| Action | Trigger |
|--------|---------|
| Feature branch push | Plan Dev only |
| PR to main | Plan Dev + Prd |
| Merge to main | Plan + Apply all environments |
| Manual | Buildkite UI approval |

---

## ⚙️ Configuration

### Provider Configuration

```hcl
# providers.tf
terraform {
  required_providers {
    teamcity = {
      source  = "jetbrains/teamcity"
      version = "~> 0.1"
    }
  }
}

provider "teamcity" {
  host  = var.teamcity_url
  token = var.teamcity_token
}
```

### Backend Configuration

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "your-tfstate-bucket"
    key            = "teamcity/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

### Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `teamcity_url` | TeamCity server URL | ✅ |
| `teamcity_token` | Access token for authentication | ✅ |
| `environment` | Target environment (dev/prd) | ✅ |

---

## 🏗️ Managed Resources

### Projects

```hcl
module "project" {
  source = "./modules/teamcity-project"

  name        = "My Application"
  description = "CI/CD for My Application"
  parent_id   = "Root"
  
  parameters = {
    "env.DEPLOY_TARGET" = "kubernetes"
  }
}
```

### Build Configurations

```hcl
module "build_config" {
  source = "./modules/teamcity-build-config"

  name       = "Build and Test"
  project_id = module.project.id
  
  vcs_root_id = module.vcs_root.id
  
  steps = [
    {
      type    = "script"
      name    = "Build"
      script  = "npm run build"
    },
    {
      type    = "script"
      name    = "Test"
      script  = "npm run test"
    }
  ]
  
  triggers = [
    {
      type    = "vcs"
      branch  = "+:refs/heads/*"
    }
  ]
}
```

### VCS Roots

```hcl
module "vcs_root" {
  source = "./modules/teamcity-vcs-root"

  name       = "GitHub - My App"
  project_id = module.project.id
  
  url    = "https://github.com/your-org/your-repo.git"
  branch = "refs/heads/main"
  
  auth_method = "token"
}
```

---

## 🛡️ Security

### Secrets Management

All secrets are stored in HashiCorp Vault:

| Secret | Vault Path |
|--------|------------|
| TeamCity Token | `secret/teamcity/token` |
| AWS Credentials | `secret/aws/terraform` |

### Security Scanning

| Tool | Purpose |
|------|---------|
| 🔒 tfsec | Terraform security scanner |
| ✅ checkov | Policy-as-code scanner |
| 🛡️ trivy | Misconfiguration scanner |

---

## 📝 Contributing

### Branch Naming

```
feature/TC-123-add-new-project
fix/TC-456-fix-build-config
chore/TC-789-update-providers
```

### Commit Messages

Follow [Conventional Commits](https://conventionalcommits.org):

```
feat(projects): add new build project
fix(builds): correct trigger configuration
docs(readme): update quick start guide
chore(deps): update teamcity provider
```

### Pull Request Process

1. 🔀 Create feature branch from `main`
2. ✏️ Make changes and commit
3. 🧪 Ensure CI passes
4. 📝 Update documentation if needed
5. 🔍 Request review
6. ✅ Merge after approval

---

## 📚 Resources

| Resource | Link |
|----------|------|
| 🏗️ TeamCity Docs | [jetbrains.com/teamcity/docs](https://www.jetbrains.com/help/teamcity/teamcity-documentation.html) |
| 📦 Terraform Provider | [registry.terraform.io](https://registry.terraform.io/providers/jetbrains/teamcity) |
| 🌐 OpenTofu Docs | [opentofu.org/docs](https://opentofu.org/docs) |
| 🏗️ Buildkite Docs | [buildkite.com/docs](https://buildkite.com/docs) |

---

## 📄 License

MIT License - see [LICENSE](LICENSE) for details.

---

<p align="center">
  <sub>Built with ❤️ by the Platform Team</sub>
</p>
