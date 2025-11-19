# Lab 02: Azure RBAC and Resource Access Management

## Lab Overview
**Duration:** 60 minutes
**Difficulty:** Intermediate
**Exam Skills:** Manage Azure Identities and Governance (20-25%)

## Learning Objectives
- Understand Azure RBAC concepts and scope levels
- Assign built-in roles at different scopes
- Create custom RBAC roles
- Interpret effective permissions
- Troubleshoot access issues

## Prerequisites
- Completed Lab 01 (Entra ID Users and Groups)
- Azure subscription with Owner or User Access Administrator role
- At least 2 test users created

## Lab Scenario
Contoso Corporation needs to implement proper access controls across their Azure environment. You'll configure role assignments at various scopes, create custom roles for specific needs, and verify access permissions.

---

## Exercise 1: Understand RBAC Scope Hierarchy

### Task 1: Review Scope Levels

RBAC in Azure operates at four scope levels:
1. **Management Group** (highest level)
2. **Subscription**
3. **Resource Group**
4. **Resource** (lowest level)

**View your scope hierarchy:**
```powershell
# Connect to Azure
Connect-AzAccount

# View subscription
Get-AzSubscription

# View resource groups
Get-AzResourceGroup | Select-Object ResourceGroupName, Location

# View management groups (if configured)
Get-AzManagementGroup
```

### Task 2: Create Test Resource Groups
```powershell
# Create resource groups for testing
New-AzResourceGroup -Name "rg-dev" -Location "eastus"
New-AzResourceGroup -Name "rg-prod" -Location "eastus"
New-AzResourceGroup -Name "rg-shared" -Location "eastus"
```

**Azure CLI:**
```bash
az group create --name rg-dev --location eastus
az group create --name rg-prod --location eastus
az group create --name rg-shared --location eastus
```

---

## Exercise 2: Assign Built-in Roles

### Task 1: Assign Reader Role at Subscription Scope

**Portal Method:**
1. Navigate to **Subscriptions** > Select your subscription
2. Click **Access control (IAM)** > **Add role assignment**
3. Select **Reader** role
4. Under **Members**, select a user (e.g., Bob Smith)
5. Review and assign

**PowerShell Method:**
```powershell
# Get subscription scope
$subscriptionId = (Get-AzSubscription).Id
$scope = "/subscriptions/$subscriptionId"

# Get user object ID
$user = Get-AzADUser -UserPrincipalName "bob@yourdomain.onmicrosoft.com"

# Assign Reader role at subscription scope
New-AzRoleAssignment -ObjectId $user.Id `
    -RoleDefinitionName "Reader" `
    -Scope $scope
```

**Azure CLI:**
```bash
# Get subscription ID
SUBSCRIPTION_ID=$(az account show --query id -o tsv)

# Get user object ID
USER_ID=$(az ad user show --id bob@yourdomain.onmicrosoft.com --query id -o tsv)

# Assign Reader role
az role assignment create \
    --assignee $USER_ID \
    --role "Reader" \
    --scope "/subscriptions/$SUBSCRIPTION_ID"
```

### Task 2: Assign Contributor Role at Resource Group Scope

**PowerShell:**
```powershell
# Assign Contributor to dev resource group
$user = Get-AzADUser -UserPrincipalName "alice@yourdomain.onmicrosoft.com"
$rg = Get-AzResourceGroup -Name "rg-dev"

New-AzRoleAssignment -ObjectId $user.Id `
    -RoleDefinitionName "Contributor" `
    -Scope $rg.ResourceId
```

**Azure CLI:**
```bash
USER_ID=$(az ad user show --id alice@yourdomain.onmicrosoft.com --query id -o tsv)

az role assignment create \
    --assignee $USER_ID \
    --role "Contributor" \
    --resource-group rg-dev
```

### Task 3: Assign Role to Group

**PowerShell:**
```powershell
# Assign Virtual Machine Contributor to group
$group = Get-AzADGroup -DisplayName "IT-Developers"
$rg = Get-AzResourceGroup -Name "rg-dev"

New-AzRoleAssignment -ObjectId $group.Id `
    -RoleDefinitionName "Virtual Machine Contributor" `
    -Scope $rg.ResourceId
```

### Task 4: View Role Assignments

**Portal Method:**
1. Navigate to resource group
2. Click **Access control (IAM)** > **Role assignments**
3. Review assignments by role, user, or scope

**PowerShell:**
```powershell
# List all role assignments for a resource group
Get-AzRoleAssignment -ResourceGroupName "rg-dev" |
    Select-Object DisplayName, RoleDefinitionName, Scope |
    Format-Table

# List role assignments for specific user
Get-AzRoleAssignment -SignInName "alice@yourdomain.onmicrosoft.com" |
    Select-Object DisplayName, RoleDefinitionName, Scope |
    Format-Table
```

**Azure CLI:**
```bash
# List role assignments for resource group
az role assignment list --resource-group rg-dev -o table

# List for specific user
az role assignment list --assignee alice@yourdomain.onmicrosoft.com -o table
```

---

## Exercise 3: Create Custom RBAC Roles

### Task 1: Create Custom Role Definition File

**Create JSON file (custom-role-vm-operator.json):**
```json
{
    "Name": "Virtual Machine Operator",
    "Id": null,
    "IsCustom": true,
    "Description": "Can monitor, start, and stop virtual machines",
    "Actions": [
        "Microsoft.Compute/*/read",
        "Microsoft.Compute/virtualMachines/start/action",
        "Microsoft.Compute/virtualMachines/restart/action",
        "Microsoft.Compute/virtualMachines/powerOff/action",
        "Microsoft.Insights/alertRules/*",
        "Microsoft.Resources/subscriptions/resourceGroups/read",
        "Microsoft.Resources/deployments/read",
        "Microsoft.Storage/storageAccounts/listKeys/action"
    ],
    "NotActions": [
        "Microsoft.Compute/virtualMachines/delete",
        "Microsoft.Compute/virtualMachines/write"
    ],
    "DataActions": [],
    "NotDataActions": [],
    "AssignableScopes": [
        "/subscriptions/YOUR_SUBSCRIPTION_ID"
    ]
}
```

### Task 2: Create Custom Role Using PowerShell

```powershell
# Get subscription ID
$subscriptionId = (Get-AzSubscription).Id

# Read the JSON file
$roleDefinition = Get-Content -Path "custom-role-vm-operator.json" | ConvertFrom-Json

# Update AssignableScopes with actual subscription ID
$roleDefinition.AssignableScopes = @("/subscriptions/$subscriptionId")

# Save updated JSON
$roleDefinition | ConvertTo-Json -Depth 10 | Out-File "custom-role-vm-operator-updated.json"

# Create the custom role
New-AzRoleDefinition -InputFile "custom-role-vm-operator-updated.json"
```

**Azure CLI:**
```bash
# Update subscription ID in JSON file
SUBSCRIPTION_ID=$(az account show --query id -o tsv)
sed "s/YOUR_SUBSCRIPTION_ID/$SUBSCRIPTION_ID/g" custom-role-vm-operator.json > custom-role-vm-operator-updated.json

# Create custom role
az role definition create --role-definition @custom-role-vm-operator-updated.json
```

### Task 3: Assign Custom Role

**PowerShell:**
```powershell
$user = Get-AzADUser -UserPrincipalName "carol@yourdomain.onmicrosoft.com"
$rg = Get-AzResourceGroup -Name "rg-prod"

New-AzRoleAssignment -ObjectId $user.Id `
    -RoleDefinitionName "Virtual Machine Operator" `
    -Scope $rg.ResourceId
```

### Task 4: Create Custom Role Using Actions from Existing Role

**PowerShell:**
```powershell
# Start with an existing role
$role = Get-AzRoleDefinition "Virtual Machine Contributor"

# Modify it
$role.Id = $null
$role.Name = "Virtual Machine Operator Custom"
$role.Description = "Can manage VMs but not delete them"
$role.IsCustom = $true

# Remove delete actions
$role.Actions.Remove("Microsoft.Compute/virtualMachines/delete")

# Set assignable scopes
$subscriptionId = (Get-AzSubscription).Id
$role.AssignableScopes.Clear()
$role.AssignableScopes.Add("/subscriptions/$subscriptionId")

# Create the role
New-AzRoleDefinition -Role $role
```

---

## Exercise 4: Interpret Effective Permissions

### Task 1: Check Effective Permissions

**Portal Method:**
1. Navigate to resource group
2. Click **Access control (IAM)** > **Check access**
3. Select a user
4. Review effective permissions

**PowerShell:**
```powershell
# Function to get effective permissions
function Get-EffectivePermissions {
    param (
        [string]$UserPrincipalName,
        [string]$ResourceGroupName
    )

    $user = Get-AzADUser -UserPrincipalName $UserPrincipalName
    $rg = Get-AzResourceGroup -Name $ResourceGroupName

    # Get direct assignments
    $directAssignments = Get-AzRoleAssignment -ObjectId $user.Id -Scope $rg.ResourceId

    # Get group assignments
    $userGroups = Get-AzADGroup -MemberObjectId $user.Id
    $groupAssignments = @()
    foreach ($group in $userGroups) {
        $groupAssignments += Get-AzRoleAssignment -ObjectId $group.Id -Scope $rg.ResourceId
    }

    Write-Host "Direct Role Assignments:" -ForegroundColor Green
    $directAssignments | Select-Object DisplayName, RoleDefinitionName | Format-Table

    Write-Host "Inherited from Groups:" -ForegroundColor Yellow
    $groupAssignments | Select-Object DisplayName, RoleDefinitionName | Format-Table
}

# Use the function
Get-EffectivePermissions -UserPrincipalName "alice@yourdomain.onmicrosoft.com" -ResourceGroupName "rg-dev"
```

### Task 2: Test Permissions

**Create a test VM to verify permissions:**
```powershell
# As user with Contributor role
Connect-AzAccount -Credential (Get-Credential)

# Try to create VM
New-AzVm -ResourceGroupName "rg-dev" `
    -Name "test-vm" `
    -Location "eastus" `
    -Image "Win2022AzureEditionCore" `
    -Size "Standard_B2s"

# This should succeed for Contributor role
# Will fail for Reader role
```

---

## Exercise 5: Troubleshoot Access Issues

### Task 1: Common Access Issues and Solutions

**Scenario 1: User can't see resources**
```powershell
# Check if user has any role assignments
$user = Get-AzADUser -UserPrincipalName "user@domain.com"
Get-AzRoleAssignment -ObjectId $user.Id

# Check group memberships
Get-AzADGroup -MemberObjectId $user.Id
```

**Scenario 2: User can see but not modify resources**
```powershell
# Check role capabilities
Get-AzRoleDefinition -Name "Reader" |
    Select-Object -ExpandProperty Actions

# Compare with required role
Get-AzRoleDefinition -Name "Contributor" |
    Select-Object -ExpandProperty Actions
```

**Scenario 3: Role assignment not taking effect**
```powershell
# RBAC changes can take up to 30 minutes
# Force refresh by signing out and back in

# Check Azure AD token
Get-AzAccessToken
```

### Task 2: Use Activity Log for Troubleshooting

**PowerShell:**
```powershell
# Check recent RBAC changes
Get-AzLog -ResourceGroupName "rg-dev" |
    Where-Object {$_.Authorization -ne $null} |
    Select-Object EventTimestamp, Caller, OperationName, @{Name="Role";Expression={$_.Authorization.Action}} |
    Format-Table
```

**Azure CLI:**
```bash
# View activity log
az monitor activity-log list \
    --resource-group rg-dev \
    --offset 7d \
    --query "[?contains(authorization.action, 'roleAssignments')].{Time:eventTimestamp, Caller:caller, Operation:operationName.localizedValue}" \
    -o table
```

---

## Exercise 6: Deny Assignments (Advanced)

### Task 1: Understand Deny Assignments

Deny assignments are created automatically by Azure Blueprints and Azure managed apps to protect resources.

**View deny assignments:**
```powershell
# List deny assignments (rare in most environments)
Get-AzDenyAssignment -Scope "/subscriptions/$subscriptionId/resourceGroups/rg-prod"
```

---

## Verification Checklist

```powershell
# Comprehensive verification script
Write-Host "=== RBAC Verification Report ===" -ForegroundColor Cyan

# 1. Resource Groups
Write-Host "`n1. Resource Groups:" -ForegroundColor Green
Get-AzResourceGroup | Select-Object ResourceGroupName | Format-Table

# 2. All Role Assignments
Write-Host "`n2. Role Assignments by Resource Group:" -ForegroundColor Green
$rgs = @("rg-dev", "rg-prod", "rg-shared")
foreach ($rg in $rgs) {
    Write-Host "`n  Resource Group: $rg" -ForegroundColor Yellow
    Get-AzRoleAssignment -ResourceGroupName $rg |
        Select-Object DisplayName, RoleDefinitionName |
        Format-Table
}

# 3. Custom Roles
Write-Host "`n3. Custom Roles:" -ForegroundColor Green
Get-AzRoleDefinition | Where-Object {$_.IsCustom -eq $true} |
    Select-Object Name, Description |
    Format-Table

# 4. User Assignments
Write-Host "`n4. User Role Assignments:" -ForegroundColor Green
$users = @("alice@yourdomain.onmicrosoft.com", "bob@yourdomain.onmicrosoft.com", "carol@yourdomain.onmicrosoft.com")
foreach ($upn in $users) {
    Write-Host "`n  User: $upn" -ForegroundColor Yellow
    Get-AzRoleAssignment -SignInName $upn |
        Select-Object RoleDefinitionName, Scope |
        Format-Table
}
```

---

## Knowledge Check

1. **What's the difference between Actions and DataActions in a role definition?**
   - Actions: Control plane operations (manage resources)
   - DataActions: Data plane operations (access data within resources)

2. **What happens when a user has multiple role assignments?**
   - Permissions are cumulative (additive)
   - Deny assignments override allow permissions

3. **How long can it take for RBAC changes to take effect?**
   - Usually immediate, but can take up to 30 minutes

4. **Can you assign roles to service principals?**
   - Yes, using the same cmdlets with the service principal's Object ID

5. **What's the maximum number of custom roles per Azure AD tenant?**
   - 5,000 custom roles

---

## Cleanup Resources

```powershell
# Remove role assignments
$users = @("alice@yourdomain.onmicrosoft.com", "bob@yourdomain.onmicrosoft.com", "carol@yourdomain.onmicrosoft.com")
foreach ($upn in $users) {
    $assignments = Get-AzRoleAssignment -SignInName $upn
    foreach ($assignment in $assignments) {
        Remove-AzRoleAssignment -ObjectId $assignment.ObjectId `
            -RoleDefinitionName $assignment.RoleDefinitionName `
            -Scope $assignment.Scope
    }
}

# Remove custom roles
$customRoles = Get-AzRoleDefinition | Where-Object {$_.IsCustom -eq $true}
foreach ($role in $customRoles) {
    Remove-AzRoleDefinition -Id $role.Id -Force
}

# Remove resource groups
Remove-AzResourceGroup -Name "rg-dev" -Force
Remove-AzResourceGroup -Name "rg-prod" -Force
Remove-AzResourceGroup -Name "rg-shared" -Force
```

---

## Additional Resources
- [Azure RBAC Documentation](https://learn.microsoft.com/en-us/azure/role-based-access-control/)
- [Built-in Roles](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles)
- [Custom Roles](https://learn.microsoft.com/en-us/azure/role-based-access-control/custom-roles)
- [Best Practices](https://learn.microsoft.com/en-us/azure/role-based-access-control/best-practices)

---

## Next Steps
Continue to [Lab 03: Azure Policy and Governance](lab03-azure-policy-governance.md)
