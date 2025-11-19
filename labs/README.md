# AZ-104 Hands-On Labs

Welcome to the comprehensive hands-on lab collection for AZ-104 Microsoft Azure Administrator certification preparation.

## 📋 Lab Structure

Each lab includes:
- **Learning Objectives**: Clear goals aligned with exam objectives
- **Prerequisites**: Required knowledge and resources
- **Step-by-Step Instructions**: Portal, PowerShell, CLI, and Bicep/ARM methods
- **Verification Steps**: Confirm successful completion
- **Knowledge Checks**: Test your understanding
- **Cleanup Scripts**: Remove lab resources to avoid charges

## 🎯 Labs by Exam Domain

### Identity and Governance (20-25%)

| Lab | Title | Duration | Difficulty |
|-----|-------|----------|------------|
| 01 | [Managing Microsoft Entra ID Users and Groups](identity-governance/lab01-entra-id-users-groups.md) | 45 min | Beginner |
| 02 | [Azure RBAC and Resource Access](identity-governance/lab02-rbac-resource-access.md) | 60 min | Intermediate |
| 03 | Azure Policy and Governance (Coming Soon) | 45 min | Intermediate |
| 04 | Cost Management and Budgets (Coming Soon) | 30 min | Beginner |
| 05 | Management Groups (Coming Soon) | 30 min | Intermediate |

**Key Skills Covered:**
- ✅ Create and manage users and groups
- ✅ Configure Self-Service Password Reset (SSPR)
- ✅ Manage external users (B2B)
- ✅ Assign RBAC roles at different scopes
- ✅ Create custom RBAC roles
- ✅ Implement Azure Policy
- ✅ Configure resource locks
- ✅ Manage tags and cost tracking

---

### Storage (15-20%)

| Lab | Title | Duration | Difficulty |
|-----|-------|----------|------------|
| 01 | [Storage Accounts and Blob Storage](storage/lab01-storage-accounts-blob.md) | 60 min | Intermediate |
| 02 | Azure Files and File Sync (Coming Soon) | 45 min | Intermediate |
| 03 | Storage Security and Access Control (Coming Soon) | 45 min | Intermediate |

**Key Skills Covered:**
- ✅ Create storage accounts with different redundancy options
- ✅ Configure blob access tiers (Hot, Cool, Archive)
- ✅ Implement blob lifecycle management
- ✅ Configure soft delete for blobs and containers (NEW)
- ✅ Generate and use SAS tokens
- ✅ Configure stored access policies
- ✅ Use Azure Storage Explorer and AzCopy
- ✅ Configure storage firewalls and virtual networks

---

### Compute (20-25%)

| Lab | Title | Duration | Difficulty |
|-----|-------|----------|------------|
| 01 | Virtual Machines Deployment (Coming Soon) | 60 min | Beginner |
| 02 | VM Availability and Scaling (Coming Soon) | 75 min | Intermediate |
| 03 | ARM Templates and Bicep (Coming Soon) | 60 min | Intermediate |
| 04 | Azure App Service (Coming Soon) | 45 min | Beginner |
| 05 | Container Instances and Container Apps (Coming Soon) | 45 min | Intermediate |

**Key Skills Covered:**
- Virtual machine creation and configuration
- Availability sets and availability zones
- VM scale sets and auto-scaling
- ARM template deployment
- Bicep file creation and deployment
- App Service deployment and configuration
- Container deployment and management

---

### Networking (15-20%)

| Lab | Title | Duration | Difficulty |
|-----|-------|----------|------------|
| 01 | [Virtual Networks and Network Security Groups](networking/lab01-virtual-networks-nsg.md) | 75 min | Intermediate |
| 02 | Load Balancer and Application Gateway (Coming Soon) | 60 min | Advanced |
| 03 | Azure DNS and Private Endpoints (Coming Soon) | 45 min | Intermediate |
| 04 | VPN Gateway and ExpressRoute (Coming Soon) | 90 min | Advanced |

**Key Skills Covered:**
- ✅ Create and configure virtual networks and subnets
- ✅ Implement virtual network peering
- ✅ Configure Network Security Groups (NSGs)
- ✅ Configure Application Security Groups (ASGs)
- ✅ Implement user-defined routes (UDRs)
- ✅ Troubleshoot network connectivity
- Load balancer configuration
- Application Gateway with WAF
- Private endpoints and service endpoints

---

### Monitoring and Backup (10-15%)

| Lab | Title | Duration | Difficulty |
|-----|-------|----------|------------|
| 01 | Azure Monitor and Log Analytics (Coming Soon) | 60 min | Intermediate |
| 02 | Alerts and Action Groups (Coming Soon) | 45 min | Beginner |
| 03 | Azure Backup for VMs (Coming Soon) | 45 min | Beginner |
| 04 | Azure Site Recovery (Coming Soon) | 60 min | Advanced |

**Key Skills Covered:**
- Azure Monitor configuration
- KQL query writing
- Alert rule creation
- Action groups and notifications
- VM backup and restore
- Azure Site Recovery setup
- Backup policies and retention

---

## 🚀 Getting Started

### Prerequisites

#### Required Azure Resources
- **Azure Subscription**: Free tier or pay-as-you-go
  - [Create free account](https://azure.microsoft.com/free/)
- **Sufficient permissions**: Contributor or Owner role on subscription

#### Local Development Environment

**PowerShell:**
```powershell
# Install Azure PowerShell module
Install-Module -Name Az -Repository PSGallery -Force

# Connect to Azure
Connect-AzAccount

# Verify installation
Get-Module -Name Az -ListAvailable
```

**Azure CLI:**
```bash
# Install Azure CLI
# Windows (via PowerShell)
Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi
Start-Process msiexec.exe -ArgumentList '/I AzureCLI.msi /quiet'

# macOS
brew update && brew install azure-cli

# Linux
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Verify installation
az --version

# Login
az login
```

**Microsoft Graph PowerShell:**
```powershell
# For Entra ID labs
Install-Module Microsoft.Graph -Scope CurrentUser
Connect-MgGraph -Scopes "User.ReadWrite.All", "Group.ReadWrite.All"
```

**Additional Tools:**
- [Visual Studio Code](https://code.visualstudio.com/)
  - Azure Account extension
  - Azure PowerShell extension
  - Bicep extension
- [Azure Storage Explorer](https://azure.microsoft.com/features/storage-explorer/)
- [AzCopy](https://learn.microsoft.com/azure/storage/common/storage-use-azcopy-v10)

### Lab Conventions

**Resource Naming:**
- Resource groups: `rg-[purpose]-lab` (e.g., `rg-storage-lab`)
- Storage accounts: `st[purpose][random]` (e.g., `stcontoso12345`)
- Virtual networks: `vnet-[environment]` (e.g., `vnet-dev`)
- VMs: `vm-[purpose]-[number]` (e.g., `vm-web-01`)

**Regions:**
- Most labs use **East US** for consistency
- Some labs require multiple regions for geo-replication

**Cost Management:**
- All labs include cleanup scripts
- Use smallest VM sizes (B-series) when possible
- Delete resources immediately after completion
- Set up cost alerts on your subscription

---

## 💡 Lab Tips

### Before Starting
1. ✅ Read the entire lab scenario first
2. ✅ Check Azure pricing calculator for estimated costs
3. ✅ Set up billing alerts
4. ✅ Ensure you have required permissions
5. ✅ Take notes and screenshots for your reference

### During the Lab
1. 📝 Follow steps carefully - copy/paste commands when provided
2. 🔍 Verify each task before moving to the next
3. 🎯 Understand WHY you're doing each step, not just HOW
4. 🐛 If you encounter errors, read the error message carefully
5. 📸 Document your work with screenshots

### After Completion
1. ✅ Complete knowledge checks
2. 🧹 Run cleanup scripts to avoid charges
3. 📊 Review what you learned
4. 🔄 Repeat labs you found challenging
5. 📝 Update your study notes

---

## 🎓 Learning Paths

### Path 1: Complete Beginner (0-2 months to exam)
1. Identity & Governance → Labs 01, 02
2. Storage → Lab 01
3. Networking → Lab 01
4. Compute → Labs 01, 04
5. Monitoring → Labs 01, 02, 03
6. Review all labs and create flashcards

### Path 2: Intermediate (1 month to exam)
1. Complete all labs in order
2. Focus on PowerShell and CLI commands
3. Practice Bicep/ARM templates
4. Work through troubleshooting scenarios
5. Take practice exams

### Path 3: Advanced (2 weeks to exam)
1. Complete advanced labs (Networking 02, 04; Monitoring 04)
2. Create your own lab scenarios
3. Practice time management
4. Review scenario-based questions
5. Focus on weak areas from practice exams

---

## 📚 Additional Resources

### Official Microsoft Learn Paths
- [AZ-104: Prerequisites for Azure administrators](https://learn.microsoft.com/training/paths/az-104-administrator-prerequisites/)
- [AZ-104: Manage identities and governance](https://learn.microsoft.com/training/paths/az-104-manage-identities-governance/)
- [AZ-104: Implement and manage storage](https://learn.microsoft.com/training/paths/az-104-manage-storage/)
- [AZ-104: Deploy and manage compute resources](https://learn.microsoft.com/training/paths/az-104-manage-compute-resources/)
- [AZ-104: Configure and manage virtual networks](https://learn.microsoft.com/training/paths/az-104-manage-virtual-networks/)
- [AZ-104: Monitor and maintain Azure resources](https://learn.microsoft.com/training/paths/az-104-monitor-maintain-resources/)

### Microsoft Documentation
- [Azure PowerShell documentation](https://learn.microsoft.com/powershell/azure/)
- [Azure CLI documentation](https://learn.microsoft.com/cli/azure/)
- [Bicep documentation](https://learn.microsoft.com/azure/azure-resource-manager/bicep/)
- [ARM template reference](https://learn.microsoft.com/azure/templates/)

### Community Resources
- [Microsoft Learn Q&A](https://learn.microsoft.com/answers/topics/azure.html)
- [Azure Reddit](https://reddit.com/r/AZURE)
- [Azure Stack Overflow](https://stackoverflow.com/questions/tagged/azure)

---

## 🔧 Troubleshooting Common Issues

### Authentication Issues
```powershell
# Clear cached credentials
Clear-AzContext -Force
Disconnect-AzAccount

# Reconnect
Connect-AzAccount
```

### Permission Issues
- Ensure you have Contributor or Owner role on subscription
- Check resource provider registration
- Verify conditional access policies don't block access

### Quota Issues
- Check subscription quotas: `az vm list-usage --location eastus -o table`
- Request quota increases through Azure Portal
- Use different regions if quotas exhausted

### Resource Creation Failures
- Check naming conventions (storage accounts: lowercase, no special chars)
- Verify resource availability in selected region
- Check for policy restrictions
- Review activity logs for detailed errors

---

## 📞 Getting Help

If you encounter issues with labs:

1. **Check the FAQ** in each lab
2. **Review Azure Activity Logs** for error details
3. **Search Microsoft Learn Q&A**
4. **Post on GitHub Issues** (include error details)
5. **Join Azure study groups**

---

## ✅ Lab Completion Tracker

Use this checklist to track your progress:

### Identity & Governance
- [ ] Lab 01: Entra ID Users and Groups
- [ ] Lab 02: RBAC and Resource Access
- [ ] Lab 03: Azure Policy and Governance
- [ ] Lab 04: Cost Management
- [ ] Lab 05: Management Groups

### Storage
- [ ] Lab 01: Storage Accounts and Blob
- [ ] Lab 02: Azure Files and File Sync
- [ ] Lab 03: Storage Security

### Compute
- [ ] Lab 01: Virtual Machines
- [ ] Lab 02: VM Availability and Scaling
- [ ] Lab 03: ARM Templates and Bicep
- [ ] Lab 04: App Service
- [ ] Lab 05: Containers

### Networking
- [ ] Lab 01: Virtual Networks and NSGs
- [ ] Lab 02: Load Balancer and App Gateway
- [ ] Lab 03: DNS and Private Endpoints
- [ ] Lab 04: VPN and ExpressRoute

### Monitoring & Backup
- [ ] Lab 01: Azure Monitor
- [ ] Lab 02: Alerts and Action Groups
- [ ] Lab 03: Azure Backup
- [ ] Lab 04: Azure Site Recovery

---

## 🎯 Exam Readiness

After completing all labs, you should be able to:

✅ Create and manage Azure AD users, groups, and external users
✅ Implement RBAC at appropriate scopes
✅ Create and assign Azure Policies
✅ Configure storage accounts with proper security
✅ Manage blob lifecycle and access tiers
✅ Deploy and manage VMs with high availability
✅ Create and modify ARM templates and Bicep files
✅ Configure virtual networks with proper security
✅ Implement network connectivity solutions
✅ Set up monitoring and alerting
✅ Configure backup and disaster recovery

---

## 📈 Next Steps

1. Complete practice questions: [Scenario-Based Questions](../practice-questions/az104-scenario-questions.md)
2. Review quick reference: [AZ-104 Cheat Sheet](../quick-reference/az104-cheat-sheet.md)
3. Take practice exams: [MeasureUp](https://www.measureup.com/az-104-microsoft-azure-administrator.html)
4. Schedule your exam: [Pearson VUE](https://learn.microsoft.com/credentials/certifications/exams/az-104)

---

**Good luck with your labs and exam preparation!** 🚀
