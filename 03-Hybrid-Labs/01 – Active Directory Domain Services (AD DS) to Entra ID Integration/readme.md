# Hybrid Lab 01 – Active Directory Domain Services (AD DS) to Entra ID Integration

## Summary
This lab demonstrates a complete hybrid identity deployment by integrating **on‑prem Active Directory (AD DS)** with **Microsoft Entra ID (Azure AD)**. I built a domain, synchronized identities to the cloud, enabled hybrid join, and configured centralized access control for Remote Desktop using security groups and Group Policy.

This serves as the foundational lab for my hybrid cloud + identity infrastructure series.

---

## Objectives
- Build an on‑prem Active Directory domain
- Configure DNS and promote Domain Controller
- Install and configure Azure AD Connect
- Sync on‑prem users to Entra ID
- Achieve Hybrid Azure AD Join for domain‑joined devices
- Enable MDM auto‑enrollment with Intune
- Implement secure RDP access via AD security group and GPO

---

## Environment Setup
| Component | Role | Notes |
|----------|------|-------|
| DC01 | Domain Controller + DNS + Azure AD Connect | Forest: `cloudwithmilu.com` |
| CL01 | Windows client | Domain‑joined and hybrid joined |
| Azure / Entra ID | Cloud identity | Domain verified: `cloudwithmilu.com` |
| Intune | Device management | Auto‑enrollment enabled |

Network: **10.10.1.0/24**  
Domain: **cloudwithmilu.com**

---

## What I Did
- Installed AD DS and promoted a new forest
- Installed and configured Azure AD Connect
- Synced AD users to Entra ID
- Verified cloud identities
- Created `LabComputers` OU and redirected domain computer join path
- Configured GPO for Hybrid Join device registration
- Joined CL01 to the domain → automatically placed in `LabComputers` OU
- Verified Hybrid Azure AD Join with `dsregcmd`
- Configured Intune MDM auto‑enrollment
- Created `RDP Users` security group
- Added `ComputerTester` to RDP Users
- Created and linked GPO to add domain RDP group to local RDP group on endpoints
- Confirmed successful RDP access without granting domain admin rights

---

## What I Accomplished
| Result | Verified By |
|--------|------------|
Domain deployed | `Get‑ADDomain` |
Users synced to Entra | Entra portal & `Get‑MsolUser` |
Hybrid Azure AD Join | `dsregcmd /status` |
Devices redirected to OU | Join log + ADUC |
RDP access controlled by AD group | Successful remote login |
Intune auto‑enrollment | Intune devices list |

---

## What I Learned
- Hybrid identity requires **both AD DS and Entra configuration**
- Hybrid Join ≠ Intune enrollment — they're separate layers
- Default `Computers` container can’t take GPOs → must use an OU
- Group Policy + AD groups = proper enterprise device access model
- DO NOT make normal users Domain Admin for remote access
- Hybrid join workflow & troubleshooting (`dsregcmd`, scheduled tasks, logs)

---

## Key Commands Used
### AD DS Setup
```powershell
Install‑WindowsFeature AD‑Domain‑Services ‑IncludeManagementTools
Install‑ADDSForest ‑DomainName "cloudwithmilu.com"
```

### Verify AD & Users
```powershell
Get‑ADDomain
Get‑ADUser ‑Filter * | Select Name
```

### Azure AD Sync Verification
```powershell
Get‑MsolUser ‑UserPrincipalName testuser1@cloudwithmilu.com
```

### Hybrid Join Status
```powershell
dsregcmd /status
```

### Redirect Default Computer Join OU
```powershell
redircmp "OU=LabComputers,DC=cloudwithmilu,DC=com"
```

### GPO Refresh
```powershell
gpupdate /force
```

### Add AD Group to Local RDP Group (manual test)
```powershell
Add‑LocalGroupMember ‑Group "Remote Desktop Users" ‑Member "CLOUDWITHMILU\RDP Users"
```

---

## GPO Design (RDP Access)
| Object | Value |
|-------|--------|
GPO Name | `Allow RDP Access` |
OU Linked | `LabComputers` |
Setting | Restricted Groups |
Group | `Remote Desktop Users` |
Members | `CLOUDWITHMILU\RDP Users` |

---

## Final Device Status Snapshot
```
AzureAdJoined : YES
DomainJoined  : YES
EnterpriseJoined : NO (Expected)
IsUserAzureAD : YES
```

---

## Next Steps
Planned Hybrid Labs:
- Intune compliance + Conditional Access
- Hybrid AD + GPO + Intune co‑management
- Azure Files with AD DS authentication
- Privileged Identity Management (PIM)
