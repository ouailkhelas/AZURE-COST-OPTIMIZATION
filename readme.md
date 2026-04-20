# Azure Cost Optimization Toolkit

**Automated cost analysis and optimization for Azure subscriptions**

## Overview

This toolkit helps reduce Azure costs by 20-40% through **automated scripts and Azure Portal-based optimization**. It combines PowerShell automation for analysis with manual Azure Portal tasks for implementation.

**Real-world result:** Reduced e-commerce platform costs from $6,800/month to $4,200/month (38% savings = $2,600/month)

### Implementation Approach
- **Analysis**: Automated PowerShell scripts identify optimization opportunities
- **Implementation**: Combination of scripts and Azure Portal manual tasks
- **Verification**: Portal-based validation and Cost Management dashboards

## Features

### 🔍 **Cost Analysis**
- Complete subscription cost breakdown by resource type
- Month-over-month spending trends
- Identify top 10 cost drivers
- Budget vs. actual comparison

### 💰 **Optimization Recommendations**

**Compute Optimization:**
- VM right-sizing based on 30-day utilization metrics
- Identify idle VMs
- Reserved Instance opportunities
- Auto-shutdown recommendations for dev/test environments

**Storage Optimization:**
- Orphaned disk detection 
- Storage tier recommendations (Hot → Cool for infrequent data = 50% savings)
- Snapshot retention policy analysis
- Blob lifecycle management suggestions

**Network Optimization:**
- Unused public IP addresses
- Load Balancer rule consolidation opportunities
- VNet peering cost analysis
- Egress traffic optimization

**Backup & DR:**
- Over-retention detection (align retention to business needs)
- Backup frequency optimization
- Cool tier backup recommendations

### 📊 **Reports Generated**

1. **Executive Summary**: High-level savings overview
2. **Detailed Recommendations**: Resource-specific actions with ROI
3. **Implementation Roadmap**: Prioritized 30/60/90-day plan
4. **Monthly Tracking**: Before/after comparison dashboard

## Quick Start

### Prerequisites

- Azure subscription with Reader or Cost Management Reader role
- PowerShell 7.0+ or Azure Cloud Shell
- Az PowerShell module

### Installation

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/azure-cost-optimization.git
cd azure-cost-optimization

# Install required modules
Install-Module -Name Az -Repository PSGallery -Force
```

### Basic Usage

```powershell
# Connect to Azure
Connect-AzAccount

# Run full cost analysis
.\scripts\Analyze-AzureCosts.ps1 -SubscriptionId "your-subscription-id"

# Output: HTML report in ./reports/ folder
```


## Key Optimization Areas

### Compute Resources
- **Right-sizing VMs**: Analyze 30-day CPU/memory metrics
- **Reserved Instances**: 1-year = 40% off, 3-year = 60% off
- **Auto-shutdown**: Dev/test VMs outside business hours 

### Storage Resources
- **Hot tier**: Frequently accessed data (default)
- **Cool tier**: Infrequent access (30+ days) = 50% cheaper storage
- **Archive tier**: Rarely accessed (180+ days) = 95% cheaper
- **Orphaned disks**: Automatic detection and cleanup recommendations

### Networking
- **Public IPs**: Identify and remove unused ($4-6/month each)
- **NAT Gateway**: Consolidate outbound traffic = egress cost reduction
- **VNet peering**: Optimize cross-region data transfer

### Backup & Retention
- **Align retention**: 7 days (dev), 30 days (prod), 90 days (compliance)
- **Cool tier backups**: Long-term retention at 50% cost
- **Incremental snapshots**: Only changed blocks = massive savings
- 

**Implementation Method: Hybrid Approach (Scripts + Portal)**

### Phase 1: Analysis (Scripted)
Ran `Analyze-AzureCosts.ps1` to identify:
- 8 over-provisioned VMs (CPU avg: 22%)
- 12 orphaned managed disks
- 8 unused public IPs
- 400GB data not accessed in 60+ days
- Dev/test VMs running 24/7

### Phase 2: Implementation (Portal + Scripts)

**Portal-Based Tasks** (manual execution):
1. ✅ **VM Right-Sizing** (Azure Portal → Virtual Machines)
   - Selected each VM → "Size" → Changed D4s_v3 to D2s_v3
   - Tested in staging environment first
   - Scheduled resize during maintenance window
   - **Saved: $960/month**

2. ✅ **Storage Tier Optimization** (Azure Portal → Storage Accounts)
   - Identified infrequent blobs via "Last Accessed" timestamp
   - Storage Account → Lifecycle Management → Created rule:
     - Move to Cool tier after 30 days of no access
     - Archive tier after 180 days
   - **Saved: $56/month**

3. ✅ **Orphaned Disk Cleanup** (Azure Portal → Disks)
   - Filtered disks where "Disk state" = Unattached
   - Verified no snapshots or dependencies
   - Created snapshot backups before deletion (safety measure)
   - Deleted 12 disks manually
   - **Saved: $540/month**

4. ✅ **Unused Public IP Removal** (Azure Portal → Public IP Addresses)
   - Filtered "Associated to" = None
   - Verified IPs not reserved for future use
   - Deleted 8 unused IPs
   - **Saved: $32/month**

5. ✅ **Auto-Shutdown Policies** (Azure Portal → DevTest Labs / VM Auto-Shutdown)
   - Configured auto-shutdown for dev/test VMs:
     - Shutdown time: 7:00 PM weekdays
     - Auto-start: 8:00 AM weekdays
     - No operation on weekends
   - **Saved: $340/month**

6. ✅ **Reserved Instances Purchase** (Azure Portal → Reservations)
   - Cost Management → Advisor Recommendations → Reserved Instances
   - Purchased 1-year RIs for 8 production VMs
   - Applied to correct subscription scope
   - **Saved: $672/month**

**Script-Based Tasks** (automated):
- Monthly cost analysis report generation
- Orphaned resource detection alerts
- Backup retention policy verification

### Phase 3: Monitoring (Portal Dashboards)
- Created Azure Cost Management dashboard
- Configured budget alerts at 80%, 90%, 100%
- Weekly email reports to stakeholders

## Usage Examples

### Workflow: Analysis → Portal Implementation → Verification

**Step 1: Run Analysis Script**

```powershell
.\scripts\Analyze-AzureCosts.ps1 -SubscriptionId "your-sub-id"
```

**Step 2: Review Recommendations (Output)**

```
=== COST OPTIMIZATION SUMMARY ===
Potential Monthly Savings: $2,600 (38%)

Top Recommendations:
• Orphaned Disk: backup-disk-20240115 - Save $73/mo
• VM Right-Sizing: production-web-01 - Save $70/mo
• Unused Public IP: pip-unused-01 - Save $4/mo
```

**Step 3: Implement via Azure Portal**

### Common Portal-Based Tasks

#### Task 1: Delete Orphaned Disk
**Portal Navigation:**
1. Azure Portal → All Services → Disks
2. Filter: "Disk state" = Unattached
3. Select disk → Click "Delete"
4. Confirm deletion

**Why Portal vs Script:**
- Visual verification before deletion
- Check for hidden dependencies
- Create snapshot if needed (one-click)

---

#### Task 2: Resize VM
**Portal Navigation:**
1. Azure Portal → Virtual Machines → Select VM
2. Stop VM (required for resize)
3. Settings → Size → Select new size
4. Click "Resize" → Start VM

**Why Portal vs Script:**
- See all available sizes visually
- Compare pricing side-by-side
- Test in staging first (easier validation)

**Alternative: Script-based resize** (for bulk operations)
```powershell
Stop-AzVM -ResourceGroupName "rg-prod" -Name "vm-web-01" -Force
$vm = Get-AzVM -ResourceGroupName "rg-prod" -Name "vm-web-01"
$vm.HardwareProfile.VmSize = "Standard_D2s_v3"
Update-AzVM -ResourceGroupName "rg-prod" -VM $vm
Start-AzVM -ResourceGroupName "rg-prod" -Name "vm-web-01"
```

---

#### Task 3: Configure Storage Lifecycle Management
**Portal Navigation:**
1. Storage Account → Data Management → Lifecycle Management
2. Add rule → "Move blob to cool storage"
3. Conditions: Last modified > 30 days ago
4. Save rule

**Why Portal:**
- Visual rule builder
- Preview affected blobs before applying
- Test rules on single container first

---

#### Task 4: Purchase Reserved Instances
**Portal Navigation:**
1. Azure Portal → Cost Management + Billing → Reservations
2. "+ Add" → Compute → Select VM series
3. Choose term (1-year or 3-year)
4. Scope: Shared or Single subscription
5. Review pricing → Purchase

**Why Portal:**
- See exact savings calculation
- Compare 1-year vs 3-year ROI
- Immediate payment confirmation

---

### Find Orphaned Disks

```powershell
.\scripts\Get-OrphanedResources.ps1 -ResourceType "Disk" -SubscriptionId "your-sub-id"

# Output:
# Disk Name: backup-disk-20240115
# Size: 512GB Premium SSD
# Cost: $73.60/month
# Last Attached: Never
# Recommendation: Delete (can recreate from snapshot if needed)

# Next step: Delete via Portal (see Task 1 above)
```

### VM Right-Sizing Analysis

```powershell
.\scripts\Get-VMRightSizing.ps1 -Days 30 -SubscriptionId "your-sub-id"

# Output:
# VM: production-web-01
# Current: Standard_D4s_v3 ($140/month)
# Avg CPU: 18% | Avg Memory: 32%
# Recommended: Standard_D2s_v3 ($70/month)
# Potential Savings: $70/month ($840/year)

# Next step: Resize via Portal (see Task 2 above)
```

### Generate Full Cost Report

```powershell
.\scripts\Export-CostReport.ps1 -SubscriptionId "your-sub-id" -OutputPath "./reports"

# Generates:
# - cost-analysis-YYYY-MM-DD.html (interactive dashboard)
# - recommendations.csv (actionable items for Portal implementation)
# - savings-summary.json (API-ready data)
```

## RBAC Requirements

**Minimum Required Roles:**
- **Cost Management Reader**: View cost data and recommendations
- **Reader**: View resource configurations (for right-sizing analysis)

**For Implementation (optional):**
- **Contributor**: Apply optimization recommendations
- **Owner**: Delete orphaned resources

## Configuration

Edit `config.json` to customize thresholds:

```json
{
  "vm_cpu_threshold": 80,
  "vm_idle_threshold": 5,
  "analysis_period_days": 30,
  "storage_cool_tier_days": 30,
  "backup_retention_dev": 7,
  "backup_retention_prod": 30
}
```
