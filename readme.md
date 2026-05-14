# Azure Cost Optimization Toolkit

Automated Azure cost analysis and optimization using PowerShell, Azure Cost Management, and Azure Portal best practices.

---

## 🚀 Overview

This project focuses on learning and applying real-world Azure cost optimization techniques through:

- PowerShell automation
- Azure governance practices
- Cost monitoring
- Resource optimization
- Azure Portal operational workflows

The repository combines script for automated analysis with documented best practices for manual optimization inside Azure Portal.

---

# ⚙️ Script

## Analyze-AzureCosts.ps1

Main analysis script used to identify Azure cost optimization opportunities.

### Features
- VM right-sizing recommendations
- Orphaned disk detection
- Unused public IP detection
- Storage optimization checks
- Reserved Instance recommendations
- Auto-shutdown candidate detection
- Cost reporting export

---

# 📘 Azure Portal Best Practices

Documented in:

```bash
docs/Portal-Workflow.md
```

The document explains practical Azure Portal optimization workflows including:

- VM resizing
- Auto-shutdown configuration
- Storage lifecycle management
- Reserved Instance planning
- Budget alerts
- Azure Advisor recommendations
- Resource cleanup
- Governance and tagging strategies

---

# ▶️ Quick Start

```powershell
Connect-AzAccount

.\scripts\Analyze-AzureCosts.ps1 `
    -SubscriptionId "your-subscription-id"
```

---

# 🎯 Learning Focus

This project helped me understand:

- Azure cost governance
- FinOps best practices
- Resource optimization strategies
- PowerShell automation
- Azure operational management
- Cost monitoring and reporting
