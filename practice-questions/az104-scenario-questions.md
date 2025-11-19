# AZ-104 Scenario-Based Practice Questions

## Introduction
These scenario-based questions mirror real-world situations you'll encounter on the AZ-104 exam. Each question presents a business scenario requiring you to apply Azure administration knowledge.

---

## Identity and Governance Scenarios

### Scenario 1: Multi-Department Access Control

**Scenario:**
Contoso Corporation has three departments: Development, Operations, and Finance. They need to implement a governance structure where:
- Developers can create and manage resources in dev environments only
- Operations team needs to manage all environments but cannot delete production resources
- Finance team needs read-only access to all resources for cost reporting
- All departments share a common set of approved Azure regions and VM sizes

**Question 1.1:**
What Azure governance features should you implement? (Select all that apply)

A. Create separate subscriptions for each department
B. Use Management Groups to organize subscriptions by environment
C. Implement Azure Policy to restrict regions and VM sizes
D. Use resource locks on production resources
E. Create custom RBAC roles for each team

**Correct Answers: B, C, D**

**Explanation:**
- B: Management Groups allow policy and RBAC inheritance across subscriptions
- C: Azure Policy enforces organizational standards (regions, VM sizes)
- D: Resource locks prevent accidental deletion of production resources
- A: Not necessary; resource groups and RBAC can provide isolation
- E: Built-in roles (Contributor, Reader) are sufficient; custom roles add complexity

**Question 1.2:**
Which RBAC role should you assign to the Operations team for production resources?

A. Owner
B. Contributor
C. Virtual Machine Contributor
D. Reader

**Correct Answer: B**

**Explanation:**
Contributor provides full management capabilities except role assignments and deletions (when combined with resource locks).

**Question 1.3:**
You need to prevent anyone from deleting a critical production SQL database. What should you configure?

A. Azure Policy with Deny effect
B. Resource lock with CanNotDelete level
C. RBAC role with restricted permissions
D. Azure Backup with long-term retention

**Correct Answer: B**

**Explanation:**
Resource locks (CanNotDelete) prevent deletion even by Owners. Azure Policy can audit but not prevent at resource level after creation.

---

### Scenario 2: External Contractor Access

**Scenario:**
Your company hires external contractors who need temporary access to Azure resources for 3 months. Requirements:
- Contractors should use their existing work email addresses
- Access should be automatically reviewed and removed after 90 days
- Contractors need access to specific resource groups only
- Must comply with conditional access policies

**Question 2.1:**
How should you provide access to external contractors?

A. Create Azure AD user accounts with temporary passwords
B. Use Azure AD B2B guest user invitations
C. Create a separate Azure AD tenant for contractors
D. Share service principal credentials

**Correct Answer: B**

**Explanation:**
B2B guest users maintain their own credentials and can be managed with access reviews and conditional access policies.

**Question 2.2:**
Which feature automatically removes contractor access after 90 days?

A. Azure AD Access Reviews
B. Conditional Access policies with session controls
C. Azure Policy with audit effect
D. Time-based RBAC assignments

**Correct Answer: A**

**Explanation:**
Azure AD Access Reviews can automatically remove access based on review cycles and policies.

---

## Storage Scenarios

### Scenario 3: Data Lifecycle Management

**Scenario:**
Your company stores the following data in Azure Blob Storage:
- Active application data: Accessed daily
- Log files: Accessed for first 30 days, occasionally after
- Archived reports: Accessed once or twice per year
- Backup data: Must be retained for 7 years for compliance

Requirements:
- Minimize storage costs
- Ensure quick access to active data
- Comply with retention requirements
- Protect against accidental deletion

**Question 3.1:**
Which storage tiers should you use for each data type? (Match each data type to appropriate tier)

A. Active application data → Hot tier
B. Log files → Cool tier after 30 days
C. Archived reports → Archive tier
D. Backup data → Archive tier with immutability policy

**Correct Answers: All**

**Explanation:**
- Hot tier: Optimized for frequent access, higher storage cost but lower access cost
- Cool tier: For data accessed infrequently (30+ days), lower storage cost
- Archive tier: Lowest cost for rarely accessed data (180+ days)
- Immutability: Ensures compliance with WORM (Write Once, Read Many) requirements

**Question 3.2:**
What feature automatically moves data between tiers based on access patterns?

A. Azure Storage Explorer
B. Blob lifecycle management policies
C. Azure Data Factory
D. AzCopy with scheduling

**Correct Answer: B**

**Explanation:**
Blob lifecycle management policies automate tier transitions and deletion based on rules.

**Question 3.3:**
You need to ensure deleted blobs can be recovered for 14 days. What should you configure?

A. Blob versioning
B. Blob snapshots
C. Soft delete for blobs
D. Container-level immutability

**Correct Answer: C**

**Explanation:**
Soft delete retains deleted blobs for a specified retention period (NEW: also applies to containers as of April 2025 exam update).

---

### Scenario 4: Secure Storage Access

**Scenario:**
Your web application needs to:
- Upload user files to blob storage
- Files should be accessible only to authenticated users
- Third-party vendor needs temporary read access to specific container
- Storage account should not be publicly accessible

**Question 4.1:**
How should the web application authenticate to storage?

A. Use storage account access keys
B. Use Shared Access Signature (SAS) tokens
C. Use Azure AD managed identity
D. Use connection string in application code

**Correct Answer: C**

**Explanation:**
Managed identity eliminates credential management and follows Azure security best practices.

**Question 4.2:**
What should you provide to the third-party vendor for temporary read access?

A. Storage account access key
B. Service SAS token with read permission and expiry time
C. Account-level SAS token
D. Storage connection string

**Correct Answer: B**

**Explanation:**
Service SAS token provides granular, time-limited access to specific resources without exposing account keys.

**Question 4.3:**
Which configuration prevents direct internet access to the storage account while allowing Azure services?

A. Configure storage firewall to allow Azure services
B. Use private endpoints
C. Enable Azure AD authentication only
D. Set default network access to Deny and add virtual network rules

**Correct Answer: D (or B for complete isolation)**

**Explanation:**
Both D and B work, but D is more common. Private endpoints provide complete network isolation.

---

## Compute Scenarios

### Scenario 5: High Availability Web Application

**Scenario:**
You're deploying a mission-critical web application with requirements:
- 99.99% SLA
- Auto-scaling based on CPU and memory
- Zero-downtime deployments
- Handle traffic from multiple regions
- Minimum 2 instances always running

**Question 5.1:**
Which compute solution provides the required SLA?

A. Single VM with Premium SSD in availability zone
B. Two VMs in an availability set
C. VMs deployed across availability zones
D. Azure App Service with Basic tier

**Correct Answer: C**

**Explanation:**
Availability zones provide 99.99% SLA by protecting against datacenter failures.

**Question 5.2:**
What should you use for auto-scaling and zero-downtime deployments?

A. Virtual Machine Scale Sets
B. Azure Kubernetes Service
C. Azure App Service with deployment slots
D. Traffic Manager with multiple VMs

**Correct Answer: C**

**Explanation:**
App Service provides built-in auto-scaling, deployment slots for staging/production swaps, and meets all requirements with less management overhead.

**Question 5.3:**
You need to route traffic to the closest regional deployment. What should you configure?

A. Azure Load Balancer
B. Azure Application Gateway
C. Azure Traffic Manager with Performance routing
D. Azure Front Door

**Correct Answer: D (or C)**

**Explanation:**
Azure Front Door or Traffic Manager provide global load balancing. Front Door offers better performance with anycast networking.

---

### Scenario 6: VM Migration and Sizing

**Scenario:**
You're migrating on-premises VMs to Azure:
- Web servers: 2 vCPU, 4 GB RAM, moderate CPU usage
- Database server: 8 vCPU, 64 GB RAM, consistent high memory usage
- Batch processing: 8 vCPU, 16 GB RAM, runs 2 hours daily

**Question 6.1:**
Which VM sizes are most cost-effective? (Match each workload)

A. Web servers → B-series (Burstable)
B. Database server → E-series (Memory optimized)
C. Batch processing → Spot VMs with D-series
D. All → Reserved instances for 3 years

**Correct Answers: A, B, C**

**Explanation:**
- B-series: Cost-effective for variable workloads
- E-series: High memory-to-CPU ratio for databases
- Spot VMs: Up to 90% savings for interruptible workloads
- D: Not all workloads need reserved instances

**Question 6.2:**
The database VM frequently experiences high CPU. What should you do?

A. Enable auto-scaling
B. Resize to a larger E-series VM
C. Add more data disks
D. Configure VM scale set

**Correct Answer: B**

**Explanation:**
Single VMs don't support auto-scaling. Vertical scaling (resize) is appropriate for persistent high resource usage.

---

## Networking Scenarios

### Scenario 7: Hub-Spoke Network Design

**Scenario:**
You're implementing a hub-spoke topology:
- Hub VNet (10.0.0.0/16): Shared services, Azure Firewall, VPN Gateway
- Spoke 1 (10.1.0.0/16): Development workloads
- Spoke 2 (10.2.0.0/16): Production workloads

Requirements:
- All internet traffic routes through Azure Firewall
- Spokes cannot communicate directly
- On-premises connectivity through VPN
- Minimize costs

**Question 7.1:**
What should you configure for inter-spoke isolation?

A. Network Security Groups on each subnet
B. User-defined routes directing spoke-to-spoke traffic through firewall
C. Do not peer spokes together
D. Azure Firewall rules blocking spoke-to-spoke traffic

**Correct Answer: C (and B for additional control)**

**Explanation:**
Without direct peering, spokes are inherently isolated. UDRs enforce routing through hub for inspection.

**Question 7.2:**
How should spokes reach on-premises network?

A. Create separate VPN gateway in each spoke
B. Configure VNet peering with AllowGatewayTransit in hub and UseRemoteGateways in spokes
C. Use Azure Firewall for VPN connectivity
D. Create ExpressRoute circuits for each spoke

**Correct Answer: B**

**Explanation:**
Gateway transit allows spokes to use hub's VPN gateway, minimizing costs (one gateway instead of multiple).

**Question 7.3:**
What ensures all outbound internet traffic goes through Azure Firewall?

A. NSG rules blocking internet access
B. User-defined route with 0.0.0.0/0 pointing to firewall private IP
C. Azure Policy requiring firewall
D. Service endpoints on all subnets

**Correct Answer: B**

**Explanation:**
UDR with default route (0.0.0.0/0) overrides system route to internet, forcing traffic through firewall.

---

### Scenario 8: Secure Web Application

**Scenario:**
You're deploying a public web application:
- Web tier: 3 VMs running IIS
- App tier: 4 VMs running .NET applications
- Database tier: SQL Server on VM

Requirements:
- Only ports 80/443 accessible from internet
- Web tier can only communicate with app tier on port 443
- App tier can only communicate with database tier on port 1433
- No direct internet access from app or database tiers
- SSL termination at load balancer

**Question 8.1:**
Which Azure services should you use? (Select all that apply)

A. Application Gateway with WAF for web tier
B. Internal Load Balancer for app tier
C. Network Security Groups for each tier
D. Application Security Groups for role-based filtering
E. Azure Firewall for outbound internet

**Correct Answers: A, C, D**

**Explanation:**
- Application Gateway: Layer 7 load balancing with SSL offload and WAF
- NSGs: Control traffic between tiers
- ASGs: Simplify NSG rules by grouping VMs by role
- Internal LB could work but ASGs provide better management
- Firewall: Not required if no outbound internet needed

**Question 8.2:**
Which NSG rule allows web tier to access app tier?

```
A. Source: VirtualNetwork, Destination: AppTier-ASG, Port: 443
B. Source: WebTier-ASG, Destination: AppTier-ASG, Port: 443
C. Source: Any, Destination: AppTier-ASG, Port: 443
D. Source: Internet, Destination: AppTier-ASG, Port: 443
```

**Correct Answer: B**

**Explanation:**
ASGs provide granular control. Web tier VMs in WebTier-ASG can access AppTier-ASG on specific port.

---

## Monitoring and Backup Scenarios

### Scenario 9: Proactive Monitoring

**Scenario:**
Your production environment needs:
- Alert when any VM CPU > 80% for 5 minutes
- Alert when storage account has >1000 failed requests in 15 minutes
- Daily report of all resource configuration changes
- Automated remediation when certain conditions met

**Question 9.1:**
Which Azure Monitor features should you configure? (Select all that apply)

A. Metric alerts for VM CPU
B. Log alerts for storage failures
C. Action Groups for notifications
D. Azure Automation runbooks for remediation
E. Activity Log alerts for resource changes

**Correct Answers: A, B, C, D, E**

**Explanation:**
- Metric alerts: Real-time monitoring of metrics
- Log alerts: Query-based alerting on log data
- Action Groups: Notification and remediation actions
- Automation: Execute remediation scripts
- Activity Log: Track resource changes

**Question 9.2:**
What KQL query identifies failed storage operations?

```kusto
A. StorageBlobLogs | where StatusCode >= 400 | summarize Count=count() by AccountName
B. AzureMetrics | where ResourceType == "storageaccounts" | where StatusCode == "Failed"
C. StorageBlobLogs | where StatusCode >= 400 | summarize Count=count() by bin(TimeGenerated, 15m) | where Count > 1000
D. AzureActivity | where ResourceProvider == "Microsoft.Storage" | where Status == "Failed"
```

**Correct Answer: C**

**Explanation:**
Queries storage logs for HTTP errors (>=400), groups by 15-minute windows, filters for >1000 failures.

**Question 9.3:**
You need to automatically restart VMs when CPU stays > 95% for 10 minutes. What should you configure?

A. Metric alert + Action Group + Automation runbook
B. Log Analytics alert + Azure Function
C. Azure Policy + DeployIfNotExists effect
D. Application Insights availability test

**Correct Answer: A**

**Explanation:**
Metric alert triggers on condition, Action Group calls Automation runbook to restart VM.

---

### Scenario 10: Backup and Disaster Recovery

**Scenario:**
Your company requires:
- Azure VM backups: Daily at 2 AM, retain for 30 days
- SQL Server on Azure VM: 15-minute RPO
- Azure File Share: Daily backups, retain for 90 days
- DR site in secondary region with 4-hour RTO

**Question 10.1:**
Which backup solutions should you implement? (Match each)

A. Azure VMs → Azure Backup with daily policy
B. SQL Server → Azure Backup with SQL workload type (15-min log backups)
C. Azure File Share → Azure Backup for Azure Files
D. DR → Azure Site Recovery with recovery plan

**Correct Answers: All**

**Explanation:**
Each workload requires specific backup configuration:
- Azure Backup: Handles VMs, SQL, and Files
- Site Recovery: Provides DR orchestration and failover

**Question 10.2:**
What Recovery Services Vault configuration supports all requirements?

A. Single vault in primary region, GRS redundancy
B. Separate vaults for each workload, LRS redundancy
C. Single vault in each region, ZRS redundancy
D. Azure Backup Vault for operational backup only

**Correct Answer: A**

**Explanation:**
Single GRS vault in primary region replicates to secondary region automatically, supporting DR requirements.

**Question 10.3:**
You need to restore SQL Server to 1:30 PM yesterday. Which restore option should you use?

A. Full backup restore
B. Point-in-time restore
C. Differential backup restore
D. Snapshot restore

**Correct Answer: B**

**Explanation:**
15-minute log backups enable point-in-time restore to any moment within retention period.

---

## Multi-Domain Scenarios

### Scenario 11: Complete Solution Design

**Scenario:**
Contoso needs to deploy a new e-commerce platform:
- Web application: 10-100 concurrent users, global audience
- Product database: 500 GB, high read/write operations
- Product images: 2 TB, frequently accessed
- Order processing: Batch job running nightly
- Compliance: Data must stay in specific regions

Requirements:
- High availability (99.95% minimum)
- Auto-scaling based on demand
- Secure, no public endpoints except web tier
- Cost optimization where possible
- Full monitoring and backup

**Question 11.1:**
What architecture components should you use? (Select all that apply)

A. Azure App Service with multiple instances across availability zones
B. Azure SQL Database with Business Critical tier
C. Azure Storage account with Hot tier for images + Azure CDN
D. Azure Functions or Logic Apps for batch processing
E. Application Gateway with WAF
F. Virtual Network with subnets for each tier
G. Azure Front Door for global distribution

**Correct Answers: A, B, C, D, E, F, G**

**Explanation:**
Complete solution requires:
- App Service: Managed PaaS with auto-scaling
- SQL Database: Managed PaaS with built-in HA
- Storage + CDN: Cost-effective for static content
- Functions: Serverless for batch processing
- App Gateway: Layer 7 load balancing + WAF
- VNet: Network isolation
- Front Door: Global distribution + caching

**Question 11.2:**
How do you restrict database access to app tier only?

A. Configure SQL firewall rules for app tier public IPs
B. Use Virtual Network service endpoints
C. Use Private Link/Private Endpoint
D. Configure NSG rules

**Correct Answer: C**

**Explanation:**
Private Link provides completely private connectivity, no public endpoint exposure.

**Question 11.3:**
What ensures data sovereignty (data in specific regions)?

A. Azure Policy requiring specific locations
B. RBAC restrictions on resource creation
C. Management Groups with geographic organization
D. Resource locks preventing movement

**Correct Answer: A**

**Explanation:**
Azure Policy with Allowed Locations ensures resources can only be created in compliant regions.

---

## Answer Summary

### Identity & Governance
- Scenario 1.1: B, C, D
- Scenario 1.2: B
- Scenario 1.3: B
- Scenario 2.1: B
- Scenario 2.2: A

### Storage
- Scenario 3.1: All
- Scenario 3.2: B
- Scenario 3.3: C
- Scenario 4.1: C
- Scenario 4.2: B
- Scenario 4.3: D

### Compute
- Scenario 5.1: C
- Scenario 5.2: C
- Scenario 5.3: D
- Scenario 6.1: A, B, C
- Scenario 6.2: B

### Networking
- Scenario 7.1: C
- Scenario 7.2: B
- Scenario 7.3: B
- Scenario 8.1: A, C, D
- Scenario 8.2: B

### Monitoring & Backup
- Scenario 9.1: A, B, C, D, E
- Scenario 9.2: C
- Scenario 9.3: A
- Scenario 10.1: All
- Scenario 10.2: A
- Scenario 10.3: B

### Multi-Domain
- Scenario 11.1: All
- Scenario 11.2: C
- Scenario 11.3: A

---

## Study Tips for Scenario Questions

1. **Read the entire scenario first** - Understand business requirements before looking at questions
2. **Identify constraints** - Cost, region, compliance, SLA requirements
3. **Think holistically** - Consider security, availability, and cost together
4. **Eliminate wrong answers** - Rule out options that don't meet requirements
5. **Watch for keywords**:
   - "Least effort" → Use managed services
   - "Most cost-effective" → Consider reserved instances, Spot VMs, tiers
   - "Highest availability" → Availability zones, geo-redundancy
   - "Minimize management" → PaaS over IaaS

---

**Next Steps:**
- Review [AZ-104 Quick Reference Guide](../quick-reference/az104-cheat-sheet.md)
- Practice with [Hands-on Labs](../labs/)
- Take practice exams to test knowledge application
