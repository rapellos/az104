# Lab 01: Managing Microsoft Entra ID Users and Groups

## Lab Overview
**Duration:** 45 minutes
**Difficulty:** Beginner
**Exam Skills:** Manage Azure Identities and Governance (20-25%)

## Learning Objectives
- Create and manage users in Microsoft Entra ID
- Create and manage groups with dynamic membership rules
- Configure Self-Service Password Reset (SSPR)
- Manage external users (B2B)
- Assign licenses to users

## Prerequisites
- Azure subscription with Global Administrator or User Administrator role
- Microsoft 365 trial (optional, for license assignment)

## Lab Scenario
You are an Azure administrator for Contoso Corporation. You need to set up identity management for a new project team including internal employees, external contractors, and configure self-service capabilities.

---

## Exercise 1: Create and Manage Users

### Task 1: Create a Bulk User Import Template

**Portal Method:**
1. Sign in to [Azure Portal](https://portal.azure.com)
2. Navigate to **Microsoft Entra ID** > **Users**
3. Click **Bulk operations** > **Bulk create**
4. Download the CSV template
5. Add the following users:

```csv
Name,User name,Initial password,Block sign in,First name,Last name,Job title,Department
Alice Johnson,alice@yourdomain.onmicrosoft.com,Pass@word123,No,Alice,Johnson,Developer,IT
Bob Smith,bob@yourdomain.onmicrosoft.com,Pass@word123,No,Bob,Smith,Project Manager,IT
Carol White,carol@yourdomain.onmicrosoft.com,Pass@word123,No,Carol,White,Business Analyst,IT
```

6. Upload the CSV file and verify the import

**PowerShell Method:**
```powershell
# Connect to Microsoft Entra ID
Connect-MgGraph -Scopes "User.ReadWrite.All"

# Create users
$passwordProfile = @{
    Password = "Pass@word123"
    ForceChangePasswordNextSignIn = $true
}

$users = @(
    @{displayName="Alice Johnson"; mailNickname="alice"; userPrincipalName="alice@yourdomain.onmicrosoft.com"; jobTitle="Developer"; department="IT"}
    @{displayName="Bob Smith"; mailNickname="bob"; userPrincipalName="bob@yourdomain.onmicrosoft.com"; jobTitle="Project Manager"; department="IT"}
    @{displayName="Carol White"; mailNickname="carol"; userPrincipalName="carol@yourdomain.onmicrosoft.com"; jobTitle="Business Analyst"; department="IT"}
)

foreach ($user in $users) {
    New-MgUser -DisplayName $user.displayName `
        -MailNickname $user.mailNickname `
        -UserPrincipalName $user.userPrincipalName `
        -PasswordProfile $passwordProfile `
        -JobTitle $user.jobTitle `
        -Department $user.department `
        -AccountEnabled $true
}
```

**Azure CLI Method:**
```bash
# Create users
az ad user create \
    --display-name "Alice Johnson" \
    --user-principal-name alice@yourdomain.onmicrosoft.com \
    --password Pass@word123 \
    --force-change-password-next-sign-in true \
    --job-title "Developer" \
    --department "IT"

az ad user create \
    --display-name "Bob Smith" \
    --user-principal-name bob@yourdomain.onmicrosoft.com \
    --password Pass@word123 \
    --force-change-password-next-sign-in true \
    --job-title "Project Manager" \
    --department "IT"

az ad user create \
    --display-name "Carol White" \
    --user-principal-name carol@yourdomain.onmicrosoft.com \
    --password Pass@word123 \
    --force-change-password-next-sign-in true \
    --job-title "Business Analyst" \
    --department "IT"
```

### Task 2: Verify User Creation
```powershell
# List all users
Get-MgUser | Select-Object DisplayName, UserPrincipalName, Department, JobTitle

# Or with Azure CLI
az ad user list --query "[].{Name:displayName, UPN:userPrincipalName}" -o table
```

---

## Exercise 2: Create and Manage Groups

### Task 1: Create Security Groups

**Portal Method:**
1. Navigate to **Microsoft Entra ID** > **Groups** > **New group**
2. Create the following groups:
   - **Name:** IT-Developers
   - **Group type:** Security
   - **Membership type:** Assigned
   - **Members:** Add Alice Johnson

**PowerShell Method:**
```powershell
# Create security group
$group = New-MgGroup -DisplayName "IT-Developers" `
    -MailEnabled:$false `
    -SecurityEnabled:$true `
    -MailNickname "it-developers" `
    -Description "IT Development Team"

# Add members
$user = Get-MgUser -Filter "displayName eq 'Alice Johnson'"
New-MgGroupMember -GroupId $group.Id -DirectoryObjectId $user.Id
```

**Azure CLI Method:**
```bash
# Create group
az ad group create \
    --display-name "IT-Developers" \
    --mail-nickname "it-developers" \
    --description "IT Development Team"

# Add member
USER_ID=$(az ad user show --id alice@yourdomain.onmicrosoft.com --query id -o tsv)
GROUP_ID=$(az ad group show --group "IT-Developers" --query id -o tsv)
az ad group member add --group $GROUP_ID --member-id $USER_ID
```

### Task 2: Create Dynamic Group

**Portal Method:**
1. Create a new group with these settings:
   - **Name:** IT-Department-Dynamic
   - **Group type:** Security
   - **Membership type:** Dynamic User
   - **Dynamic membership rules:**
     ```
     user.department -eq "IT"
     ```

**PowerShell Method:**
```powershell
# Create dynamic group
$dynamicRule = "(user.department -eq `"IT`")"

New-MgGroup -DisplayName "IT-Department-Dynamic" `
    -MailEnabled:$false `
    -SecurityEnabled:$true `
    -MailNickname "it-dept-dynamic" `
    -Description "All IT Department Users" `
    -GroupTypes "DynamicMembership" `
    -MembershipRule $dynamicRule `
    -MembershipRuleProcessingState "On"
```

### Task 3: Verify Dynamic Membership
```powershell
# Wait a few minutes for processing, then check
Get-MgGroupMember -GroupId (Get-MgGroup -Filter "displayName eq 'IT-Department-Dynamic'").Id
```

---

## Exercise 3: Configure External Users (B2B)

### Task 1: Invite External User

**Portal Method:**
1. Navigate to **Microsoft Entra ID** > **Users** > **New user** > **Invite external user**
2. Fill in:
   - **Email:** contractor@external.com
   - **Display name:** External Contractor
   - **Personal message:** "Welcome to the Contoso project team"
3. Click **Invite**

**PowerShell Method:**
```powershell
# Invite external user
New-MgInvitation -InvitedUserEmailAddress "contractor@external.com" `
    -InvitedUserDisplayName "External Contractor" `
    -SendInvitationMessage:$true `
    -InviteRedirectUrl "https://myapps.microsoft.com"
```

**Azure CLI Method:**
```bash
az rest --method POST \
    --url https://graph.microsoft.com/v1.0/invitations \
    --headers "Content-Type=application/json" \
    --body '{
        "invitedUserEmailAddress": "contractor@external.com",
        "invitedUserDisplayName": "External Contractor",
        "sendInvitationMessage": true,
        "inviteRedirectUrl": "https://myapps.microsoft.com"
    }'
```

---

## Exercise 4: Configure Self-Service Password Reset (SSPR)

### Task 1: Enable SSPR

**Portal Method:**
1. Navigate to **Microsoft Entra ID** > **Password reset**
2. Under **Properties**, set **Self service password reset enabled** to **All**
3. Configure **Authentication methods**:
   - Number of methods required: **2**
   - Available methods: Enable **Email**, **Mobile phone**, and **Security questions**
4. Set **Registration**:
   - Require users to register: **Yes**
   - Days before users must reconfirm: **180**
5. Configure **Notifications**:
   - Notify users on password resets: **Yes**
   - Notify all admins: **Yes**

**PowerShell Method:**
```powershell
# Enable SSPR (requires Azure AD Premium)
$params = @{
    selfServicePasswordResetEnabled = "All"
}

# Note: Full SSPR configuration requires Graph API calls
# This is a simplified example
```

### Task 2: Test SSPR Registration
1. Open an InPrivate browser window
2. Navigate to https://aka.ms/ssprsetup
3. Sign in as one of the created users
4. Register authentication methods

---

## Exercise 5: Assign Licenses (Optional)

### Task 1: Assign Azure AD Premium License

**Portal Method:**
1. Navigate to **Microsoft Entra ID** > **Licenses** > **All products**
2. Select a product (e.g., **Azure AD Premium P2**)
3. Click **Assign**
4. Select users to assign licenses
5. Configure assignment options
6. Click **Assign**

**PowerShell Method:**
```powershell
# Assign license
$user = Get-MgUser -Filter "displayName eq 'Alice Johnson'"
$sku = Get-MgSubscribedSku -All | Where-Object {$_.SkuPartNumber -eq 'AAD_PREMIUM_P2'}

Set-MgUserLicense -UserId $user.Id `
    -AddLicenses @{SkuId = $sku.SkuId} `
    -RemoveLicenses @()
```

---

## Verification Steps

### Verify All Tasks Complete
```powershell
# 1. Check users
Get-MgUser | Select-Object DisplayName, UserPrincipalName | Format-Table

# 2. Check groups
Get-MgGroup | Select-Object DisplayName, GroupTypes, MembershipRule | Format-Table

# 3. Check group membership
$group = Get-MgGroup -Filter "displayName eq 'IT-Department-Dynamic'"
Get-MgGroupMember -GroupId $group.Id

# 4. Check external users
Get-MgUser -Filter "userType eq 'Guest'" | Select-Object DisplayName, UserPrincipalName
```

---

## Knowledge Check

1. **What is the difference between assigned and dynamic groups?**
   - Assigned: Manual membership management
   - Dynamic: Automatic membership based on user attributes

2. **What authentication methods can be used for SSPR?**
   - Email, mobile phone, office phone, security questions, mobile app notification, mobile app code

3. **What permissions are needed to invite external users?**
   - Guest Inviter role or higher

4. **How long does it take for dynamic group membership to update?**
   - Can take 5 minutes to several hours depending on tenant size

---

## Cleanup Resources

```powershell
# Remove users
$users = @("alice@yourdomain.onmicrosoft.com", "bob@yourdomain.onmicrosoft.com", "carol@yourdomain.onmicrosoft.com")
foreach ($upn in $users) {
    Remove-MgUser -UserId $upn
}

# Remove groups
$groups = @("IT-Developers", "IT-Department-Dynamic")
foreach ($groupName in $groups) {
    $group = Get-MgGroup -Filter "displayName eq '$groupName'"
    Remove-MgGroup -GroupId $group.Id
}

# Remove external user
Remove-MgUser -UserId "contractor_external.com#EXT#@yourdomain.onmicrosoft.com"
```

---

## Additional Resources
- [Microsoft Entra ID Documentation](https://learn.microsoft.com/en-us/entra/identity/)
- [Create dynamic groups](https://learn.microsoft.com/en-us/entra/identity/users/groups-dynamic-membership)
- [Configure SSPR](https://learn.microsoft.com/en-us/entra/identity/authentication/tutorial-enable-sspr)
- [B2B Collaboration](https://learn.microsoft.com/en-us/entra/external-id/what-is-b2b)

---

## Next Steps
Continue to [Lab 02: Azure RBAC and Resource Access](lab02-rbac-resource-access.md)
