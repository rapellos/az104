# AZ-104 Study Plan Guide

## Choose Your Study Timeline

Select the plan that matches your timeline to the exam:

- [4-Week Intensive Plan](#4-week-intensive-plan) - For experienced Azure users
- [8-Week Comprehensive Plan](#8-week-comprehensive-plan) - Recommended for most learners
- [12-Week Foundation Plan](#12-week-foundation-plan) - For Azure beginners

---

## 4-Week Intensive Plan

**Prerequisites:** 6+ months Azure experience, familiar with basic concepts

### Week 1: Identity, Governance & Storage
**Study (10 hours):**
- Review [Skills Measured](az104-skills-measured-apr-2025.md)
- Study Identity & Governance (20-25%)
- Study Storage (15-20%)
- Review [Quick Reference Guide](quick-reference/az104-cheat-sheet.md)

**Hands-On (10 hours):**
- [Lab 01: Entra ID Users and Groups](labs/identity-governance/lab01-entra-id-users-groups.md)
- [Lab 02: RBAC and Resource Access](labs/identity-governance/lab02-rbac-resource-access.md)
- [Lab 01: Storage Accounts and Blob](labs/storage/lab01-storage-accounts-blob.md)

**Practice (5 hours):**
- Complete Identity & Storage [practice questions](practice-questions/az104-practice-questions.md)
- Review [scenario questions](practice-questions/az104-scenario-questions.md)

**Total: ~25 hours**

---

### Week 2: Compute & Containers
**Study (10 hours):**
- Study Compute (20-25%)
- ARM Templates and Bicep deep dive
- App Service and Containers
- Review Microsoft Learn modules

**Hands-On (10 hours):**
- Create VMs with availability sets and zones
- Deploy ARM templates and Bicep
- Configure VM Scale Sets
- Deploy containers (ACI, Container Apps)

**Practice (5 hours):**
- Complete Compute practice questions
- Work through troubleshooting scenarios
- Practice Bicep syntax

**Total: ~25 hours**

---

### Week 3: Networking
**Study (8 hours):**
- Study Networking (15-20%)
- VNets, NSGs, ASGs deep dive
- Load balancers vs Application Gateway
- VPN Gateway and ExpressRoute

**Hands-On (12 hours):**
- [Lab 01: Virtual Networks and NSGs](labs/networking/lab01-virtual-networks-nsg.md)
- Configure VNet peering (hub-spoke topology)
- Set up load balancers
- Implement private endpoints

**Practice (5 hours):**
- Complete Networking practice questions
- Troubleshoot connectivity scenarios
- Practice NSG rule creation

**Total: ~25 hours**

---

### Week 4: Monitoring, Review & Practice Exams
**Study (5 hours):**
- Study Monitoring & Backup (10-15%)
- Azure Monitor and Log Analytics
- KQL queries
- Backup and Site Recovery

**Hands-On (8 hours):**
- Configure Azure Monitor
- Create alert rules and action groups
- Set up VM backup
- Practice KQL queries

**Practice (12 hours):**
- Take 2-3 full practice exams (MeasureUp)
- Review all weak areas
- Flash cards for commands
- Review troubleshooting guide

**Total: ~25 hours**

**EXAM READY! Total: ~100 hours**

---

## 8-Week Comprehensive Plan

**Prerequisites:** 3+ months Azure experience or cloud experience

### Weeks 1-2: Identity & Governance
**Study (8 hours/week):**
- Microsoft Entra ID concepts
- RBAC roles and scopes
- Azure Policy and initiatives
- Management Groups
- Cost management

**Hands-On (8 hours/week):**
- All Identity & Governance labs
- Create custom RBAC roles
- Implement policy assignments
- Configure cost alerts

**Practice (4 hours/week):**
- Practice questions
- Microsoft Learn assessments
- Create flashcards

**Total: ~40 hours**

---

### Weeks 3-4: Storage
**Study (8 hours/week):**
- Storage account types and tiers
- Redundancy options (LRS, GRS, ZRS, GZRS)
- Blob lifecycle management
- Soft delete (blobs AND containers - new!)
- SAS tokens and access policies
- Azure Files and File Sync

**Hands-On (8 hours/week):**
- All Storage labs
- Configure lifecycle policies
- Test soft delete recovery
- Practice AzCopy commands
- Implement private endpoints

**Practice (4 hours/week):**
- Storage scenarios
- Cost optimization exercises
- Security configurations

**Total: ~40 hours**

---

### Weeks 5-6: Compute & Containers
**Study (8 hours/week):**
- VM sizing and series
- Availability options (Sets vs Zones)
- VM Scale Sets
- ARM templates structure
- Bicep syntax
- App Service plans and tiers
- Container Instances and Apps

**Hands-On (8 hours/week):**
- Create VMs all ways (Portal, PS, CLI, Bicep)
- Deploy scale sets with auto-scaling
- Convert ARM to Bicep
- Deploy App Service with slots
- Run containerized applications

**Practice (4 hours/week):**
- Compute scenarios
- Template modification practice
- High availability design

**Total: ~40 hours**

---

### Week 7: Networking
**Study (10 hours):**
- VNet addressing and subnetting
- NSG vs ASG
- VNet peering and gateway transit
- Load Balancer types
- Application Gateway with WAF
- Azure DNS
- Private Link vs Service Endpoints

**Hands-On (10 hours):**
- Build hub-spoke topology
- Configure load balancing
- Set up Application Gateway
- Implement private endpoints
- Troubleshoot connectivity with Network Watcher

**Practice (5 hours):**
- Networking scenarios
- IP addressing exercises
- Routing problems

**Total: ~25 hours**

---

### Week 8: Monitoring, Backup & Final Review
**Study (8 hours):**
- Azure Monitor metrics and logs
- KQL query language
- Alert rules and action groups
- Azure Backup
- Site Recovery

**Hands-On (8 hours):**
- Configure monitoring for all resources
- Write KQL queries
- Set up backup policies
- Test restore procedures

**Practice (15 hours):**
- 3-4 full practice exams
- Review all weak areas
- Speed practice (time management)
- Final review of cheat sheet

**Total: ~31 hours**

**EXAM READY! Total: ~216 hours**

---

## 12-Week Foundation Plan

**Prerequisites:** Little to no Azure experience

### Weeks 1-2: Azure Fundamentals
**Study (10 hours/week):**
- Azure basics and terminology
- Azure Portal navigation
- Resource groups and resources
- Subscriptions and billing
- PowerShell and CLI basics

**Hands-On (6 hours/week):**
- Create free Azure account
- Install PowerShell and CLI tools
- Practice basic commands
- Explore Azure Portal

**Total: ~32 hours**

---

### Weeks 3-4: Identity & Governance
**Study (8 hours/week):**
- Microsoft Entra ID deep dive
- Users, groups, and external users
- RBAC concepts
- Azure Policy
- Management Groups

**Hands-On (8 hours/week):**
- Complete all Identity labs
- Create users and groups
- Assign RBAC roles
- Implement policies

**Practice (4 hours/week):**
- Practice questions
- Microsoft Learn modules

**Total: ~40 hours**

---

### Weeks 5-6: Storage
**Study (8 hours/week):**
- Storage fundamentals
- Account types and tiers
- Blob storage and access tiers
- Lifecycle management
- Security and access control

**Hands-On (8 hours/week):**
- Complete all Storage labs
- Practice with Storage Explorer
- Use AzCopy
- Configure security

**Practice (4 hours/week):**
- Storage scenarios
- Security exercises

**Total: ~40 hours**

---

### Weeks 7-8: Compute
**Study (8 hours/week):**
- Virtual Machines
- Availability options
- ARM templates basics
- Bicep introduction
- App Service

**Hands-On (8 hours/week):**
- Create and manage VMs
- Work with ARM/Bicep
- Deploy App Service
- Configure auto-scaling

**Practice (4 hours/week):**
- Compute scenarios
- Template practice

**Total: ~40 hours**

---

### Weeks 9-10: Networking
**Study (8 hours/week):**
- Networking fundamentals
- IP addressing basics
- VNets and subnets
- NSGs and security
- Load balancing

**Hands-On (8 hours/week):**
- Complete all Networking labs
- Build network topologies
- Configure security rules
- Troubleshoot connectivity

**Practice (4 hours/week):**
- Networking scenarios
- Subnetting exercises

**Total: ~40 hours**

---

### Week 11: Monitoring & Backup
**Study (8 hours):**
- Azure Monitor
- Log Analytics
- Basic KQL
- Backup and recovery

**Hands-On (8 hours):**
- Configure monitoring
- Create alerts
- Set up backups
- Practice restore

**Practice (4 hours):**
- Monitoring scenarios
- KQL practice

**Total: ~20 hours**

---

### Week 12: Review & Practice
**Review (10 hours):**
- Review all weak areas
- Cheat sheet memorization
- Command syntax review

**Practice (20 hours):**
- 4-5 practice exams
- Timed practice
- Scenario review
- Final preparation

**Total: ~30 hours**

**EXAM READY! Total: ~242 hours**

---

## Daily Study Routine

### Morning (Before Work/School)
- **30 minutes:** Review flashcards and cheat sheet
- **30 minutes:** Watch video or read documentation

### Evening
- **1 hour:** Hands-on labs
- **30 minutes:** Practice questions
- **30 minutes:** Note taking and review

**Total: 3 hours/day**

---

## Study Tips

### Active Learning
1. ✅ **Hands-on first** - Do the labs before watching videos
2. ✅ **Teach others** - Explain concepts to someone
3. ✅ **Take notes** - Write summaries in your own words
4. ✅ **Practice commands** - Type them out, don't copy-paste
5. ✅ **Create diagrams** - Draw architectures by hand

### Retention Techniques
1. **Spaced Repetition** - Review material at increasing intervals
2. **Flashcards** - Create cards for commands and concepts
3. **Mind Maps** - Visual connections between topics
4. **Practice Tests** - Take regularly to identify gaps
5. **Real-world Projects** - Build actual solutions in Azure

### Time Management
1. **Schedule study time** - Treat it like a class
2. **Eliminate distractions** - Phone off, quiet environment
3. **Pomodoro technique** - 25 min study, 5 min break
4. **Track progress** - Use the lab completion tracker
5. **Adjust as needed** - If behind, increase hours or extend timeline

---

## Weekly Checklist Template

```markdown
## Week X: [Topic]

### Monday
- [ ] Study: [specific topic] (X hours)
- [ ] Lab: [specific lab] (X hours)
- [ ] Review notes from last week (30 min)

### Tuesday
- [ ] Study: [specific topic] (X hours)
- [ ] Lab: [specific lab] (X hours)
- [ ] Practice questions (30 min)

### Wednesday
- [ ] Study: [specific topic] (X hours)
- [ ] Hands-on practice (X hours)
- [ ] Create flashcards (30 min)

### Thursday
- [ ] Study: [specific topic] (X hours)
- [ ] Lab: [specific lab] (X hours)
- [ ] Practice questions (30 min)

### Friday
- [ ] Review week's material (1 hour)
- [ ] Hands-on practice (2 hours)
- [ ] Take practice test (1 hour)

### Weekend
- [ ] Review weak areas (2 hours)
- [ ] Additional labs as needed (2-4 hours)
- [ ] Plan next week (30 min)

Weekly Total: XX hours
```

---

## Resources Checklist

### Required
- [ ] Azure subscription (free tier)
- [ ] PowerShell 7+ installed
- [ ] Azure CLI installed
- [ ] Visual Studio Code with Azure extensions
- [ ] Microsoft Graph PowerShell module
- [ ] Azure Storage Explorer
- [ ] AzCopy

### Recommended
- [ ] MeasureUp practice exams (paid)
- [ ] Microsoft Learn account (free)
- [ ] Notebook for hand-written notes
- [ ] Whiteboard for diagrams
- [ ] Study group or partner

### Nice to Have
- [ ] Second monitor
- [ ] Udemy/Pluralsight courses
- [ ] Azure book/eBook
- [ ] Flashcard app (Anki)

---

## Progress Tracking

### Knowledge Areas (Rate 1-5)
- [ ] Identity & Governance: __/5
- [ ] Storage: __/5
- [ ] Compute: __/5
- [ ] Networking: __/5
- [ ] Monitoring: __/5

### Skills (Rate 1-5)
- [ ] PowerShell: __/5
- [ ] Azure CLI: __/5
- [ ] ARM/Bicep: __/5
- [ ] KQL: __/5
- [ ] Azure Portal: __/5

### Practice Exam Scores
1. First attempt: __/1000 (Date: ______)
2. Second attempt: __/1000 (Date: ______)
3. Third attempt: __/1000 (Date: ______)
4. Fourth attempt: __/1000 (Date: ______)

**Target: 750+ consistently before scheduling exam**

---

## Final Week Checklist

### 7 Days Before
- [ ] Take final practice exam
- [ ] Identify remaining weak areas
- [ ] Review all scenarios
- [ ] Practice time management

### 3 Days Before
- [ ] Review cheat sheet
- [ ] Light hands-on practice
- [ ] Get good sleep
- [ ] Confirm exam details

### Day Before
- [ ] Review cheat sheet only (no new material!)
- [ ] Prepare exam environment (if online)
- [ ] Test equipment (webcam, internet)
- [ ] Get full night's sleep

### Exam Day
- [ ] Eat a good breakfast
- [ ] Arrive 15 min early (or login early for online)
- [ ] Bring ID and confirmation number
- [ ] Stay calm and trust your preparation!

---

## After the Exam

### If You Pass 🎉
- [ ] Celebrate!
- [ ] Share your success
- [ ] Update LinkedIn
- [ ] Plan next certification (AZ-400, AZ-305, etc.)
- [ ] Help others prepare

### If You Don't Pass
- [ ] Don't be discouraged (many people don't pass first try)
- [ ] Review exam report for weak areas
- [ ] Schedule retake (wait period may apply)
- [ ] Focus on failed domains
- [ ] More hands-on practice in weak areas

---

## Questions?

Refer back to:
- [README.md](README.md) - Main resource hub
- [Labs README](labs/README.md) - Lab details
- [Quick Reference](quick-reference/az104-cheat-sheet.md) - Commands and concepts
- [Troubleshooting](troubleshooting/common-scenarios.md) - Common issues

**Good luck with your AZ-104 journey!** 🚀
