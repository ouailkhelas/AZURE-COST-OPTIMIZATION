# Azure Portal Workflow Guide - Monitoring Dashboard

This guide covers the manual Azure Portal tasks performed alongside Terraform deployment for the monitoring solution.

## Why Hybrid Approach (Terraform + Portal)?

**Terraform is best for:**
- Repeatable infrastructure deployment
- Version-controlled alert definitions
- Multi-environment consistency (dev/staging/prod)
- Initial RBAC role definitions

**Portal is best for:**
- Visual KQL query development and testing
- Custom workbook design with drag-and-drop
- Dynamic threshold configuration (requires ML visualization)
- Ad-hoc troubleshooting and analysis
- Dashboard sharing and collaboration
- Fine-tuning alert thresholds based on real data

---

## Complete Deployment Workflow

### Phase 1: Terraform Deployment (Infrastructure)

```bash
# Deploy base infrastructure
terraform init
terraform plan -var="subscription_id=your-sub-id"
terraform apply -auto-approve

# Outputs:
# - Log Analytics Workspace
# - Action Groups (email/webhook)
# - Base metric alerts (CPU, memory, disk)
# - Saved KQL queries
# - RBAC role definitions
```

### Phase 2: Portal Configuration (Customization)

---

## Portal Task 1: Create Custom Azure Monitor Workbook

**When:** After Terraform deploys Log Analytics Workspace  
**Why:** Visual workbook designer is faster than JSON/Bicep templates  
**Time:** 30-60 minutes per workbook

### Steps:

1. **Navigate to Workbooks**
   - Azure Portal → **Monitor** → **Workbooks**
   - Click **+ New**

2. **Add Query Visualization (VM CPU Trend)**
   - Click **Add** → **Add query**
   - Data source: **Logs**
   - Resource: Select Log Analytics Workspace
   - KQL Query:
     ```kql
     Perf
     | where TimeGenerated > ago(24h)
     | where ObjectName == "Processor" and CounterName == "% Processor Time"
     | summarize AvgCPU = avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
     | render timechart
     ```
   - Visualization: **Line chart**
   - Title: "VM CPU Utilization (24 hours)"
   - Click **Done Editing**

3. **Add Metrics Visualization (Memory Across VMs)**
   - Click **Add** → **Add metric**
   - Scope: Select all production VMs
   - Metric: **Available Memory Bytes**
   - Aggregation: **Average**
   - Time range: **Last 24 hours**
   - Chart type: **Line chart**

4. **Add Parameters (Time Range Selector)**
   - Click **Add** → **Add parameters**
   - Parameter name: `TimeRange`
   - Parameter type: **Time range picker**
   - Default value: **Last 24 hours**
   - Click **Save**
   - Update queries to use: `{TimeRange:start}` and `{TimeRange:end}`

5. **Add Text Section (Documentation)**
   - Click **Add** → **Add text**
   - Content (Markdown):
     ```markdown
     ## VM Performance Dashboard
     
     This dashboard tracks:
     - CPU utilization trends
     - Memory consumption
     - Disk IOPS and latency
     - Network throughput
     
     **Alert if:** CPU > 85% for 10+ minutes
     ```

6. **Save Workbook**
   - Click **Done Editing** → **Save**
   - Title: "VM Performance Dashboard"
   - Subscription: Select
   - Resource group: `rg-monitoring`
   - Location: `East US`
   - **Apply**

7. **Share Workbook**
   - Click **Share** → Get shareable link
   - OR Pin to dashboard for easy access

---

## Portal Task 2: Develop and Test KQL Queries

**When:** Ongoing troubleshooting and analysis  
**Why:** Interactive query development with instant results  
**Time:** 10-30 minutes per query

### Workflow Example: "Find VMs with High Memory Pressure"

1. **Open Log Analytics**
   - Portal → **Log Analytics Workspaces** → Select workspace
   - Click **Logs**

2. **Write Query Iteratively**

   **Step 1 - Get all memory data:**
   ```kql
   Perf
   | where TimeGenerated > ago(1h)
   | where ObjectName == "Memory"
   ```

   **Step 2 - Filter to "Available MBytes":**
   ```kql
   Perf
   | where TimeGenerated > ago(1h)
   | where ObjectName == "Memory" and CounterName == "Available MBytes"
   ```

   **Step 3 - Calculate average per VM:**
   ```kql
   Perf
   | where TimeGenerated > ago(1h)
   | where ObjectName == "Memory" and CounterName == "Available MBytes"
   | summarize AvgMemoryMB = avg(CounterValue) by Computer
   ```

   **Step 4 - Find VMs with < 1GB available:**
   ```kql
   Perf
   | where TimeGenerated > ago(1h)
   | where ObjectName == "Memory" and CounterName == "Available MBytes"
   | summarize AvgMemoryMB = avg(CounterValue) by Computer
   | where AvgMemoryMB < 1024
   | order by AvgMemoryMB asc
   ```

3. **Test Query**
   - Click **Run**
   - Verify results make sense
   - Adjust time range and thresholds

4. **Save Query**
   - Click **Save** → **Save as query**
   - Name: "VMs with Low Memory"
   - Category: "Performance"
   - **Save**

5. **Create Alert from Query**
   - Click **+ New alert rule**
   - Condition: Already populated from query
   - Threshold: Results > 0 (if any VM has low memory)
   - Evaluation frequency: Every 5 minutes
   - Add action group
   - **Create alert rule**

---

## Portal Task 3: Configure Dynamic Threshold Alerts

**When:** After collecting 7+ days of metric data  
**Why:** Requires historical data visualization to validate ML patterns  
**Time:** 15 minutes per alert

### Steps:

1. **Create Alert Rule**
   - Portal → **Monitor** → **Alerts** → **+ Create** → **Alert rule**

2. **Select Scope**
   - Click **Select resource**
   - Filter: Virtual machines
   - Select all production VMs
   - **Done**

3. **Configure Condition**
   - Click **Add condition**
   - Signal: **Percentage CPU**
   - Alert logic:
     - **Operator:** Greater than
     - **Aggregation type:** Average
     - **Threshold:** Dynamic
     - **Threshold sensitivity:** Medium (High = more sensitive, Low = less)
     - **Aggregation granularity:** 5 minutes
     - **Frequency of evaluation:** Every 1 minute

4. **Preview Dynamic Threshold**
   - View chart showing:
     - Actual CPU values (blue line)
     - Dynamic threshold band (gray area)
     - Violations (red dots)
   - Adjust sensitivity if needed

5. **Configure Actions**
   - Click **Next: Actions**
   - Select action group: `ag-devops-team`

6. **Add Details**
   - Alert rule name: `vm-cpu-dynamic-alert`
   - Severity: Warning
   - Description: "ML-based CPU alerting for production VMs"
   - **Review + create**

---

## Portal Task 4: Assign RBAC Roles

**When:** After user groups created in Azure AD  
**Why:** Visual group search and permission validation  
**Time:** 5 minutes per assignment

### Scenario: Assign Monitoring Contributor to DevOps Team

1. **Navigate to Resource**
   - Portal → **Resource groups** → `rg-monitoring`
   - Left menu → **Access control (IAM)**

2. **Check Existing Access**
   - Click **View** → **Role assignments**
   - Review current assignments
   - Export to CSV for audit trail

3. **Add Role Assignment**
   - Click **+ Add** → **Add role assignment**

4. **Select Role (Step 1)**
   - **Role** tab
   - Search: "Monitoring Contributor"
   - Select role → **Next**

5. **Select Members (Step 2)**
   - **Members** tab
   - **Assign access to:** User, group, or service principal
   - Click **+ Select members**
   - Search: "DevOps Team" (Azure AD group name)
   - Select group → **Select**
   - **Next**

6. **Review + Assign (Step 3)**
   - Verify:
     - Scope: `/subscriptions/.../resourceGroups/rg-monitoring`
     - Role: Monitoring Contributor
     - Member: DevOps Team group
   - **Review + assign**

7. **Verify Assignment**
   - IAM → **Check access**
   - Enter group name → Verify role appears
   - Test with member of DevOps team:
     - Should see monitoring resources
     - Can create new alerts
     - Cannot delete Log Analytics Workspace

### Repeat for Other Roles:
- **Monitoring Reader** → Developers group
- **Backup Contributor** → Backup Admins group

---

## Portal Task 5: Create Shared Dashboard

**When:** After workbooks and alerts configured  
**Why:** Executive summary view for stakeholders  
**Time:** 30 minutes

### Steps:

1. **Create New Dashboard**
   - Portal home → Click **Dashboard**
   - Click **+ New dashboard** → **Blank dashboard**
   - Name: "Production Monitoring Overview"

2. **Add Metrics Tile (CPU Across All VMs)**
   - Click **Edit** → **Tile Gallery**
   - Search: "Metrics chart"
   - Drag to canvas
   - Configure:
     - **Scope:** Resource group `rg-production`
     - **Metric Namespace:** Virtual Machines
     - **Metric:** Percentage CPU
     - **Aggregation:** Average
     - **Chart type:** Line chart
   - Resize tile: Medium width
   - **Apply**

3. **Add Workbook Tile**
   - Tile Gallery → Search: "Workbook"
   - Configure:
     - **Workbook:** Select "VM Performance Dashboard"
     - **Show header:** Yes
   - Resize: Large width
   - **Apply**

4. **Add Query Results Tile (Failed Backups)**
   - Tile Gallery → "Logs"
   - Enter KQL query:
     ```kql
     AzureDiagnostics
     | where Category == "AzureBackupReport"
     | where TimeGenerated > ago(24h)
     | where JobStatus_s == "Failed"
     | summarize FailedBackups = count() by Resource
     ```
   - Visualization: **Grid**
   - Title: "Failed Backups (Last 24h)"
   - **Apply**

5. **Add Cost Tile**
   - Tile Gallery → "Azure Cost Management"
   - Configure:
     - **View:** Daily costs
     - **Time range:** Last 30 days
     - **Granularity:** Daily
   - **Apply**

6. **Add Text Tile (Instructions)**
   - Tile Gallery → "Markdown"
   - Content:
     ```markdown
     ## Production Monitoring Dashboard
     
     **Status:** All systems operational
     **Last updated:** Real-time
     
     ### Alert Thresholds:
     - CPU > 85% for 10 minutes
     - Memory < 1GB available
     - Disk < 15% free
     
     **On-call:** DevOps rotation (see PagerDuty)
     ```
   - **Apply**

7. **Organize and Save**
   - Drag tiles to arrange
   - Resize for optimal layout
   - Click **Done customizing**
   - **Save**

8. **Share Dashboard**
   - Click **Share**
   - Select Azure AD groups: Management, DevOps
   - **Publish**

---

## Portal Task 6: Configure Alert Processing Rules

**When:** Planned maintenance windows or known high-load events  
**Why:** Suppress alerts during expected deviations  
**Time:** 10 minutes

### Scenario: Suppress Alerts During Monthly Patching

1. **Navigate to Alert Processing**
   - Portal → **Monitor** → **Alerts** → **Alert processing rules**
   - Click **+ Create**

2. **Define Scope**
   - **Scope:** Select resource group `rg-production`
   - **Filter:**
     - Resource type = Virtual machines
     - Severity = 2, 3, 4 (exclude critical alerts)
   - **Next**

3. **Configure Rule**
   - **Rule type:** Suppress notifications
   - **Suppression time:** Scheduled
   - **Schedule:**
     - Recurrence: Monthly
     - Day: Second Tuesday (Patch Tuesday)
     - Start time: 10:00 PM
     - Duration: 4 hours
   - **Next**

4. **Add Details**
   - **Rule name:** `suppress-patching-alerts`
   - **Description:** "Suppress non-critical alerts during monthly patching window"
   - **Resource group:** `rg-monitoring`
   - **Next**

5. **Review + Create**
   - Verify schedule and scope
   - **Create**

---

## Portal Task 7: Set Up Metric Alert with Multiple Dimensions

**When:** Monitoring Application Gateway with multiple backend pools  
**Why:** Single alert covering all pools with granular tracking  
**Time:** 10 minutes

### Steps:

1. **Create Alert**
   - Portal → **Monitor** → **Alerts** → **+ Create**
   - Select **Application Gateway** resource

2. **Configure Multi-Dimensional Alert**
   - Condition: **Unhealthy Host Count**
   - **Dimensions:**
     - BackendSettingsPool = All current and future values
     - BackendHttpSettings = All current and future values
   - Alert logic:
     - Operator: Greater than
     - Threshold: 0
     - Aggregation: Maximum
   - **Done**

3. **Dynamic Alert Splitting**
   - Each backend pool gets its own alert instance
   - Automatically includes new pools without reconfiguration

4. **Configure Actions**
   - Action group: `ag-devops-team`
   - **Create alert rule**

---

## Portal vs Terraform Decision Matrix

| Task | Portal | Terraform | Reasoning |
|------|--------|-----------|-----------|
| **Deploy Log Analytics Workspace** | ❌ | ✅ | Infrastructure, needs version control |
| **Create workbook template** | ❌ | ✅ | Repeatable across environments |
| **Design workbook visuals** | ✅ | ❌ | Visual designer faster than JSON |
| **Write KQL queries** | ✅ | ❌ | Interactive testing essential |
| **Create static threshold alerts** | ❌ | ✅ | Codified, reviewable thresholds |
| **Configure dynamic thresholds** | ✅ | ❌ | Requires ML visualization |
| **Assign RBAC roles** | Both | Both | Terraform for base, Portal for ad-hoc |
| **Create shared dashboards** | ✅ | ❌ | Visual layout tool |
| **Alert processing rules** | ✅ | ❌ | Schedule visualization helpful |
| **Action group webhooks** | ❌ | ✅ | Sensitive URLs, version controlled |

---

## Integration Workflow

**Recommended Process:**

```
Week 1: Terraform Deployment
├── Deploy infrastructure via terraform apply
├── Verify Log Analytics Workspace created
├── Verify base alerts firing
└── Test action groups (send test notification)

Week 2: Portal Customization
├── Create 3 custom workbooks (VM, Network, Cost)
├── Develop 15+ KQL queries for common scenarios
├── Configure dynamic thresholds for production VMs
└── Assign RBAC to user groups

Week 3: Dashboard and Sharing
├── Build executive dashboard with key metrics
├── Create operational dashboard for on-call engineers
├── Share workbooks with teams
└── Document alert response procedures

Week 4: Tuning
├── Review alert history (false positives?)
├── Adjust dynamic threshold sensitivity
├── Add alert processing rules for known events
└── Export audit trail of all RBAC assignments
```

---

## Best Practices for Portal Work

1. **Always test in dev first** - Create alerts in non-prod before prod
2. **Use naming conventions** - Prefix alerts with environment (`prod-cpu-alert`)
3. **Tag resources** - Makes workbook filtering easier
4. **Export configurations** - Backup JSON from Portal before major changes
5. **Document custom queries** - Add comments in KQL for future reference
6. **Review audit logs** - Monthly RBAC assignment review
7. **Leverage parameters** - Workbook parameters for time ranges, filters
8. **Share knowledge** - Export workbooks for team collaboration

---

## Common Portal Tasks Reference

| Task | Portal Path | Time |
|------|-------------|------|
| **View alert history** | Monitor → Alerts → Alert history | 2 min |
| **Test action group** | Monitor → Alerts → Action groups → Test | 1 min |
| **Export RBAC audit** | IAM → Download role assignments | 1 min |
| **Clone workbook** | Workbooks → Edit → Save As | 3 min |
| **Create dashboard from workbook** | Workbook → Pin to dashboard | 2 min |
| **Share query** | Log Analytics → Queries → Export | 1 min |
| **View Log Analytics costs** | Workspace → Usage and estimated costs | 1 min |

---

## Troubleshooting

**Alert not firing:**
1. Portal → Monitor → Alerts → Alert rules → Find rule
2. Click rule → "Alert evaluation history"
3. Check if metric breached threshold
4. Verify action group configured correctly

**Workbook query timeout:**
1. Optimize KQL query (add time filters)
2. Use summarize to reduce data points
3. Increase timeout in workbook settings

**RBAC not working:**
1. Portal → IAM → Check access → Enter user
2. Verify role assignment scope
3. Check Azure AD group membership
4. Clear user's browser cache

**Dashboard not loading:**
1. Verify shared with user's Azure AD group
2. Check if underlying resources still exist
3. Refresh dashboard → Edit → Republish