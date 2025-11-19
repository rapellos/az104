# Lab 01: Azure Storage Accounts and Blob Storage

## Lab Overview
**Duration:** 60 minutes
**Difficulty:** Intermediate
**Exam Skills:** Implement and Manage Storage (15-20%)

## Learning Objectives
- Create and configure storage accounts with different redundancy options
- Configure storage account security (firewall, encryption, access keys)
- Work with Azure Blob Storage (containers, access tiers, lifecycle management)
- Configure soft delete for blobs and containers (NEW in April 2025 exam)
- Implement SAS tokens and stored access policies
- Use Azure Storage Explorer and AzCopy

## Prerequisites
- Azure subscription
- Azure Storage Explorer installed
- AzCopy v10 installed

## Lab Scenario
Contoso needs to implement a storage solution for their web application data, backups, and archive storage with appropriate security and cost optimization.

---

## Exercise 1: Create and Configure Storage Accounts

### Task 1: Create Storage Account with Standard Tier

**Portal Method:**
1. Navigate to **Storage accounts** > **Create**
2. Configure:
   - **Resource group:** Create new "rg-storage-lab"
   - **Storage account name:** "stcontosodev[uniqueid]"
   - **Region:** East US
   - **Performance:** Standard
   - **Redundancy:** Locally-redundant storage (LRS)
3. **Networking:** Enable public endpoint
4. **Data protection:** Enable soft delete for blobs (7 days)
5. Review and create

**PowerShell:**
```powershell
# Create resource group
New-AzResourceGroup -Name "rg-storage-lab" -Location "eastus"

# Create storage account
$storageParams = @{
    ResourceGroupName = "rg-storage-lab"
    Name = "stcontosodev$((Get-Random -Maximum 9999))"
    Location = "eastus"
    SkuName = "Standard_LRS"
    Kind = "StorageV2"
    AccessTier = "Hot"
    EnableHttpsTrafficOnly = $true
    MinimumTlsVersion = "TLS1_2"
}
$storageAccount = New-AzStorageAccount @storageParams

# Enable blob soft delete
Enable-AzStorageBlobDeleteRetentionPolicy `
    -ResourceGroupName "rg-storage-lab" `
    -StorageAccountName $storageAccount.StorageAccountName `
    -RetentionDays 7
```

**Bicep Template:**
```bicep
param location string = resourceGroup().location
param storageAccountName string = 'stcontosodev${uniqueString(resourceGroup().id)}'

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
  properties: {
    accessTier: 'Hot'
    minimumTlsVersion: 'TLS1_2'
    supportsHttpsTrafficOnly: true
    allowBlobPublicAccess: false
    networkAcls: {
      defaultAction: 'Allow'
    }
  }
}

resource blobService 'Microsoft.Storage/storageAccounts/blobServices@2023-01-01' = {
  parent: storageAccount
  name: 'default'
  properties: {
    deleteRetentionPolicy: {
      enabled: true
      days: 7
    }
    containerDeleteRetentionPolicy: {
      enabled: true
      days: 7
    }
  }
}

output storageAccountName string = storageAccount.name
output storageAccountId string = storageAccount.id
```

**Azure CLI:**
```bash
# Create resource group
az group create --name rg-storage-lab --location eastus

# Create storage account
STORAGE_NAME="stcontosodev$RANDOM"
az storage account create \
    --name $STORAGE_NAME \
    --resource-group rg-storage-lab \
    --location eastus \
    --sku Standard_LRS \
    --kind StorageV2 \
    --access-tier Hot \
    --https-only true \
    --min-tls-version TLS1_2 \
    --allow-blob-public-access false

# Enable blob soft delete
az storage account blob-service-properties update \
    --account-name $STORAGE_NAME \
    --resource-group rg-storage-lab \
    --enable-delete-retention true \
    --delete-retention-days 7

# Enable container soft delete (NEW requirement)
az storage account blob-service-properties update \
    --account-name $STORAGE_NAME \
    --resource-group rg-storage-lab \
    --enable-container-delete-retention true \
    --container-delete-retention-days 7
```

### Task 2: Create Premium Storage Account for VMs

**PowerShell:**
```powershell
$premiumStorage = @{
    ResourceGroupName = "rg-storage-lab"
    Name = "stpremium$((Get-Random -Maximum 9999))"
    Location = "eastus"
    SkuName = "Premium_LRS"
    Kind = "StorageV2"
}
New-AzStorageAccount @premiumStorage
```

### Task 3: Configure Storage Account with GRS

**PowerShell:**
```powershell
# Create GRS storage for disaster recovery
$grsStorage = @{
    ResourceGroupName = "rg-storage-lab"
    Name = "stcontosogrs$((Get-Random -Maximum 9999))"
    Location = "eastus"
    SkuName = "Standard_GRS"
    Kind = "StorageV2"
}
$grsAccount = New-AzStorageAccount @grsStorage

# Verify replication status
Get-AzStorageAccount -ResourceGroupName "rg-storage-lab" -Name $grsAccount.StorageAccountName |
    Select-Object StorageAccountName, Location, @{Name="SecondaryLocation";Expression={$_.SecondaryLocation}}, @{Name="StatusOfPrimary";Expression={$_.StatusOfPrimary}}
```

---

## Exercise 2: Configure Storage Account Security

### Task 1: Manage Access Keys

**PowerShell:**
```powershell
# Get storage account
$storageAccount = Get-AzStorageAccount -ResourceGroupName "rg-storage-lab" | Select-Object -First 1

# List access keys
$keys = Get-AzStorageAccountKey -ResourceGroupName $storageAccount.ResourceGroupName `
    -Name $storageAccount.StorageAccountName

Write-Host "Key1: $($keys[0].Value)"
Write-Host "Key2: $($keys[1].Value)"

# Regenerate key1
New-AzStorageAccountKey -ResourceGroupName $storageAccount.ResourceGroupName `
    -Name $storageAccount.StorageAccountName `
    -KeyName key1
```

**Azure CLI:**
```bash
# List keys
az storage account keys list \
    --account-name $STORAGE_NAME \
    --resource-group rg-storage-lab \
    --output table

# Regenerate key
az storage account keys renew \
    --account-name $STORAGE_NAME \
    --resource-group rg-storage-lab \
    --key key1
```

### Task 2: Configure Storage Firewall

**Portal Method:**
1. Navigate to storage account > **Networking**
2. Select **Enabled from selected virtual networks and IP addresses**
3. Add your client IP address
4. Save changes

**PowerShell:**
```powershell
# Get your public IP
$myIP = (Invoke-WebRequest -Uri "https://api.ipify.org").Content

# Update network rules
Update-AzStorageAccountNetworkRuleSet `
    -ResourceGroupName $storageAccount.ResourceGroupName `
    -Name $storageAccount.StorageAccountName `
    -DefaultAction Deny

Add-AzStorageAccountNetworkRule `
    -ResourceGroupName $storageAccount.ResourceGroupName `
    -AccountName $storageAccount.StorageAccountName `
    -IPAddressOrRange $myIP
```

**Azure CLI:**
```bash
# Update default action
az storage account update \
    --name $STORAGE_NAME \
    --resource-group rg-storage-lab \
    --default-action Deny

# Add IP rule
MY_IP=$(curl -s https://api.ipify.org)
az storage account network-rule add \
    --account-name $STORAGE_NAME \
    --resource-group rg-storage-lab \
    --ip-address $MY_IP
```

### Task 3: Enable Storage Analytics and Logging

**PowerShell:**
```powershell
# Get storage context
$ctx = (Get-AzStorageAccount -ResourceGroupName $storageAccount.ResourceGroupName `
    -Name $storageAccount.StorageAccountName).Context

# Enable blob storage logging
Set-AzStorageServiceLoggingProperty -ServiceType Blob `
    -Context $ctx `
    -LoggingOperations All `
    -RetentionDays 7

# Enable metrics
Set-AzStorageServiceMetricsProperty -ServiceType Blob `
    -Context $ctx `
    -MetricsType Hour `
    -MetricsLevel ServiceAndApi `
    -RetentionDays 7
```

---

## Exercise 3: Work with Blob Storage

### Task 1: Create Containers with Different Access Levels

**PowerShell:**
```powershell
$ctx = (Get-AzStorageAccount -ResourceGroupName $storageAccount.ResourceGroupName `
    -Name $storageAccount.StorageAccountName).Context

# Private container (default)
New-AzStorageContainer -Name "private-data" -Context $ctx -Permission Off

# Blob-level public access
New-AzStorageContainer -Name "public-blobs" -Context $ctx -Permission Blob

# Container-level public access
New-AzStorageContainer -Name "public-container" -Context $ctx -Permission Container
```

**Azure CLI:**
```bash
# Get connection string
CONNECTION_STRING=$(az storage account show-connection-string \
    --name $STORAGE_NAME \
    --resource-group rg-storage-lab \
    --output tsv)

# Create containers
az storage container create --name private-data --connection-string $CONNECTION_STRING --public-access off
az storage container create --name public-blobs --connection-string $CONNECTION_STRING --public-access blob
az storage container create --name public-container --connection-string $CONNECTION_STRING --public-access container
```

### Task 2: Upload Blobs and Set Access Tiers

**PowerShell:**
```powershell
# Create test files
"Sample content for hot tier" | Out-File "hot-file.txt"
"Sample content for cool tier" | Out-File "cool-file.txt"
"Sample content for archive tier" | Out-File "archive-file.txt"

# Upload to hot tier (default)
Set-AzStorageBlobContent -File "hot-file.txt" `
    -Container "private-data" `
    -Blob "hot-file.txt" `
    -Context $ctx

# Upload to cool tier
Set-AzStorageBlobContent -File "cool-file.txt" `
    -Container "private-data" `
    -Blob "cool-file.txt" `
    -Context $ctx `
    -StandardBlobTier Cool

# Upload to archive tier
Set-AzStorageBlobContent -File "archive-file.txt" `
    -Container "private-data" `
    -Blob "archive-file.txt" `
    -Context $ctx `
    -StandardBlobTier Archive

# Change blob tier
$blob = Get-AzStorageBlob -Container "private-data" -Blob "hot-file.txt" -Context $ctx
$blob.ICloudBlob.SetStandardBlobTier("Cool")
```

**Azure CLI:**
```bash
# Create test files
echo "Sample content for hot tier" > hot-file.txt
echo "Sample content for cool tier" > cool-file.txt

# Upload files
az storage blob upload \
    --file hot-file.txt \
    --container-name private-data \
    --name hot-file.txt \
    --connection-string $CONNECTION_STRING \
    --tier Hot

az storage blob upload \
    --file cool-file.txt \
    --container-name private-data \
    --name cool-file.txt \
    --connection-string $CONNECTION_STRING \
    --tier Cool

# Change blob tier
az storage blob set-tier \
    --name hot-file.txt \
    --container-name private-data \
    --tier Cool \
    --connection-string $CONNECTION_STRING
```

### Task 3: Configure Blob Lifecycle Management

**Portal Method:**
1. Navigate to storage account > **Lifecycle management**
2. Add a rule:
   - **Rule name:** MoveToArchive
   - **Rule scope:** Apply to all blobs
   - **Blob type:** Block blobs
   - **Conditions:**
     - Days after last modification > 90: Move to cool
     - Days after last modification > 180: Move to archive
     - Days after last modification > 365: Delete

**PowerShell:**
```powershell
# Create lifecycle management rule
$action = Add-AzStorageAccountManagementPolicyAction -BaseBlobAction TierToCool -DaysAfterModificationGreaterThan 90
$action = Add-AzStorageAccountManagementPolicyAction -BaseBlobAction TierToArchive -DaysAfterModificationGreaterThan 180 -InputObject $action
$action = Add-AzStorageAccountManagementPolicyAction -BaseBlobAction Delete -DaysAfterModificationGreaterThan 365 -InputObject $action

$filter = New-AzStorageAccountManagementPolicyFilter -PrefixMatch "private-data/"
$rule = New-AzStorageAccountManagementPolicyRule -Name "ArchiveOldData" -Action $action -Filter $filter

$policy = Set-AzStorageAccountManagementPolicy -ResourceGroupName $storageAccount.ResourceGroupName `
    -StorageAccountName $storageAccount.StorageAccountName `
    -Rule $rule
```

**JSON Policy:**
```json
{
  "rules": [
    {
      "enabled": true,
      "name": "MoveToArchive",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 90
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 180
            },
            "delete": {
              "daysAfterModificationGreaterThan": 365
            }
          }
        },
        "filters": {
          "blobTypes": [
            "blockBlob"
          ],
          "prefixMatch": [
            "private-data/"
          ]
        }
      }
    }
  ]
}
```

### Task 4: Configure Soft Delete for Blobs and Containers

**PowerShell:**
```powershell
# Enable blob soft delete (already done in Exercise 1, but showing again)
Enable-AzStorageBlobDeleteRetentionPolicy `
    -ResourceGroupName $storageAccount.ResourceGroupName `
    -StorageAccountName $storageAccount.StorageAccountName `
    -RetentionDays 14

# Enable container soft delete (NEW in April 2025 exam)
Enable-AzStorageContainerDeleteRetentionPolicy `
    -ResourceGroupName $storageAccount.ResourceGroupName `
    -StorageAccountName $storageAccount.StorageAccountName `
    -RetentionDays 14

# Test soft delete
$testContainer = New-AzStorageContainer -Name "test-soft-delete" -Context $ctx
Remove-AzStorageContainer -Name "test-soft-delete" -Context $ctx -Force

# List deleted containers (including soft-deleted)
Get-AzStorageContainer -Context $ctx -IncludeDeleted

# Restore deleted container
$deletedContainer = Get-AzStorageContainer -Name "test-soft-delete" -Context $ctx -IncludeDeleted
Restore-AzStorageContainer -Context $ctx -Name $deletedContainer.Name
```

---

## Exercise 4: Implement SAS Tokens and Access Policies

### Task 1: Create Account-Level SAS

**PowerShell:**
```powershell
# Create account SAS with limited permissions
$ctx = (Get-AzStorageAccount -ResourceGroupName $storageAccount.ResourceGroupName `
    -Name $storageAccount.StorageAccountName).Context

$startTime = Get-Date
$endTime = $startTime.AddHours(2)

$sasToken = New-AzStorageAccountSASToken -Context $ctx `
    -Service Blob `
    -ResourceType Service,Container,Object `
    -Permission "rl" `
    -StartTime $startTime `
    -ExpiryTime $endTime `
    -Protocol HttpsOnly

Write-Host "Account SAS Token: $sasToken"
```

### Task 2: Create Blob-Level SAS

**PowerShell:**
```powershell
# Create SAS for specific blob
$blobSAS = New-AzStorageBlobSASToken -Container "private-data" `
    -Blob "hot-file.txt" `
    -Permission "r" `
    -StartTime $startTime `
    -ExpiryTime $endTime `
    -Context $ctx `
    -Protocol HttpsOnly

Write-Host "Blob SAS Token: $blobSAS"

# Create full URL
$blobUrl = "https://$($storageAccount.StorageAccountName).blob.core.windows.net/private-data/hot-file.txt$blobSAS"
Write-Host "Full URL with SAS: $blobUrl"
```

### Task 3: Create Stored Access Policy

**PowerShell:**
```powershell
# Create stored access policy
$policy = New-AzStorageContainerStoredAccessPolicy -Container "private-data" `
    -Policy "ReadPolicy" `
    -Permission "r" `
    -StartTime $startTime `
    -ExpiryTime $endTime.AddDays(30) `
    -Context $ctx

# Create SAS using stored access policy
$policySAS = New-AzStorageContainerSASToken -Container "private-data" `
    -Policy "ReadPolicy" `
    -Context $ctx

Write-Host "SAS based on policy: $policySAS"

# Modify the policy (affects all SAS tokens using this policy)
Set-AzStorageContainerStoredAccessPolicy -Container "private-data" `
    -Policy "ReadPolicy" `
    -Permission "rl" `
    -Context $ctx
```

---

## Exercise 5: Use Azure Storage Explorer and AzCopy

### Task 1: Connect Storage Explorer

1. Open Azure Storage Explorer
2. Click **Connect** > **Storage account or service**
3. Choose **Account name and key**
4. Enter storage account details
5. Test connection

### Task 2: Use AzCopy for Bulk Operations

**Create test files:**
```powershell
# Create directory with multiple files
New-Item -Path "test-upload" -ItemType Directory -Force
1..10 | ForEach-Object {
    "Sample content file $_" | Out-File "test-upload/file$_.txt"
}
```

**Upload with AzCopy:**
```bash
# Get storage account key
$key = (Get-AzStorageAccountKey -ResourceGroupName $storageAccount.ResourceGroupName `
    -Name $storageAccount.StorageAccountName)[0].Value

# Upload directory
azcopy copy "test-upload" `
    "https://$($storageAccount.StorageAccountName).blob.core.windows.net/private-data?$sasToken" `
    --recursive

# Download
azcopy copy `
    "https://$($storageAccount.StorageAccountName).blob.core.windows.net/private-data?$sasToken" `
    "test-download" `
    --recursive

# Sync (only copies changes)
azcopy sync "test-upload" `
    "https://$($storageAccount.StorageAccountName).blob.core.windows.net/private-data?$sasToken" `
    --recursive
```

---

## Verification Steps

```powershell
# Complete verification script
Write-Host "=== Storage Account Verification ===" -ForegroundColor Cyan

# 1. List storage accounts
Write-Host "`n1. Storage Accounts:" -ForegroundColor Green
Get-AzStorageAccount -ResourceGroupName "rg-storage-lab" |
    Select-Object StorageAccountName, Location, Sku, Kind |
    Format-Table

# 2. Check blob soft delete
Write-Host "`n2. Soft Delete Configuration:" -ForegroundColor Green
$accounts = Get-AzStorageAccount -ResourceGroupName "rg-storage-lab"
foreach ($account in $accounts) {
    $blobService = Get-AzStorageBlobServiceProperty -ResourceGroupName "rg-storage-lab" `
        -StorageAccountName $account.StorageAccountName

    Write-Host "  $($account.StorageAccountName):"
    Write-Host "    Blob Soft Delete: $($blobService.DeleteRetentionPolicy.Enabled) - $($blobService.DeleteRetentionPolicy.Days) days"
    Write-Host "    Container Soft Delete: $($blobService.ContainerDeleteRetentionPolicy.Enabled) - $($blobService.ContainerDeleteRetentionPolicy.Days) days"
}

# 3. List containers
Write-Host "`n3. Containers:" -ForegroundColor Green
$ctx = $accounts[0].Context
Get-AzStorageContainer -Context $ctx |
    Select-Object Name, PublicAccess |
    Format-Table

# 4. List blobs by tier
Write-Host "`n4. Blobs by Access Tier:" -ForegroundColor Green
Get-AzStorageBlob -Container "private-data" -Context $ctx |
    Select-Object Name, AccessTier, LastModified |
    Format-Table

# 5. List lifecycle policies
Write-Host "`n5. Lifecycle Management Policies:" -ForegroundColor Green
$policy = Get-AzStorageAccountManagementPolicy -ResourceGroupName "rg-storage-lab" `
    -StorageAccountName $accounts[0].StorageAccountName
$policy.Policy.Rules | Select-Object Name, Enabled | Format-Table
```

---

## Knowledge Check

1. **What's the difference between LRS, GRS, and ZRS?**
   - LRS: 3 copies in one datacenter
   - ZRS: 3 copies across availability zones
   - GRS: Replicates to secondary region

2. **When should you use different blob access tiers?**
   - Hot: Frequently accessed data
   - Cool: Infrequently accessed (30+ days)
   - Archive: Rarely accessed (180+ days)

3. **What's the difference between account SAS and service SAS?**
   - Account SAS: Access to multiple services
   - Service SAS: Access to specific service

4. **How does soft delete protect data?**
   - Retains deleted blobs/containers for specified retention period
   - Allows recovery within retention window

---

## Cleanup Resources

```powershell
# Remove all storage lab resources
Remove-AzResourceGroup -Name "rg-storage-lab" -Force

# Clean up local files
Remove-Item -Path "test-upload" -Recurse -Force
Remove-Item -Path "test-download" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item -Path "*.txt" -Force
```

---

## Additional Resources
- [Storage Account Overview](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-overview)
- [Blob Storage Documentation](https://learn.microsoft.com/en-us/azure/storage/blobs/)
- [Lifecycle Management](https://learn.microsoft.com/en-us/azure/storage/blobs/lifecycle-management-overview)
- [Soft Delete](https://learn.microsoft.com/en-us/azure/storage/blobs/soft-delete-blob-overview)
- [AzCopy Documentation](https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azcopy-v10)

---

## Next Steps
Continue to [Lab 02: Azure Files and File Sync](lab02-azure-files-sync.md)
