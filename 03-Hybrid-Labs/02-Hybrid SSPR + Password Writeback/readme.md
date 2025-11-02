# Hybrid SSPR + Password Writeback Lab

This lab enables **Self-Service Password Reset (SSPR)** in Microsoft Entra ID with **on-premises Active Directory password writeback** using Entra Connect.

Users can reset their passwords from the Microsoft portal or “Forgot Password” screen, and the password synchronizes back to **on-prem AD** automatically.

## Objectives
- Enable SSPR for cloud users
- Configure password writeback to on-prem AD
- Reconfigure Entra Connect for hybrid identity
- Modify AD password policy for lab testing
- Validate password reset from portal & login screen

## Tools & Technologies Used
| Technology | Purpose |
|---|---|
Active Directory Domain Services | Identity source (on-prem)  
Microsoft Entra ID | Cloud identity & SSPR  
Microsoft Entra Connect | Directory sync + Password writeback  
PowerShell | Verification & sync commands  
Group Policy Management | Modify AD password policy  

## Architecture
```
User → Entra SSPR Portal
           ↓
   Entra ID verifies identity
           ↓
Password writeback via Entra Connect
           ↓
On-Prem Active Directory (DC01)
```

## Configuration Summary
### Entra ID
- Enabled **SSPR for all users**
- Required registration methods configured
- Verified user registration

### Entra Connect
- Enabled **Password Writeback**
- Verified sync + writeback agent status
- Ran delta sync

### AD DS
Configured **Default Domain Policy** password policies (lab settings):
- Minimum password age: `0`
- Password complexity: **Enabled**
- Password history: `0`
- Minimum length: `8`
- Lockout Threshold: `5`
- Lockout Duration (minutes): `30`

Command to update policy:
```powershell
gpupdate /force
```

## Testing & Results
| Test | Result |
|---|---|
Reset password via **Security Info Portal** | ✅ Success  
Reset password via **Forgot Password** sign‑in link | ✅ Success  
Password synced back to AD | ✅ Confirmed  
PasswordLastSet timestamp updated | ✅ Verified  

## Verification Commands
Trigger sync:
```powershell
Start-ADSyncSyncCycle -PolicyType Delta
```

Confirm writeback:
```powershell
Get-ADSyncAADPasswordPolicy
```

Check AD timestamp:
```powershell
Get-ADUser TestUser1 -Properties PasswordLastSet | FL
```

## What I Learned
- Hybrid identity password flows (cloud → on‑prem)
- SSPR registration requirements
- Entra Connect troubleshooting
- AD password policy interaction with writeback
- Real‑world hybrid IAM skill building

## 📂 Repo Structure
```
02-Hybrid SSPR + Password-Writeback/
│── README.md
│── screenshots/
```
