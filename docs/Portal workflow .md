# Azure Portal Workflow Guide

Practical Azure Portal operations performed alongside Terraform deployment for monitoring and governance.

---

# 🚀 Workflow

## Phase 1 — Terraform Deployment

Infrastructure deployed with Terraform:

- Log Analytics Workspace
- Action Groups
- Metric Alerts
- Saved KQL Queries
- RBAC Definitions

```bash
terraform init
terraform plan
terraform apply
```

---

## Phase 2 — Azure Portal Configuration

Portal used for visualization, monitoring, troubleshooting, and fine-tuning.

---

# 📊 Portal Tasks

## Azure Monitor Workbooks
- Create monitoring dashboards
- Add CPU and memory charts
- Build custom visualizations
- Add workbook parameters
- Share dashboards with teams

---

## KQL Query Development
- Analyze VM performance
- Detect memory pressure
- Troubleshoot resources
- Save reusable queries
- Create alerts from queries

Example:

```kql
Perf
| where ObjectName == "Memory"
| summarize AvgMemory = avg(CounterValue) by Computer
```

---

## Dynamic Threshold Alerts
- Configure smart CPU alerts
- Use Azure Monitor ML-based thresholds
- Reduce false positives
- Monitor production workloads

---

## RBAC Management
- Assign Monitoring Contributor roles
- Validate user permissions
- Manage IAM access
- Apply least privilege access

---

## Shared Dashboards
- Build operational dashboards
- Add monitoring tiles
- Add cost management views
- Share dashboards with teams

---

## Alert Processing Rules
- Suppress alerts during maintenance
- Configure scheduled rules
- Reduce alert noise

---

## Multi-Dimensional Alerts
- Monitor Application Gateway pools
- Track backend health
- Automatically include new resources

---

# ⚖️ Terraform vs Portal

## Terraform
Best for:
- Infrastructure deployment
- Version control
- Reusable configurations
- Base alert definitions

## Azure Portal
Best for:
- KQL testing
- Workbook design
- Dashboard sharing
- Dynamic threshold tuning
- Troubleshooting

---

# ✅ Best Practices

- Test alerts in dev first
- Use naming conventions
- Apply resource tags
- Review audit logs
- Export configurations
- Document KQL queries
- Share dashboards with teams

---

# 🛠 Technologies

- Terraform
- Azure Monitor
- Azure Portal
- Log Analytics
- KQL
- Azure RBAC
- Azure Cost Management
