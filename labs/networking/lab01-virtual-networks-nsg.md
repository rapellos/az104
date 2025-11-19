# Lab 01: Virtual Networks and Network Security Groups

## Lab Overview
**Duration:** 75 minutes
**Difficulty:** Intermediate
**Exam Skills:** Implement and Manage Virtual Networking (15-20%)

## Learning Objectives
- Create and configure virtual networks and subnets
- Implement virtual network peering
- Configure and manage Network Security Groups (NSGs)
- Configure Application Security Groups (ASGs)
- Implement user-defined routes (UDRs)
- Troubleshoot network connectivity

## Prerequisites
- Azure subscription
- Basic understanding of IP addressing and subnetting
- Completed identity labs (for testing with VMs - optional)

## Lab Scenario
Contoso Corporation is implementing a hub-and-spoke network topology in Azure. You'll create virtual networks, configure security rules, implement peering, and set up custom routing.

---

## Exercise 1: Create Virtual Network Infrastructure

### Task 1: Create Hub Virtual Network

**Portal Method:**
1. Navigate to **Virtual networks** > **Create**
2. Configure:
   - **Resource group:** Create "rg-network-lab"
   - **Name:** vnet-hub
   - **Region:** East US
3. **IP Addresses:**
   - **Address space:** 10.0.0.0/16
   - **Subnets:**
     - GatewaySubnet: 10.0.0.0/27
     - AzureFirewallSubnet: 10.0.1.0/26
     - AzureBastionSubnet: 10.0.2.0/26
     - SharedServicesSubnet: 10.0.10.0/24
4. **Security:** Keep defaults
5. Review and create

**PowerShell:**
```powershell
# Create resource group
New-AzResourceGroup -Name "rg-network-lab" -Location "eastus"

# Define subnets
$gwSubnet = New-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -AddressPrefix "10.0.0.0/27"
$fwSubnet = New-AzVirtualNetworkSubnetConfig -Name "AzureFirewallSubnet" -AddressPrefix "10.0.1.0/26"
$bastionSubnet = New-AzVirtualNetworkSubnetConfig -Name "AzureBastionSubnet" -AddressPrefix "10.0.2.0/26"
$sharedSubnet = New-AzVirtualNetworkSubnetConfig -Name "SharedServicesSubnet" -AddressPrefix "10.0.10.0/24"

# Create virtual network
$vnetHub = New-AzVirtualNetwork -Name "vnet-hub" `
    -ResourceGroupName "rg-network-lab" `
    -Location "eastus" `
    -AddressPrefix "10.0.0.0/16" `
    -Subnet $gwSubnet,$fwSubnet,$bastionSubnet,$sharedSubnet

Write-Host "Hub VNet created: $($vnetHub.Name)"
```

**Bicep Template:**
```bicep
param location string = resourceGroup().location

resource vnetHub 'Microsoft.Network/virtualNetworks@2023-05-01' = {
  name: 'vnet-hub'
  location: location
  properties: {
    addressSpace: {
      addressPrefixes: [
        '10.0.0.0/16'
      ]
    }
    subnets: [
      {
        name: 'GatewaySubnet'
        properties: {
          addressPrefix: '10.0.0.0/27'
        }
      }
      {
        name: 'AzureFirewallSubnet'
        properties: {
          addressPrefix: '10.0.1.0/26'
        }
      }
      {
        name: 'AzureBastionSubnet'
        properties: {
          addressPrefix: '10.0.2.0/26'
        }
      }
      {
        name: 'SharedServicesSubnet'
        properties: {
          addressPrefix: '10.0.10.0/24'
        }
      }
    ]
  }
}

output vnetId string = vnetHub.id
output vnetName string = vnetHub.name
```

**Azure CLI:**
```bash
# Create resource group
az group create --name rg-network-lab --location eastus

# Create virtual network with subnets
az network vnet create \
    --name vnet-hub \
    --resource-group rg-network-lab \
    --location eastus \
    --address-prefix 10.0.0.0/16 \
    --subnet-name SharedServicesSubnet \
    --subnet-prefix 10.0.10.0/24

# Add additional subnets
az network vnet subnet create \
    --vnet-name vnet-hub \
    --resource-group rg-network-lab \
    --name GatewaySubnet \
    --address-prefix 10.0.0.0/27

az network vnet subnet create \
    --vnet-name vnet-hub \
    --resource-group rg-network-lab \
    --name AzureFirewallSubnet \
    --address-prefix 10.0.1.0/26

az network vnet subnet create \
    --vnet-name vnet-hub \
    --resource-group rg-network-lab \
    --name AzureBastionSubnet \
    --address-prefix 10.0.2.0/26
```

### Task 2: Create Spoke Virtual Networks

**PowerShell:**
```powershell
# Create Spoke 1 (Development)
$spoke1Subnet = New-AzVirtualNetworkSubnetConfig -Name "AppSubnet" -AddressPrefix "10.1.1.0/24"
$vnetSpoke1 = New-AzVirtualNetwork -Name "vnet-spoke1-dev" `
    -ResourceGroupName "rg-network-lab" `
    -Location "eastus" `
    -AddressPrefix "10.1.0.0/16" `
    -Subnet $spoke1Subnet

# Create Spoke 2 (Production)
$spoke2Subnet = New-AzVirtualNetworkSubnetConfig -Name "AppSubnet" -AddressPrefix "10.2.1.0/24"
$vnetSpoke2 = New-AzVirtualNetwork -Name "vnet-spoke2-prod" `
    -ResourceGroupName "rg-network-lab" `
    -Location "eastus" `
    -AddressPrefix "10.2.0.0/16" `
    -Subnet $spoke2Subnet

Write-Host "Spoke VNets created: $($vnetSpoke1.Name), $($vnetSpoke2.Name)"
```

**Azure CLI:**
```bash
# Create Spoke 1
az network vnet create \
    --name vnet-spoke1-dev \
    --resource-group rg-network-lab \
    --location eastus \
    --address-prefix 10.1.0.0/16 \
    --subnet-name AppSubnet \
    --subnet-prefix 10.1.1.0/24

# Create Spoke 2
az network vnet create \
    --name vnet-spoke2-prod \
    --resource-group rg-network-lab \
    --location eastus \
    --address-prefix 10.2.0.0/16 \
    --subnet-name AppSubnet \
    --subnet-prefix 10.2.1.0/24
```

---

## Exercise 2: Configure Network Security Groups

### Task 1: Create NSG with Security Rules

**PowerShell:**
```powershell
# Create NSG rules
$rule1 = New-AzNetworkSecurityRuleConfig -Name "Allow-HTTP" `
    -Description "Allow HTTP traffic" `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 100 `
    -SourceAddressPrefix Internet `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 80

$rule2 = New-AzNetworkSecurityRuleConfig -Name "Allow-HTTPS" `
    -Description "Allow HTTPS traffic" `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 110 `
    -SourceAddressPrefix Internet `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 443

$rule3 = New-AzNetworkSecurityRuleConfig -Name "Allow-RDP-From-Bastion" `
    -Description "Allow RDP from Bastion subnet" `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 120 `
    -SourceAddressPrefix "10.0.2.0/26" `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 3389

$rule4 = New-AzNetworkSecurityRuleConfig -Name "Deny-All-Inbound" `
    -Description "Deny all other inbound traffic" `
    -Access Deny `
    -Protocol * `
    -Direction Inbound `
    -Priority 4096 `
    -SourceAddressPrefix * `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange *

# Create NSG
$nsg = New-AzNetworkSecurityGroup -Name "nsg-web-servers" `
    -ResourceGroupName "rg-network-lab" `
    -Location "eastus" `
    -SecurityRules $rule1,$rule2,$rule3,$rule4

Write-Host "NSG created: $($nsg.Name)"
```

**Azure CLI:**
```bash
# Create NSG
az network nsg create \
    --name nsg-web-servers \
    --resource-group rg-network-lab \
    --location eastus

# Add rules
az network nsg rule create \
    --nsg-name nsg-web-servers \
    --resource-group rg-network-lab \
    --name Allow-HTTP \
    --priority 100 \
    --direction Inbound \
    --access Allow \
    --protocol Tcp \
    --source-address-prefixes Internet \
    --source-port-ranges '*' \
    --destination-address-prefixes '*' \
    --destination-port-ranges 80

az network nsg rule create \
    --nsg-name nsg-web-servers \
    --resource-group rg-network-lab \
    --name Allow-HTTPS \
    --priority 110 \
    --direction Inbound \
    --access Allow \
    --protocol Tcp \
    --source-address-prefixes Internet \
    --source-port-ranges '*' \
    --destination-address-prefixes '*' \
    --destination-port-ranges 443

az network nsg rule create \
    --nsg-name nsg-web-servers \
    --resource-group rg-network-lab \
    --name Allow-RDP-From-Bastion \
    --priority 120 \
    --direction Inbound \
    --access Allow \
    --protocol Tcp \
    --source-address-prefixes 10.0.2.0/26 \
    --source-port-ranges '*' \
    --destination-address-prefixes '*' \
    --destination-port-ranges 3389
```

### Task 2: Associate NSG with Subnet

**PowerShell:**
```powershell
# Get subnet
$vnet = Get-AzVirtualNetwork -Name "vnet-spoke1-dev" -ResourceGroupName "rg-network-lab"
$subnet = Get-AzVirtualNetworkSubnetConfig -Name "AppSubnet" -VirtualNetwork $vnet

# Associate NSG
$subnet.NetworkSecurityGroup = $nsg
Set-AzVirtualNetwork -VirtualNetwork $vnet

Write-Host "NSG associated with subnet"
```

**Azure CLI:**
```bash
# Associate NSG with subnet
az network vnet subnet update \
    --vnet-name vnet-spoke1-dev \
    --name AppSubnet \
    --resource-group rg-network-lab \
    --network-security-group nsg-web-servers
```

### Task 3: View Effective Security Rules

**PowerShell:**
```powershell
# View effective NSG rules (requires a NIC)
# This example shows how to check after creating a VM

# Get-AzEffectiveNetworkSecurityGroup -NetworkInterfaceName "vm-nic" -ResourceGroupName "rg-network-lab"
```

**Azure CLI:**
```bash
# View NSG rules
az network nsg rule list \
    --nsg-name nsg-web-servers \
    --resource-group rg-network-lab \
    --output table
```

---

## Exercise 3: Configure Application Security Groups

### Task 1: Create Application Security Groups

**PowerShell:**
```powershell
# Create ASGs
$asgWeb = New-AzApplicationSecurityGroup -Name "asg-web" `
    -ResourceGroupName "rg-network-lab" `
    -Location "eastus"

$asgApp = New-AzApplicationSecurityGroup -Name "asg-app" `
    -ResourceGroupName "rg-network-lab" `
    -Location "eastus"

$asgDb = New-AzApplicationSecurityGroup -Name "asg-database" `
    -ResourceGroupName "rg-network-lab" `
    -Location "eastus"

Write-Host "ASGs created: $($asgWeb.Name), $($asgApp.Name), $($asgDb.Name)"
```

**Azure CLI:**
```bash
az network asg create --name asg-web --resource-group rg-network-lab --location eastus
az network asg create --name asg-app --resource-group rg-network-lab --location eastus
az network asg create --name asg-database --resource-group rg-network-lab --location eastus
```

### Task 2: Create NSG Rules Using ASGs

**PowerShell:**
```powershell
# Create NSG for ASG-based rules
$asgNsg = New-AzNetworkSecurityGroup -Name "nsg-with-asg" `
    -ResourceGroupName "rg-network-lab" `
    -Location "eastus"

# Allow web to app tier
$rule1 = New-AzNetworkSecurityRuleConfig -Name "Allow-Web-to-App" `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 100 `
    -SourceApplicationSecurityGroup $asgWeb `
    -SourcePortRange * `
    -DestinationApplicationSecurityGroup $asgApp `
    -DestinationPortRange 443

# Allow app to database tier
$rule2 = New-AzNetworkSecurityRuleConfig -Name "Allow-App-to-DB" `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 110 `
    -SourceApplicationSecurityGroup $asgApp `
    -SourcePortRange * `
    -DestinationApplicationSecurityGroup $asgDb `
    -DestinationPortRange 1433

# Update NSG
$asgNsg.SecurityRules.Add($rule1)
$asgNsg.SecurityRules.Add($rule2)
Set-AzNetworkSecurityGroup -NetworkSecurityGroup $asgNsg
```

**Azure CLI:**
```bash
# Create NSG
az network nsg create --name nsg-with-asg --resource-group rg-network-lab --location eastus

# Create rules with ASGs
az network nsg rule create \
    --nsg-name nsg-with-asg \
    --resource-group rg-network-lab \
    --name Allow-Web-to-App \
    --priority 100 \
    --direction Inbound \
    --access Allow \
    --protocol Tcp \
    --source-asgs asg-web \
    --destination-asgs asg-app \
    --destination-port-ranges 443

az network nsg rule create \
    --nsg-name nsg-with-asg \
    --resource-group rg-network-lab \
    --name Allow-App-to-DB \
    --priority 110 \
    --direction Inbound \
    --access Allow \
    --protocol Tcp \
    --source-asgs asg-app \
    --destination-asgs asg-database \
    --destination-port-ranges 1433
```

---

## Exercise 4: Configure VNet Peering

### Task 1: Create Hub-to-Spoke Peering

**PowerShell:**
```powershell
# Hub to Spoke1 peering
Add-AzVirtualNetworkPeering -Name "hub-to-spoke1" `
    -VirtualNetwork $vnetHub `
    -RemoteVirtualNetworkId $vnetSpoke1.Id `
    -AllowForwardedTraffic `
    -AllowGatewayTransit

# Spoke1 to Hub peering
Add-AzVirtualNetworkPeering -Name "spoke1-to-hub" `
    -VirtualNetwork $vnetSpoke1 `
    -RemoteVirtualNetworkId $vnetHub.Id `
    -AllowForwardedTraffic `
    -UseRemoteGateways:$false

# Hub to Spoke2 peering
Add-AzVirtualNetworkPeering -Name "hub-to-spoke2" `
    -VirtualNetwork $vnetHub `
    -RemoteVirtualNetworkId $vnetSpoke2.Id `
    -AllowForwardedTraffic `
    -AllowGatewayTransit

# Spoke2 to Hub peering
Add-AzVirtualNetworkPeering -Name "spoke2-to-hub" `
    -VirtualNetwork $vnetSpoke2 `
    -RemoteVirtualNetworkId $vnetHub.Id `
    -AllowForwardedTraffic `
    -UseRemoteGateways:$false

Write-Host "VNet peering configured"
```

**Azure CLI:**
```bash
# Get VNet IDs
HUB_ID=$(az network vnet show --name vnet-hub --resource-group rg-network-lab --query id -o tsv)
SPOKE1_ID=$(az network vnet show --name vnet-spoke1-dev --resource-group rg-network-lab --query id -o tsv)
SPOKE2_ID=$(az network vnet show --name vnet-spoke2-prod --resource-group rg-network-lab --query id -o tsv)

# Create peerings
az network vnet peering create \
    --name hub-to-spoke1 \
    --resource-group rg-network-lab \
    --vnet-name vnet-hub \
    --remote-vnet $SPOKE1_ID \
    --allow-forwarded-traffic \
    --allow-gateway-transit

az network vnet peering create \
    --name spoke1-to-hub \
    --resource-group rg-network-lab \
    --vnet-name vnet-spoke1-dev \
    --remote-vnet $HUB_ID \
    --allow-forwarded-traffic

az network vnet peering create \
    --name hub-to-spoke2 \
    --resource-group rg-network-lab \
    --vnet-name vnet-hub \
    --remote-vnet $SPOKE2_ID \
    --allow-forwarded-traffic \
    --allow-gateway-transit

az network vnet peering create \
    --name spoke2-to-hub \
    --resource-group rg-network-lab \
    --vnet-name vnet-spoke2-prod \
    --remote-vnet $HUB_ID \
    --allow-forwarded-traffic
```

### Task 2: Verify Peering Status

**PowerShell:**
```powershell
# Check peering status
Get-AzVirtualNetworkPeering -VirtualNetworkName "vnet-hub" -ResourceGroupName "rg-network-lab" |
    Select-Object Name, PeeringState, AllowForwardedTraffic, AllowGatewayTransit |
    Format-Table
```

**Azure CLI:**
```bash
az network vnet peering list \
    --vnet-name vnet-hub \
    --resource-group rg-network-lab \
    --output table
```

---

## Exercise 5: Configure User-Defined Routes

### Task 1: Create Route Table

**PowerShell:**
```powershell
# Create route to force traffic through hub
$route1 = New-AzRouteConfig -Name "to-spoke2-via-hub" `
    -AddressPrefix "10.2.0.0/16" `
    -NextHopType "VirtualAppliance" `
    -NextHopIpAddress "10.0.10.4"

$routeTable = New-AzRouteTable -Name "rt-spoke1" `
    -ResourceGroupName "rg-network-lab" `
    -Location "eastus" `
    -Route $route1

# Associate with subnet
$vnet = Get-AzVirtualNetwork -Name "vnet-spoke1-dev" -ResourceGroupName "rg-network-lab"
$subnet = Get-AzVirtualNetworkSubnetConfig -Name "AppSubnet" -VirtualNetwork $vnet
$subnet.RouteTable = $routeTable
Set-AzVirtualNetwork -VirtualNetwork $vnet
```

**Azure CLI:**
```bash
# Create route table
az network route-table create \
    --name rt-spoke1 \
    --resource-group rg-network-lab \
    --location eastus

# Create route
az network route-table route create \
    --route-table-name rt-spoke1 \
    --resource-group rg-network-lab \
    --name to-spoke2-via-hub \
    --address-prefix 10.2.0.0/16 \
    --next-hop-type VirtualAppliance \
    --next-hop-ip-address 10.0.10.4

# Associate with subnet
az network vnet subnet update \
    --vnet-name vnet-spoke1-dev \
    --name AppSubnet \
    --resource-group rg-network-lab \
    --route-table rt-spoke1
```

### Task 2: Create Default Route to Internet

**PowerShell:**
```powershell
# Add default route
Add-AzRouteConfig -Name "default-via-firewall" `
    -AddressPrefix "0.0.0.0/0" `
    -NextHopType "VirtualAppliance" `
    -NextHopIpAddress "10.0.1.4" `
    -RouteTable $routeTable

Set-AzRouteTable -RouteTable $routeTable
```

---

## Exercise 6: Troubleshoot Network Connectivity

### Task 1: Use Network Watcher (Requires VMs)

**Enable Network Watcher:**
```powershell
# Register Network Watcher provider
Register-AzResourceProvider -ProviderNamespace "Microsoft.Network"

# Create Network Watcher
$nw = New-AzNetworkWatcher -Name "NetworkWatcher_eastus" `
    -ResourceGroupName "NetworkWatcherRG" `
    -Location "eastus"
```

### Task 2: Test Connectivity

**PowerShell (requires VMs):**
```powershell
# Test connectivity between VMs
# Test-AzNetworkWatcherConnectivity -NetworkWatcher $nw `
#     -SourceId $sourceVMId `
#     -DestinationId $destVMId `
#     -DestinationPort 443
```

### Task 3: Verify Effective Routes

**PowerShell:**
```powershell
# View effective routes (requires a NIC)
# Get-AzEffectiveRouteTable -NetworkInterfaceName "vm-nic" -ResourceGroupName "rg-network-lab"
```

---

## Verification Checklist

```powershell
Write-Host "=== Network Configuration Verification ===" -ForegroundColor Cyan

# 1. Virtual Networks
Write-Host "`n1. Virtual Networks:" -ForegroundColor Green
Get-AzVirtualNetwork -ResourceGroupName "rg-network-lab" |
    Select-Object Name, Location, @{Name="AddressSpace";Expression={$_.AddressSpace.AddressPrefixes -join ", "}} |
    Format-Table

# 2. Subnets
Write-Host "`n2. Subnets:" -ForegroundColor Green
foreach ($vnet in Get-AzVirtualNetwork -ResourceGroupName "rg-network-lab") {
    Write-Host "`n  VNet: $($vnet.Name)" -ForegroundColor Yellow
    $vnet.Subnets | Select-Object Name, AddressPrefix | Format-Table
}

# 3. NSGs
Write-Host "`n3. Network Security Groups:" -ForegroundColor Green
Get-AzNetworkSecurityGroup -ResourceGroupName "rg-network-lab" |
    Select-Object Name, @{Name="RulesCount";Expression={$_.SecurityRules.Count}} |
    Format-Table

# 4. VNet Peerings
Write-Host "`n4. VNet Peerings:" -ForegroundColor Green
foreach ($vnet in Get-AzVirtualNetwork -ResourceGroupName "rg-network-lab") {
    $peerings = Get-AzVirtualNetworkPeering -VirtualNetworkName $vnet.Name -ResourceGroupName "rg-network-lab"
    if ($peerings) {
        Write-Host "`n  VNet: $($vnet.Name)" -ForegroundColor Yellow
        $peerings | Select-Object Name, PeeringState | Format-Table
    }
}

# 5. Route Tables
Write-Host "`n5. Route Tables:" -ForegroundColor Green
Get-AzRouteTable -ResourceGroupName "rg-network-lab" |
    Select-Object Name, @{Name="RoutesCount";Expression={$_.Routes.Count}} |
    Format-Table

# 6. ASGs
Write-Host "`n6. Application Security Groups:" -ForegroundColor Green
Get-AzApplicationSecurityGroup -ResourceGroupName "rg-network-lab" |
    Select-Object Name, Location |
    Format-Table
```

---

## Knowledge Check

1. **What's the difference between NSG and ASG?**
   - NSG: Network Security Group - filters traffic with rules
   - ASG: Application Security Group - logical grouping of resources

2. **Can spokes communicate directly in hub-spoke topology?**
   - No, by default. Need routing through hub or additional peering

3. **What are the next hop types in UDRs?**
   - VirtualNetworkGateway, VirtualAppliance, VirtualNetwork, Internet, None

4. **What's the difference between service tags and ASGs?**
   - Service tags: Microsoft-managed IP prefixes
   - ASGs: User-defined resource groupings

---

## Cleanup Resources

```powershell
# Remove all network lab resources
Remove-AzResourceGroup -Name "rg-network-lab" -Force
```

---

## Additional Resources
- [Virtual Networks Documentation](https://learn.microsoft.com/en-us/azure/virtual-network/)
- [Network Security Groups](https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview)
- [VNet Peering](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview)
- [User-Defined Routes](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview)

---

## Next Steps
Continue to [Lab 02: Azure Load Balancer and Application Gateway](lab02-load-balancer-appgw.md)
