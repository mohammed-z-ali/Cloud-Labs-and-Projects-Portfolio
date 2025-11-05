# Hybrid Azure AD Join + Intune Autopilot Lab

## Summary

This lab simulates a real-world enterprise deployment where new Windows
devices are automatically enrolled into Intune, hybrid joined to on‑prem
Active Directory and Entra ID, and receive security policies such as
BitLocker during Autopilot setup with zero‑touch provisioning.

## Objectives

-   Deploy Hybrid Azure AD Autopilot
-   Automatically enroll device into Intune
-   Automatically join device to on‑prem Active Directory OU
-   Sync join to Entra ID (Hybrid Join)
-   Apply compliance & security policies (BitLocker)
-   Validate device write‑back to AD & Intune visibility
-   Prepare foundation for Zero Trust endpoint onboarding

## Tools & Services Used

  Category                 | Tools
  ------------------------ | ---------------------------------------------
  Identity & Device Mgmt   | Intune, Entra ID Hybrid Join
  On‑prem Infrastructure   | Windows Server AD DS, DNS, Intune Connector
  Networking               | Azure VM + Hyper‑V nested VM
  Deployment               | Windows Autopilot (User‑Driven Hybrid)
  Security                 | BitLocker (XTS‑AES256 + Key Escrow)
  Validation               | dsregcmd, Intune portal, ADUC

## What We Performed (Step‑By‑Step)

### Environment Setup

-   Built Windows Server DC on Azure VM
-   Configured on‑prem AD + DNS
-   Enabled Entra Connect Hybrid Sync
-   Installed **Intune Connector for Active Directory**
-   Created `LabComputers` OU for Autopilot device objects

### Azure & Intune Configuration

-   Imported Autopilot hardware hash
-   Created **Dynamic Autopilot Device Group**
-   Created **Hybrid Autopilot Profile**
-   Configured **Domain Join profile** targeting OU
-   Verified **Intune AD Connector status = Active**

### Device Provisioning

-   Created nested Windows VM in Hyper‑V
-   Booted from Windows 11 ISO
-   Performed OOBE and signed in using `hybridadmin@cloudwithmilu.com`
-   Device auto‑joined AD + Entra ID
-   Device auto‑enrolled to Intune

### Security Policy: BitLocker

-   Applied Intune disk encryption security profile:
    -   Require encryption ✅
    -   XTS‑AES256 ✅
    -   Key escrow to Entra ✅
    -   Silent enable ✅

### Validation Steps

  Check                            Result
  -------------------------------- | -----------------------
  Device object created in AD      | ✅ `OU=LabComputers`
  Device appears in Intune         | ✅
  Hybrid Azure AD Join confirmed   | `dsregcmd /status` ✅
  BitLocker enabled                | ✅
  Recovery key stored in Entra     | ✅

## Commands Used

### Check device join state

``` powershell
dsregcmd /status
```

### Trigger BitLocker manually (if needed)

``` powershell
Enable-BitLocker -MountPoint "C:" -UsedSpaceOnly -TpmProtector
```

### Reset device for re‑enrollment

``` powershell
systemreset -factoryreset
```

## What I Learned

-   Difference between **AAD Join vs Hybrid Join vs manual MDM**
-   Why offline domain join requires Autopilot OOBE
-   Device writes to AD only via **Intune Connector**
-   Importance of network line‑of‑sight to DC for hybrid login
-   Intune OSU + encryption policies automation
-   Enterprise‑grade BitLocker configuration using Intune
-   Troubleshooting Autopilot and WebView2 issues

## Accomplishments

Successfully: - Built a complete hybrid identity environment -
Integrated Autopilot with Intune & AD - Achieved fully automated hybrid
domain join - Implemented a Zero‑Touch deployment model - Applied
enterprise security controls (BitLocker) - Demonstrated end‑to‑end
device lifecycle onboarding
