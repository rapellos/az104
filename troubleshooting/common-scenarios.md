# AZ-104 Troubleshooting Scenarios

## Introduction
This guide covers common troubleshooting scenarios you'll encounter as an Azure Administrator and on the AZ-104 exam. Each scenario includes symptoms, diagnosis steps, and solutions.

---

## Table of Contents
- [Identity & Access Issues](#identity--access-issues)
- [Storage Problems](#storage-problems)
- [VM and Compute Issues](#vm-and-compute-issues)
- [Networking Troubleshooting](#networking-troubleshooting)
- [Monitoring & Alerting](#monitoring--alerting)
- [Cost and Billing](#cost-and-billing)

---

## Identity & Access Issues

### Scenario 1: User Cannot Access Azure Portal

**Symptoms:**
- User reports "Access Denied" when logging into Azure Portal
- User exists in Azure AD but can't see any resources

**Diagnosis:**
```powershell
# Check if user exists
Get-MgUser -Filter "userPrincipalName eq 'user@domain.com'"

# Check role assignments
Get-AzRoleAssignment -SignInName "user@domain.com"

# Check group memberships
Get-MgUserMemberOf -UserId "user@domain.com"
```

**Common Causes:**
1. No RBAC role assigned
2. Conditional Access policy blocking access
3. MFA not configured
4. Guest user not accepted invitation

**Solutions:**
```powershell
# Assign Reader role at subscription level
$user = Get-MgUser -Filter "userPrincipalName eq 'user@domain.com'"
New-AzRoleAssignment -ObjectId $user.Id -RoleDefinitionName "Reader" -Scope "/subscriptions/<subscription-id>"

# Check conditional access
Get-MgIdentityConditionalAccessPolicy | Where-Object {$_.State -eq "enabled"}

# Resend B2B invitation
New-MgInvitation -InvitedUserEmailAddress "user@external.com" -SendInvitationMessage:$true -InviteRedirectUrl "https://myapps.microsoft.com"
```

---

### Scenario 2: "Insufficient Privileges" Error

**Symptoms:**
- Error when trying to create/modify resources
- User has Contributor role but can't perform certain actions

**Diagnosis:**
```powershell
# Check effective permissions
Get-AzRoleAssignment -SignInName "user@domain.com" | Format-Table RoleDefinitionName, Scope

# Check for deny assignments
Get-AzDenyAssignment -Scope "/subscriptions/<subscription-id>/resourceGroups/myRG"

# Check Azure Policy
Get-AzPolicyState -ResourceGroupName "myRG" -Filter "ComplianceState eq 'NonCompliant'"
```

**Common Causes:**
1. RBAC role at wrong scope
2. Azure Policy denying action
3. Resource lock preventing changes
4. Management group policy inheritance

**Solutions:**
```powershell
# Check and remove resource lock
Get-AzResourceLock -ResourceGroupName "myRG"
Remove-AzResourceLock -LockId <lock-id> -Force

# Check policy compliance
Get-AzPolicyAssignment -Scope "/subscriptions/<subscription-id>" | Where-Object {$_.Properties.policyDefinitionId -like "*deny*"}

# Verify correct scope
New-AzRoleAssignment -ObjectId $user.Id -RoleDefinitionName "Contributor" -ResourceGroupName "myRG"
```

---

### Scenario 3: Service Principal Authentication Failure

**Symptoms:**
- Automated scripts failing with authentication errors
- "Invalid client secret" or "Application not found" errors

**Diagnosis:**
```powershell
# Check service principal
Get-AzADServicePrincipal -DisplayName "MyApp"

# Check application registration
Get-AzADApplication -DisplayName "MyApp"

# Test authentication
$securePassword = ConvertTo-SecureString -String "secret" -AsPlainText -Force
$credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList "app-id", $securePassword
Connect-AzAccount -ServicePrincipal -Credential $credential -Tenant "tenant-id"
```

**Common Causes:**
1. Expired client secret
2. Service principal deleted
3. Incorrect tenant ID
4. Missing role assignments

**Solutions:**
```powershell
# Create new client secret
$app = Get-AzADApplication -DisplayName "MyApp"
$endDate = (Get-Date).AddYears(1)
New-AzADAppCredential -ApplicationId $app.AppId -EndDate $endDate

# Assign role to service principal
$sp = Get-AzADServicePrincipal -DisplayName "MyApp"
New-AzRoleAssignment -ObjectId $sp.Id -RoleDefinitionName "Contributor" -ResourceGroupName "myRG"
```

---

## Storage Problems

### Scenario 4: Cannot Access Storage Account

**Symptoms:**
- "403 Forbidden" errors when accessing blobs
- "This request is not authorized" messages

**Diagnosis:**
```powershell
# Check storage account
$storage = Get-AzStorageAccount -ResourceGroupName "myRG" -Name "mystorageacct"

# Check network rules
$storage.NetworkRuleSet

# Check firewall settings
Get-AzStorageAccountNetworkRuleSet -ResourceGroupName "myRG" -AccountName "mystorageacct"

# Check container permissions
$ctx = $storage.Context
Get-AzStorageContainer -Context $ctx | Select-Object Name, PublicAccess
```

**Common Causes:**
1. Storage firewall blocking IP
2. Incorrect SAS token
3. Expired access policy
4. Virtual network rules misconfigured

**Solutions:**
```powershell
# Add IP to firewall
$myIP = (Invoke-WebRequest -Uri "https://api.ipify.org").Content
Add-AzStorageAccountNetworkRule -ResourceGroupName "myRG" -AccountName "mystorageacct" -IPAddressOrRange $myIP

# Generate new SAS token
$sasToken = New-AzStorageAccountSASToken -Context $ctx -Service Blob -ResourceType Object,Container -Permission "rl" -ExpiryTime (Get-Date).AddHours(2)

# Allow Azure services
Update-AzStorageAccountNetworkRuleSet -ResourceGroupName "myRG" -Name "mystorageacct" -Bypass AzureServices

# Check and update access keys
$keys = Get-AzStorageAccountKey -ResourceGroupName "myRG" -Name "mystorageacct"
```

---

### Scenario 5: Slow Blob Upload/Download

**Symptoms:**
- Uploads taking much longer than expected
- Timeouts during large file transfers

**Diagnosis:**
```powershell
# Check storage account metrics
$storageAccount = Get-AzStorageAccount -ResourceGroupName "myRG" -Name "mystorageacct"

# View metrics
$endTime = Get-Date
$startTime = $endTime.AddHours(-1)
Get-AzMetric -ResourceId $storageAccount.Id -MetricName "Transactions","Ingress","Egress" -StartTime $startTime -EndTime $endTime -TimeGrain 00:01:00

# Check throttling
Get-AzMetric -ResourceId $storageAccount.Id -MetricName "Transactions" -StartTime $startTime -EndTime $endTime | Where-Object {$_.Data.ResponseType -eq "ClientThrottlingError"}
```

**Common Causes:**
1. Network bandwidth limitations
2. Storage account throttling
3. Using LRS instead of ZRS/GRS
4. Not using AzCopy for large transfers

**Solutions:**
```bash
# Use AzCopy with optimization
azcopy copy "source" "destination?sas" --block-size-mb 8 --cap-mbps 500

# Enable large file shares (Premium)
az storage account update --name mystorageacct --resource-group myRG --enable-large-file-share

# Upgrade to Premium storage for high IOPS
az storage account update --name mystorageacct --resource-group myRG --sku Premium_LRS
```

---

### Scenario 6: Deleted Blob Recovery

**Symptoms:**
- Important blob accidentally deleted
- Need to recover data from archive tier

**Diagnosis:**
```powershell
# Check soft delete settings
$blobService = Get-AzStorageBlobServiceProperty -ResourceGroupName "myRG" -StorageAccountName "mystorageacct"
$blobService.DeleteRetentionPolicy

# List deleted blobs
$ctx = (Get-AzStorageAccount -ResourceGroupName "myRG" -Name "mystorageacct").Context
Get-AzStorageBlob -Container "mycontainer" -Context $ctx -IncludeDeleted
```

**Solutions:**
```powershell
# Restore soft-deleted blob
$deletedBlob = Get-AzStorageBlob -Container "mycontainer" -Blob "deletedfile.txt" -Context $ctx -IncludeDeleted
$deletedBlob.ICloudBlob.Undelete()

# Rehydrate from archive
$blob = Get-AzStorageBlob -Container "mycontainer" -Blob "archivedfile.txt" -Context $ctx
$blob.ICloudBlob.SetStandardBlobTier("Hot", "High")

# Enable soft delete if not configured
Enable-AzStorageBlobDeleteRetentionPolicy -ResourceGroupName "myRG" -StorageAccountName "mystorageacct" -RetentionDays 14

# Enable container soft delete (NEW)
Enable-AzStorageContainerDeleteRetentionPolicy -ResourceGroupName "myRG" -StorageAccountName "mystorageacct" -RetentionDays 14
```

---

## VM and Compute Issues

### Scenario 7: VM Won't Start

**Symptoms:**
- VM stuck in "Starting" state
- Boot diagnostics show errors

**Diagnosis:**
```powershell
# Check VM status
Get-AzVM -ResourceGroupName "myRG" -Name "myVM" -Status

# View boot diagnostics
$vm = Get-AzVM -ResourceGroupName "myRG" -Name "myVM"
Get-AzVMBootDiagnosticsData -ResourceGroupName "myRG" -Name "myVM" -Windows

# Check activity log
Get-AzLog -ResourceGroupName "myRG" -StartTime (Get-Date).AddHours(-1) | Where-Object {$_.ResourceId -like "*myVM*"}

# Check VM allocation
Get-AzVM -ResourceGroupName "myRG" -Name "myVM" | Select-Object -ExpandProperty HardwareProfile
```

**Common Causes:**
1. Allocation failure (capacity issues)
2. Corrupted OS disk
3. Quota exceeded
4. Failed extension

**Solutions:**
```powershell
# Deallocate and restart
Stop-AzVM -ResourceGroupName "myRG" -Name "myVM" -Force
Start-AzVM -ResourceGroupName "myRG" -Name "myVM"

# Move to different availability zone
Stop-AzVM -ResourceGroupName "myRG" -Name "myVM" -Force
# Change zone in configuration
$vm.Zones = @("2")
Update-AzVM -ResourceGroupName "myRG" -VM $vm

# Check quotas
Get-AzVMUsage -Location "eastus" | Where-Object {$_.Name.Value -like "*StandardDSv3*"}

# Remove failed extension
Remove-AzVMExtension -ResourceGroupName "myRG" -VMName "myVM" -Name "extensionName" -Force
```

---

### Scenario 8: Cannot RDP/SSH to VM

**Symptoms:**
- Connection timeout when trying to connect
- "No route to host" errors

**Diagnosis:**
```powershell
# Check VM running
Get-AzVM -ResourceGroupName "myRG" -Name "myVM" -Status

# Check NSG rules
$nic = Get-AzNetworkInterface -ResourceGroupName "myRG" | Where-Object {$_.VirtualMachine.Id -like "*myVM*"}
$nsg = Get-AzNetworkSecurityGroup -ResourceGroupName "myRG" | Where-Object {$_.NetworkInterfaces.Id -contains $nic.Id}
$nsg.SecurityRules | Where-Object {$_.DestinationPortRange -contains "3389" -or $_.DestinationPortRange -contains "22"}

# Check effective security rules
Get-AzEffectiveNetworkSecurityGroup -NetworkInterfaceName $nic.Name -ResourceGroupName "myRG"

# Check public IP
Get-AzPublicIpAddress -ResourceGroupName "myRG" | Where-Object {$_.IpConfiguration.Id -eq $nic.IpConfigurations[0].Id}

# Use Connection Troubleshoot
$nw = Get-AzNetworkWatcher -ResourceGroupName "NetworkWatcherRG" -Name "NetworkWatcher_eastus"
Test-AzNetworkWatcherConnectivity -NetworkWatcher $nw -SourceId $vm.Id -DestinationPort 3389 -DestinationAddress "1.2.3.4"
```

**Common Causes:**
1. NSG blocking RDP/SSH port
2. No public IP assigned
3. Windows Firewall blocking connection
4. VM not running

**Solutions:**
```powershell
# Add NSG rule for RDP
$nsg = Get-AzNetworkSecurityGroup -ResourceGroupName "myRG" -Name "myNSG"
Add-AzNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg -Name "AllowRDP" -Priority 100 -Direction Inbound -Access Allow -Protocol Tcp -SourceAddressPrefix "Internet" -SourcePortRange "*" -DestinationAddressPrefix "*" -DestinationPortRange 3389
Set-AzNetworkSecurityGroup -NetworkSecurityGroup $nsg

# Create and assign public IP
$pip = New-AzPublicIpAddress -Name "myVM-pip" -ResourceGroupName "myRG" -Location "eastus" -AllocationMethod Static
$nic.IpConfigurations[0].PublicIpAddress = $pip
Set-AzNetworkInterface -NetworkInterface $nic

# Reset VM credentials
Set-AzVMAccessExtension -ResourceGroupName "myRG" -VMName "myVM" -Name "VMAccessAgent" -UserName "newadmin" -Password "NewPassword123!"

# Use Azure Bastion (more secure)
New-AzBastion -ResourceGroupName "myRG" -Name "myBastion" -VirtualNetwork $vnet -PublicIpAddress $bastionPip
```

---

### Scenario 9: VM Performance Issues

**Symptoms:**
- High CPU usage alerts
- Slow application response
- Out of memory errors

**Diagnosis:**
```powershell
# Check VM metrics
$vm = Get-AzVM -ResourceGroupName "myRG" -Name "myVM"
$endTime = Get-Date
$startTime = $endTime.AddHours(-1)

Get-AzMetric -ResourceId $vm.Id -MetricName "Percentage CPU" -StartTime $startTime -EndTime $endTime -TimeGrain 00:01:00

# Query Log Analytics
$query = @"
Perf
| where Computer == "myVM"
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| where TimeGenerated > ago(1h)
| summarize avg(CounterValue) by bin(TimeGenerated, 5m)
| render timechart
"@

Invoke-AzOperationalInsightsQuery -WorkspaceId $workspaceId -Query $query

# Check disk performance
Get-AzMetric -ResourceId $vm.Id -MetricName "OS Disk IOPS Consumed Percentage" -StartTime $startTime -EndTime $endTime
```

**Solutions:**
```powershell
# Resize VM to larger size
Stop-AzVM -ResourceGroupName "myRG" -Name "myVM" -Force
$vm.HardwareProfile.VmSize = "Standard_D4s_v3"
Update-AzVM -ResourceGroupName "myRG" -VM $vm
Start-AzVM -ResourceGroupName "myRG" -Name "myVM"

# Add more disks or increase disk size
$diskConfig = New-AzDiskConfig -Location "eastus" -CreateOption Empty -DiskSizeGB 256 -SkuName Premium_LRS
$dataDisk = New-AzDisk -ResourceGroupName "myRG" -DiskName "datadisk02" -Disk $diskConfig
Add-AzVMDataDisk -VM $vm -Name "datadisk02" -CreateOption Attach -ManagedDiskId $dataDisk.Id -Lun 1
Update-AzVM -ResourceGroupName "myRG" -VM $vm

# Enable VM insights
Set-AzVMExtension -ResourceGroupName "myRG" -VMName "myVM" -Name "AzureMonitorWindowsAgent" -Publisher "Microsoft.Azure.Monitor" -ExtensionType "AzureMonitorWindowsAgent" -TypeHandlerVersion "1.0"
```

---

## Networking Troubleshooting

### Scenario 10: VNet Peering Not Working

**Symptoms:**
- Resources in peered VNets cannot communicate
- Peering shows "Connected" status but traffic fails

**Diagnosis:**
```powershell
# Check peering status
Get-AzVirtualNetworkPeering -VirtualNetworkName "vnet1" -ResourceGroupName "myRG"

# Check for transitive routing issue
Get-AzVirtualNetworkPeering -VirtualNetworkName "vnet2" -ResourceGroupName "myRG" | Select-Object Name, PeeringState, AllowForwardedTraffic, UseRemoteGateways

# Check route tables
$subnet = Get-AzVirtualNetwork -Name "vnet1" -ResourceGroupName "myRG" | Get-AzVirtualNetworkSubnetConfig -Name "subnet1"
Get-AzRouteTable -ResourceGroupName "myRG" | Where-Object {$_.Subnets.Id -contains $subnet.Id}

# Check effective routes on NIC
Get-AzEffectiveRouteTable -NetworkInterfaceName "vm-nic" -ResourceGroupName "myRG"
```

**Common Causes:**
1. AllowForwardedTraffic not enabled
2. NSG blocking traffic
3. User-defined routes overriding system routes
4. Trying to create transitive peering (spokes talking directly)

**Solutions:**
```powershell
# Update peering to allow forwarded traffic
$peering = Get-AzVirtualNetworkPeering -Name "vnet1-to-vnet2" -VirtualNetworkName "vnet1" -ResourceGroupName "myRG"
$peering.AllowForwardedTraffic = $true
Set-AzVirtualNetworkPeering -VirtualNetworkPeering $peering

# For hub-spoke, enable gateway transit
$hubPeering = Get-AzVirtualNetworkPeering -Name "hub-to-spoke1" -VirtualNetworkName "vnet-hub" -ResourceGroupName "myRG"
$hubPeering.AllowGatewayTransit = $true
Set-AzVirtualNetworkPeering -VirtualNetworkPeering $hubPeering

$spokePeering = Get-AzVirtualNetworkPeering -Name "spoke1-to-hub" -VirtualNetworkName "vnet-spoke1" -ResourceGroupName "myRG"
$spokePeering.UseRemoteGateways = $true
Set-AzVirtualNetworkPeering -VirtualNetworkPeering $spokePeering

# Check NSG rules between VNets
Test-AzNetworkWatcherIPFlow -NetworkWatcher $nw -TargetVirtualMachineId $vmId -Direction Outbound -Protocol TCP -LocalIPAddress "10.0.1.4" -LocalPort 80 -RemoteIPAddress "10.1.1.4" -RemotePort 80
```

---

### Scenario 11: DNS Resolution Failures

**Symptoms:**
- Cannot resolve Azure private DNS names
- Custom DNS not working

**Diagnosis:**
```powershell
# Check VNet DNS settings
Get-AzVirtualNetwork -Name "myVnet" -ResourceGroupName "myRG" | Select-Object -ExpandProperty DhcpOptions

# Check private DNS zone links
Get-AzPrivateDnsVirtualNetworkLink -ResourceGroupName "myRG" -ZoneName "privatelink.database.windows.net"

# Test DNS from VM
# From within VM
nslookup myserver.privatelink.database.windows.net
```

**Solutions:**
```powershell
# Configure custom DNS
$vnet = Get-AzVirtualNetwork -Name "myVnet" -ResourceGroupName "myRG"
$vnet.DhcpOptions.DnsServers = @("10.0.0.4", "10.0.0.5")
Set-AzVirtualNetwork -VirtualNetwork $vnet

# Link private DNS zone to VNet
New-AzPrivateDnsVirtualNetworkLink -ResourceGroupName "myRG" -ZoneName "privatelink.database.windows.net" -Name "myVnetLink" -VirtualNetworkId $vnet.Id -EnableRegistration
```

---

### Scenario 12: Load Balancer Health Probe Failing

**Symptoms:**
- Backend pool instances showing unhealthy
- Traffic not distributing to all instances

**Diagnosis:**
```powershell
# Check load balancer backend health
$lb = Get-AzLoadBalancer -ResourceGroupName "myRG" -Name "myLB"
$probe = Get-AzLoadBalancerProbeConfig -LoadBalancer $lb
$backendHealth = Get-AzNetworkInterfaceLoadBalancerBackendAddressPool -ResourceGroupName "myRG"

# Check probe configuration
$probe | Select-Object Name, Protocol, Port, IntervalInSeconds, NumberOfProbes

# Check NSG rules allowing probe traffic
# Health probes come from 168.63.129.16
Get-AzNetworkSecurityGroup -ResourceGroupName "myRG" | Get-AzNetworkSecurityRuleConfig | Where-Object {$_.SourceAddressPrefix -contains "168.63.129.16"}
```

**Solutions:**
```powershell
# Allow health probe traffic in NSG
$nsg = Get-AzNetworkSecurityGroup -Name "myNSG" -ResourceGroupName "myRG"
Add-AzNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg -Name "AllowAzureLoadBalancerInBound" -Priority 100 -Direction Inbound -Access Allow -Protocol "*" -SourceAddressPrefix "AzureLoadBalancer" -SourcePortRange "*" -DestinationAddressPrefix "*" -DestinationPortRange "80"
Set-AzNetworkSecurityGroup -NetworkSecurityGroup $nsg

# Update probe configuration
Set-AzLoadBalancerProbeConfig -LoadBalancer $lb -Name "httpProbe" -Protocol Http -Port 80 -RequestPath "/health" -IntervalInSeconds 15 -ProbeCount 2
Set-AzLoadBalancer -LoadBalancer $lb

# Verify application is listening on probe port
# From VM: netstat -an | findstr :80
```

---

## Monitoring & Alerting

### Scenario 13: Missing Metrics Data

**Symptoms:**
- Metrics not showing in Azure Monitor
- Gaps in historical data

**Diagnosis:**
```powershell
# Check diagnostic settings
Get-AzDiagnosticSetting -ResourceId $resourceId

# Verify metrics are being collected
Get-AzMetric -ResourceId $resourceId -MetricName "Percentage CPU" -StartTime (Get-Date).AddHours(-1) -EndTime (Get-Date)

# Check Log Analytics workspace
Get-AzOperationalInsightsWorkspace -ResourceGroupName "myRG"
```

**Solutions:**
```powershell
# Enable diagnostic settings
$workspaceId = (Get-AzOperationalInsightsWorkspace -ResourceGroupName "myRG" -Name "myWorkspace").ResourceId

Set-AzDiagnosticSetting -ResourceId $resourceId -Name "toLogAnalytics" -WorkspaceId $workspaceId -Enabled $true -MetricCategory AllMetrics

# For VMs, install monitoring agent
Set-AzVMExtension -ResourceGroupName "myRG" -VMName "myVM" -Name "AzureMonitorWindowsAgent" -Publisher "Microsoft.Azure.Monitor" -ExtensionType "AzureMonitorWindowsAgent" -TypeHandlerVersion "1.0"
```

---

### Scenario 14: Alerts Not Firing

**Symptoms:**
- Alert rule configured but no notifications received
- Condition met but alert not triggered

**Diagnosis:**
```powershell
# Check alert rule configuration
Get-AzMetricAlertRuleV2 -ResourceGroupName "myRG" -Name "High CPU Alert"

# Check action group
Get-AzActionGroup -ResourceGroupName "myRG"

# Check alert history
Get-AzAlertHistory -ResourceGroupName "myRG"

# Verify alert processing rules (suppression)
Get-AzAlertProcessingRule -ResourceGroupName "myRG"
```

**Solutions:**
```powershell
# Update alert rule
$condition = New-AzMetricAlertRuleV2Criteria -MetricName "Percentage CPU" -Operator GreaterThan -Threshold 80 -TimeAggregation Average
$actionGroup = Get-AzActionGroup -ResourceGroupName "myRG" -Name "myActionGroup"

Add-AzMetricAlertRuleV2 -Name "High CPU Alert" -ResourceGroupName "myRG" -TargetResourceId $vm.Id -Condition $condition -ActionGroupId $actionGroup.Id -WindowSize (New-TimeSpan -Minutes 5) -Frequency (New-TimeSpan -Minutes 1) -Severity 2

# Test action group
Test-AzActionGroup -ActionGroupResourceId $actionGroup.Id -ResourceGroupName "myRG"

# Check action group receivers
$actionGroup.EmailReceivers | Format-Table Name, EmailAddress, Status
```

---

## Cost and Billing

### Scenario 15: Unexpected High Costs

**Symptoms:**
- Azure bill higher than expected
- Need to identify cost drivers

**Diagnosis:**
```powershell
# Get cost analysis
$endDate = Get-Date
$startDate = $endDate.AddDays(-30)

# View costs by resource group
Get-AzConsumptionUsageDetail -StartDate $startDate -EndDate $endDate | Group-Object ResourceGroupName | Select-Object Name, @{Name="Cost";Expression={($_.Group | Measure-Object PretaxCost -Sum).Sum}}

# View costs by resource type
Get-AzConsumptionUsageDetail -StartDate $startDate -EndDate $endDate | Group-Object ConsumedService | Select-Object Name, @{Name="Cost";Expression={($_.Group | Measure-Object PretaxCost -Sum).Sum}} | Sort-Object Cost -Descending

# Check for running resources
Get-AzVM -Status | Where-Object {$_.PowerState -eq "VM running"} | Select-Object Name, ResourceGroupName, HardwareProfile

# Check unused resources
Get-AzPublicIpAddress | Where-Object {$_.IpConfiguration -eq $null}
Get-AzDisk | Where-Object {$_.ManagedBy -eq $null}
```

**Solutions:**
```powershell
# Stop deallocated VMs (deallocate to avoid charges)
Get-AzVM -Status | Where-Object {$_.PowerState -eq "VM running"} | ForEach-Object {
    Stop-AzVM -ResourceGroupName $_.ResourceGroupName -Name $_.Name -Force
}

# Delete unused resources
Get-AzPublicIpAddress | Where-Object {$_.IpConfiguration -eq $null} | Remove-AzPublicIpAddress -Force

# Set up budget
New-AzConsumptionBudget -Name "MonthlyBudget" -Amount 1000 -Category Cost -TimeGrain Monthly -StartDate (Get-Date) -EndDate (Get-Date).AddYears(1)

# Use tags for cost tracking
Set-AzResource -ResourceId $resourceId -Tag @{Environment="Production"; CostCenter="IT"; Owner="admin@company.com"}

# Use Reserved Instances for predictable workloads
# Purchase through Azure Portal
```

---

## Quick Troubleshooting Commands

### PowerShell
```powershell
# Get recent errors
Get-AzLog -Status Failed -StartTime (Get-Date).AddHours(-24)

# Test network connectivity
Test-NetConnection -ComputerName "server.com" -Port 443

# Check resource health
Get-AzResourceHealth -ResourceId $resourceId

# View all running resources
Get-AzResource | Where-Object {$_.ResourceType -like "Microsoft.Compute/virtualMachines"} | ForEach-Object {
    Get-AzVM -ResourceGroupName $_.ResourceGroupName -Name $_.Name -Status
}
```

### Azure CLI
```bash
# Activity log errors
az monitor activity-log list --status Failed --offset 24h

# Network troubleshooting
az network watcher test-connectivity --source-resource vm1 --dest-address 10.0.0.4 --dest-port 80

# List all costs
az consumption usage list --start-date 2025-01-01 --end-date 2025-01-31 --query "[].{Name:instanceName, Cost:pretaxCost}" -o table
```

---

## Best Practices for Troubleshooting

1. **Always check Activity Logs first** - Shows what changed recently
2. **Use Network Watcher** - Essential for connectivity issues
3. **Enable diagnostic settings** - Before problems occur
4. **Tag resources** - Makes filtering and cost tracking easier
5. **Use resource locks** - Prevent accidental deletions
6. **Set up alerts** - Proactive monitoring
7. **Document changes** - Use tags, comments, or change management system
8. **Test in dev first** - Never troubleshoot in production

---

## Additional Resources

- [Azure Troubleshooting Documentation](https://learn.microsoft.com/azure/troubleshoot/)
- [Network Watcher](https://learn.microsoft.com/azure/network-watcher/)
- [Azure Monitor Troubleshooting](https://learn.microsoft.com/azure/azure-monitor/troubleshoot)
- [Azure Support](https://azure.microsoft.com/support/options/)

---

**Remember: Most issues have simple solutions. Start with the basics before diving into complex troubleshooting!**
