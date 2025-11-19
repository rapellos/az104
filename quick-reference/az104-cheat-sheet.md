# AZ-104 Azure Administrator Quick Reference Guide

## Table of Contents
- [Identity & Governance](#identity--governance)
- [Storage](#storage)
- [Compute](#compute)
- [Networking](#networking)
- [Monitoring & Backup](#monitoring--backup)
- [Common Commands](#common-commands)
- [Exam Tips](#exam-tips)

---

## Identity & Governance

### Microsoft Entra ID (formerly Azure AD)

#### Key Concepts
- **Tenant**: Single instance of Azure AD
- **Directory**: Contains users, groups, and applications
- **Subscription**: Billing boundary linked to tenant

#### User Management
```powershell
# PowerShell
Connect-MgGraph
New-MgUser -DisplayName "John Doe" -MailNickname "john" -UserPrincipalName "john@domain.com" -PasswordProfile @{Password="Pass@123"}
Get-MgUser -Filter "displayName eq 'John Doe'"
Remove-MgUser -UserId "user@domain.com"
```

```bash
# Azure CLI
az ad user create --display-name "John Doe" --user-principal-name john@domain.com --password Pass@123
az ad user list --query "[].{Name:displayName, UPN:userPrincipalName}" -o table
az ad user delete --id john@domain.com
```

#### Group Management
```powershell
# Static group
New-MgGroup -DisplayName "IT-Team" -MailEnabled:$false -SecurityEnabled:$true -MailNickname "it-team"

# Dynamic group
New-MgGroup -DisplayName "IT-Dynamic" -GroupTypes "DynamicMembership" -MembershipRule "(user.department -eq `"IT`")" -MembershipRuleProcessingState "On"
```

### Azure RBAC

#### Scope Levels (Highest to Lowest)
1. Management Group
2. Subscription
3. Resource Group
4. Resource

#### Common Built-in Roles
| Role | Scope | Permissions |
|------|-------|-------------|
| Owner | All | Full access + assign roles |
| Contributor | All | Full access (no role assignments) |
| Reader | All | View only |
| User Access Administrator | IAM | Manage user access only |
| Virtual Machine Contributor | Compute | Manage VMs (not access) |
| Network Contributor | Network | Manage networks |
| Storage Account Contributor | Storage | Manage storage accounts |

#### Role Assignment Commands
```powershell
# Assign role
New-AzRoleAssignment -ObjectId <user-id> -RoleDefinitionName "Contributor" -Scope "/subscriptions/<sub-id>/resourceGroups/<rg-name>"

# List assignments
Get-AzRoleAssignment -SignInName user@domain.com

# Remove assignment
Remove-AzRoleAssignment -ObjectId <user-id> -RoleDefinitionName "Contributor" -Scope <scope>
```

```bash
# Azure CLI
az role assignment create --assignee user@domain.com --role "Contributor" --resource-group myRG
az role assignment list --assignee user@domain.com -o table
az role assignment delete --assignee user@domain.com --role "Contributor" --resource-group myRG
```

### Azure Policy

#### Policy vs Initiative
- **Policy**: Single rule (e.g., "Require tag")
- **Initiative**: Collection of policies (e.g., "Security baseline")

#### Common Built-in Policies
- Allowed locations
- Allowed resource types
- Require tag and its value
- Require SQL Server version
- Audit VMs without managed disks

```powershell
# Assign policy
New-AzPolicyAssignment -Name "require-tag" -PolicyDefinition $policy -Scope $scope

# Check compliance
Get-AzPolicyState -ResourceGroupName "myRG"
```

```bash
# Azure CLI
az policy assignment create --name "require-tag" --policy "policy-definition-id" --scope "/subscriptions/sub-id"
az policy state list --resource-group myRG
```

---

## Storage

### Storage Account Types
| Type | Use Case | Performance | Redundancy Options |
|------|----------|-------------|-------------------|
| General Purpose v2 | Most scenarios | Standard/Premium | LRS, ZRS, GRS, RA-GRS, GZRS |
| Premium Block Blobs | High transaction rates | Premium | LRS, ZRS |
| Premium File Shares | Enterprise file shares | Premium | LRS, ZRS |
| Premium Page Blobs | VHD disks | Premium | LRS |

### Redundancy Options
| Type | Description | Copies | Use Case |
|------|-------------|--------|----------|
| LRS | Locally Redundant | 3 in one datacenter | Cost-effective, low durability needs |
| ZRS | Zone Redundant | 3 across AZs | High availability within region |
| GRS | Geo Redundant | 6 across regions | Disaster recovery |
| RA-GRS | Read-Access GRS | 6 + read access | DR + read from secondary |
| GZRS | Geo-Zone Redundant | 6 across zones & regions | Highest availability |

### Blob Access Tiers
| Tier | Access | Storage Cost | Access Cost | Use Case |
|------|--------|--------------|-------------|----------|
| Hot | Frequent | Highest | Lowest | Active data |
| Cool | Infrequent | Lower | Higher | >30 days |
| Archive | Rare | Lowest | Highest | >180 days, hours to rehydrate |

### Storage Commands
```powershell
# Create storage account
New-AzStorageAccount -ResourceGroupName "myRG" -Name "mystorageacct" -Location "eastus" -SkuName "Standard_LRS"

# Create container
$ctx = (Get-AzStorageAccount -ResourceGroupName "myRG" -Name "mystorageacct").Context
New-AzStorageContainer -Name "mycontainer" -Context $ctx -Permission Off

# Upload blob
Set-AzStorageBlobContent -File "file.txt" -Container "mycontainer" -Blob "file.txt" -Context $ctx

# Generate SAS token
New-AzStorageAccountSASToken -Context $ctx -Service Blob -ResourceType Object -Permission "r" -ExpiryTime (Get-Date).AddHours(2)
```

```bash
# Azure CLI
az storage account create --name mystorageacct --resource-group myRG --location eastus --sku Standard_LRS
az storage container create --name mycontainer --account-name mystorageacct
az storage blob upload --file file.txt --container-name mycontainer --name file.txt --account-name mystorageacct
```

### AzCopy Commands
```bash
# Copy to blob storage
azcopy copy "local-path" "https://account.blob.core.windows.net/container?SAS" --recursive

# Copy between storage accounts
azcopy copy "https://source.blob.core.windows.net/container?SAS" "https://dest.blob.core.windows.net/container?SAS" --recursive

# Sync (only copies changes)
azcopy sync "local-path" "https://account.blob.core.windows.net/container?SAS"
```

---

## Compute

### VM Sizes Series
| Series | Use Case | Characteristics |
|--------|----------|-----------------|
| B-series | Burstable | Low-cost, baseline performance |
| D-series | General purpose | Balanced CPU/memory |
| E-series | Memory optimized | High memory-to-CPU ratio |
| F-series | Compute optimized | High CPU-to-memory ratio |
| M-series | Memory optimized | Largest memory offerings |
| N-series | GPU | Machine learning, graphics |

### Availability Options
| Option | SLA | Protection | Use Case |
|--------|-----|------------|----------|
| Single VM (Premium SSD) | 99.9% | None | Dev/Test |
| Availability Set | 99.95% | Rack failure | Traditional apps |
| Availability Zone | 99.99% | Datacenter failure | Mission-critical |
| VM Scale Set | 99.95%+ | Auto-scaling | Scalable workloads |

### VM Commands
```powershell
# Create VM
New-AzVm -ResourceGroupName "myRG" -Name "myVM" -Location "eastus" -Image "Win2022Datacenter" -Size "Standard_B2s"

# Start/Stop
Start-AzVM -ResourceGroupName "myRG" -Name "myVM"
Stop-AzVM -ResourceGroupName "myRG" -Name "myVM" -Force

# Resize VM
$vm = Get-AzVM -ResourceGroupName "myRG" -Name "myVM"
$vm.HardwareProfile.VmSize = "Standard_D2s_v3"
Update-AzVM -VM $vm -ResourceGroupName "myRG"

# Create VM Scale Set
New-AzVmss -ResourceGroupName "myRG" -Name "myVMSS" -Location "eastus" -VirtualNetworkName "myVnet" -SubnetName "default" -InstanceCount 2
```

```bash
# Azure CLI
az vm create --resource-group myRG --name myVM --image Win2022Datacenter --size Standard_B2s
az vm start --resource-group myRG --name myVM
az vm stop --resource-group myRG --name myVM
az vm resize --resource-group myRG --name myVM --size Standard_D2s_v3
az vmss create --resource-group myRG --name myVMSS --image UbuntuLTS --instance-count 2
```

### App Service
```powershell
# Create App Service Plan
New-AzAppServicePlan -ResourceGroupName "myRG" -Name "myPlan" -Location "eastus" -Tier "Standard" -NumberofWorkers 1 -WorkerSize "Small"

# Create Web App
New-AzWebApp -ResourceGroupName "myRG" -Name "myWebApp" -Location "eastus" -AppServicePlan "myPlan"

# Configure deployment slot
New-AzWebAppSlot -ResourceGroupName "myRG" -Name "myWebApp" -Slot "staging" -AppServicePlan "myPlan"
```

---

## Networking

### Virtual Network Address Spaces
| CIDR | Addresses | Usable Hosts | Common Use |
|------|-----------|--------------|------------|
| /8 | 16,777,216 | 16,777,214 | Large organizations |
| /16 | 65,536 | 65,534 | VNet (default) |
| /24 | 256 | 251 | Subnet |
| /27 | 32 | 27 | GatewaySubnet (minimum) |
| /26 | 64 | 59 | AzureFirewallSubnet, Bastion |

### Reserved Azure IPs (per subnet)
- **.0**: Network address
- **.1**: Azure gateway
- **.2-.3**: Azure DNS
- **.255**: Broadcast

### Network Security Groups

#### Default Rules (Cannot delete)
**Inbound:**
- AllowVNetInBound (Priority 65000)
- AllowAzureLoadBalancerInBound (Priority 65001)
- DenyAllInBound (Priority 65500)

**Outbound:**
- AllowVNetOutBound (Priority 65000)
- AllowInternetOutBound (Priority 65001)
- DenyAllOutBound (Priority 65500)

#### Service Tags (Common)
- `VirtualNetwork`: Your VNet address space
- `Internet`: Public internet
- `AzureLoadBalancer`: Azure load balancer
- `Storage`: Azure Storage
- `Sql`: Azure SQL Database
- `AzureActiveDirectory`: Azure AD

### Networking Commands
```powershell
# Create VNet
New-AzVirtualNetwork -Name "myVnet" -ResourceGroupName "myRG" -Location "eastus" -AddressPrefix "10.0.0.0/16"

# Add subnet
$vnet = Get-AzVirtualNetwork -Name "myVnet" -ResourceGroupName "myRG"
Add-AzVirtualNetworkSubnetConfig -Name "mySubnet" -VirtualNetwork $vnet -AddressPrefix "10.0.1.0/24"
$vnet | Set-AzVirtualNetwork

# Create NSG
New-AzNetworkSecurityGroup -Name "myNSG" -ResourceGroupName "myRG" -Location "eastus"

# Add NSG rule
$nsg = Get-AzNetworkSecurityGroup -Name "myNSG" -ResourceGroupName "myRG"
Add-AzNetworkSecurityRuleConfig -Name "AllowHTTP" -NetworkSecurityGroup $nsg -Priority 100 -Direction Inbound -Access Allow -Protocol Tcp -SourceAddressPrefix "*" -SourcePortRange "*" -DestinationAddressPrefix "*" -DestinationPortRange 80
$nsg | Set-AzNetworkSecurityGroup

# VNet Peering
Add-AzVirtualNetworkPeering -Name "vnet1-to-vnet2" -VirtualNetwork $vnet1 -RemoteVirtualNetworkId $vnet2.Id
```

```bash
# Azure CLI
az network vnet create --name myVnet --resource-group myRG --address-prefix 10.0.0.0/16
az network vnet subnet create --vnet-name myVnet --name mySubnet --resource-group myRG --address-prefix 10.0.1.0/24
az network nsg create --name myNSG --resource-group myRG
az network nsg rule create --nsg-name myNSG --name AllowHTTP --priority 100 --direction Inbound --access Allow --protocol Tcp --destination-port-ranges 80
az network vnet peering create --name vnet1-to-vnet2 --resource-group myRG --vnet-name vnet1 --remote-vnet vnet2 --allow-vnet-access
```

### Load Balancer vs Application Gateway
| Feature | Load Balancer | Application Gateway |
|---------|---------------|---------------------|
| Layer | Layer 4 (Transport) | Layer 7 (Application) |
| Protocol | TCP/UDP | HTTP/HTTPS |
| Routing | IP/Port | URL path, host headers |
| SSL Offload | No | Yes |
| WAF | No | Yes (with WAF SKU) |
| Use Case | General purpose | Web applications |

---

## Monitoring & Backup

### Azure Monitor Components
- **Metrics**: Numerical values (CPU%, memory)
- **Logs**: Text records (application logs, events)
- **Alerts**: Notifications based on conditions
- **Application Insights**: APM for applications
- **Log Analytics**: Query and analyze logs

### Common KQL Queries
```kusto
// VM Performance - High CPU
Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| where CounterValue > 80
| summarize avg(CounterValue) by Computer, bin(TimeGenerated, 5m)

// Failed sign-ins
SigninLogs
| where ResultType != 0
| summarize Count=count() by UserPrincipalName, ResultDescription

// Storage account operations
StorageBlobLogs
| where StatusCode == 403
| summarize Count=count() by AccountName, OperationName

// Resource changes
AzureActivity
| where OperationNameValue contains "write"
| project TimeGenerated, Caller, OperationNameValue, ResourceGroup
```

### Monitoring Commands
```powershell
# Create alert rule
$condition = New-AzMetricAlertRuleV2Criteria -MetricName "Percentage CPU" -Operator GreaterThan -Threshold 80
Add-AzMetricAlertRuleV2 -Name "High CPU" -ResourceGroupName "myRG" -TargetResourceId $vmId -Condition $condition -WindowSize 00:05:00

# Query logs
Invoke-AzOperationalInsightsQuery -WorkspaceId $workspaceId -Query "Perf | where CounterName == '% Processor Time' | top 10 by CounterValue"

# Enable diagnostics
Set-AzDiagnosticSetting -ResourceId $vmId -WorkspaceId $workspaceId -Enabled $true
```

### Backup & Recovery

#### Backup Frequency Options
- **Azure VM**: Daily or more frequent
- **SQL in Azure VM**: Every 15 minutes (log backups)
- **Azure Files**: Daily
- **Azure Blobs**: Continuous (operational backup)

#### Retention
- **Standard**: Up to 9999 days
- **Long-term**: Up to 10 years (SQL)

```powershell
# Create Recovery Services Vault
New-AzRecoveryServicesVault -Name "myVault" -ResourceGroupName "myRG" -Location "eastus"

# Set vault context
$vault = Get-AzRecoveryServicesVault -Name "myVault"
Set-AzRecoveryServicesVaultContext -Vault $vault

# Enable backup for VM
$policy = Get-AzRecoveryServicesBackupProtectionPolicy -Name "DefaultPolicy"
Enable-AzRecoveryServicesBackupProtection -ResourceGroupName "myRG" -Name "myVM" -Policy $policy

# Backup now
$container = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVM -FriendlyName "myVM"
$item = Get-AzRecoveryServicesBackupItem -Container $container -WorkloadType AzureVM
Backup-AzRecoveryServicesBackupItem -Item $item
```

---

## Common Commands

### Resource Groups
```powershell
# PowerShell
New-AzResourceGroup -Name "myRG" -Location "eastus"
Get-AzResourceGroup
Remove-AzResourceGroup -Name "myRG" -Force
```

```bash
# Azure CLI
az group create --name myRG --location eastus
az group list -o table
az group delete --name myRG --yes --no-wait
```

### Tags
```powershell
# PowerShell
Set-AzResource -ResourceGroupName "myRG" -Name "myVM" -ResourceType "Microsoft.Compute/virtualMachines" -Tag @{Environment="Production"; CostCenter="IT"}
Get-AzResource -Tag @{Environment="Production"}
```

```bash
# Azure CLI
az resource tag --tags Environment=Production CostCenter=IT --resource-group myRG --name myVM --resource-type Microsoft.Compute/virtualMachines
az resource list --tag Environment=Production
```

### Resource Locks
```powershell
# Create lock
New-AzResourceLock -LockName "DoNotDelete" -LockLevel CanNotDelete -ResourceGroupName "myRG"

# Remove lock
Remove-AzResourceLock -LockName "DoNotDelete" -ResourceGroupName "myRG" -Force
```

```bash
# Azure CLI
az lock create --name DoNotDelete --lock-type CanNotDelete --resource-group myRG
az lock delete --name DoNotDelete --resource-group myRG
```

---

## Exam Tips

### Time Management
- **Duration**: 150-180 minutes (with unscored items)
- **Questions**: ~40-60 questions
- **Passing Score**: 700/1000
- **Question Types**:
  - Multiple choice
  - Multiple response
  - Drag and drop
  - Build list
  - Case studies

### Question Distribution (Approximate)
1. Identity & Governance: 20-25%
2. Storage: 15-20%
3. Compute: 20-25%
4. Networking: 15-20%
5. Monitoring & Backup: 10-15%

### Key Study Areas
1. **RBAC**: Understand scopes, built-in roles, custom roles
2. **NSGs**: Priority, rules, service tags, ASGs
3. **Storage**: Redundancy options, access tiers, SAS tokens
4. **VNet Peering**: Transitive routing, gateway transit
5. **VM Availability**: Sets vs Zones, SLA differences
6. **KQL Queries**: Be comfortable reading and understanding queries
7. **Bicep/ARM**: Read and modify templates
8. **Azure Policy**: Policy vs Initiative, compliance

### Command Syntax Comparison
| Task | PowerShell | Azure CLI |
|------|-----------|-----------|
| Create resource | `New-Az[Resource]` | `az [resource] create` |
| List resources | `Get-Az[Resource]` | `az [resource] list` |
| Update | `Update-Az[Resource]` | `az [resource] update` |
| Delete | `Remove-Az[Resource]` | `az [resource] delete` |
| Show details | `Get-Az[Resource]` | `az [resource] show` |

### Portal Navigation Tips
- Use **Resource Groups** view for managing related resources
- Use **All Resources** for global search
- Bookmark **Cost Management** for budget monitoring
- Use **Resource Graph Explorer** for complex queries

### Common Pitfalls to Avoid
1. ❌ Not waiting for peering to complete (check "Connected" status)
2. ❌ Forgetting that NSG rules are stateful
3. ❌ Not understanding RBAC inheritance from parent scopes
4. ❌ Confusing service endpoints with private endpoints
5. ❌ Not knowing VM states that incur charges (stopped vs deallocated)
6. ❌ Forgetting Azure reserves first 4 IPs and last IP in each subnet
7. ❌ Not understanding that Cool/Archive tiers have minimum retention periods
8. ❌ Forgetting to configure soft delete retention (new exam requirement)

### Before the Exam
- [ ] Review all 5 skill areas
- [ ] Practice with hands-on labs
- [ ] Take practice exams (MeasureUp, Microsoft)
- [ ] Review this cheat sheet
- [ ] Understand JSON/Bicep syntax
- [ ] Know KQL basics
- [ ] Test your environment (for online exams)

### During the Exam
- ✓ Read questions carefully (key words: "LEAST effort", "most cost-effective", "highest availability")
- ✓ Mark questions for review
- ✓ Eliminate obviously wrong answers
- ✓ Watch for "Select all that apply" questions
- ✓ Case studies: Read scenario first, then questions
- ✓ Manage your time (don't spend too long on one question)

---

## Quick Links
- [Azure Documentation](https://docs.microsoft.com/azure)
- [Azure Pricing Calculator](https://azure.microsoft.com/pricing/calculator/)
- [Azure SLA Summary](https://azure.microsoft.com/support/legal/sla/)
- [Azure Portal](https://portal.azure.com)
- [Azure CLI Reference](https://learn.microsoft.com/cli/azure/)
- [PowerShell Reference](https://learn.microsoft.com/powershell/azure/)
- [Bicep Documentation](https://learn.microsoft.com/azure/azure-resource-manager/bicep/)

---

**Good luck on your AZ-104 exam!** 🎯
