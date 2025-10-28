# On-Prem Exchange 2019 → Exchange Online Hybrid Migration (CloudWithMilu.com)

## Project Overview
This lab demonstrates the full deployment and hybrid integration of **Microsoft Exchange Server 2019** (on-premises) with **Exchange Online (Microsoft 365)** using a custom domain — **cloudwithmilu.com** — hosted in **Microsoft Azure**.  
The goal was to simulate a real-world enterprise hybrid Exchange environment, secure all communications with **SSL/TLS**, and implement complete **domain authentication** using **SPF, DKIM, and DMARC**.

---

## Objective
- Deploy and configure Exchange Server 2019 CU12 on Azure VM.  
- Integrate on-prem Exchange 2019 with Exchange Online (Microsoft 365).  
- Configure SSL for HTTPS, SMTP, IMAP, and POP.  
- Implement SPF, DKIM, DMARC for secure email authentication.  
- Troubleshoot Hybrid Configuration Wizard (HCW) connectivity and authentication.  
- Validate hybrid mail flow and TLS encryption end-to-end.

---

## Tools & Environment
| Category | Tools / Services |
|-----------|------------------|
| Cloud | Microsoft Azure |
| OS | Windows Server 2022 |
| Mail Server | Microsoft Exchange Server 2019 CU12 |
| DNS Hosting | GoDaddy |
| SSL Provider | SSLS.com |
| Cloud Mail | Microsoft 365 Exchange Online |
| Security Features | SPF / DKIM / DMARC / TLS 1.2 |
| Management | PowerShell • Exchange Admin Center • IIS |
| References | [Exchange Server Prerequisites](https://learn.microsoft.com/en-us/Exchange/plan-and-deploy/prerequisites) |

---

## Hybrid Configuration Wizard (HCW) & Licensing Challenges
During hybrid setup, the **Hybrid Configuration Wizard** initially failed to authenticate to Exchange Online due to **MFA enforcement** on the admin account.  
To resolve this, the following steps were taken:

1. Created a dedicated account named **hybridadmin**.  
2. Assigned **Global Administrator** and **Exchange Administrator** roles.  
3. MFA could not be disabled per user because of tenant-wide security defaults.  
4. Purchased and activated **Microsoft Entra ID Premium P2** license to create a **Conditional Access Policy**:  
   - Required MFA for all users accessing cloud apps.  
   - Excluded `hybridadmin` from MFA enforcement so HCW could authenticate.  
5. HCW still did not proceed — Exchange Online required a licensed mailbox for connection.  
   → Purchased **Microsoft 365 E3 license** and assigned to `hybridadmin`.  
6. After license activation and policy adjustment, HCW completed successfully and established hybrid connectivity.

---

## SSL Configuration
### Certificate Import
```powershell
Import-ExchangeCertificate -FileData ([Byte[]](Get-Content -Path "C:\Users\Administrator\Desktop\cloudwithmilu.com_cert\mail_cloudwithmilu_com_fixed.pfx" -Encoding byte -ReadCount 0)) -Password (ConvertTo-SecureString -String "XXXX" -AsPlainText -Force)
```

### Enable Certificate for All Services
```powershell
Enable-ExchangeCertificate -Thumbprint 69AAD21578B43CC4E001C1C3FB142CXXXXXXXX -Services IIS,SMTP -Force
iisreset /noforce
Restart-Service MSExchangeTransport
Restart-Service MSExchangeFrontEndTransport
```

### Validation Tests
- **OWA:** <https://mail.cloudwithmilu.com/owa>  
- **ECP:** <https://mail.cloudwithmilu.com/ecp>  
- **TLS:** `openssl s_client -connect mail.cloudwithmilu.com:25 -starttls smtp`

Result: SSL successfully enabled for IIS, SMTP, POP, and IMAP. Browser padlock verified.

---

## Domain Authentication (SPF, DKIM, DMARC)
| Type | Record | Purpose | Example Value |
|------|---------|----------|---------------|
| **TXT** | `@` | SPF – Authorized senders | `v=spf1 include:spf.protection.outlook.com -all` |
| **CNAME** | `selector1._domainkey` | DKIM – Signature 1 | `selector1-cloudwithmilu-com._domainkey.cloudwithmilu.onmicrosoft.com` |
| **CNAME** | `selector2._domainkey` | DKIM – Signature 2 | `selector2-cloudwithmilu-com._domainkey.cloudwithmilu.onmicrosoft.com` |
| **TXT** | `_dmarc` | DMARC Policy | `v=DMARC1; p=none; rua=mailto:dmarc@cloudwithmilu.com; ruf=mailto:dmarc@cloudwithmilu.com; pct=100` |

### Summary
- **SPF** authorizes outbound mail servers (Exchange Online + on-prem).  
- **DKIM** cryptographically signs outgoing messages to ensure integrity.  
- **DMARC** enforces policy and monitors auth failures (`p=none` → `quarantine` → `reject`).  

Records were created in **GoDaddy DNS** and verified via **MxToolbox**.

---

## PowerShell Commands Used
```powershell
# View Exchange certificates
Get-ExchangeCertificate | fl Subject,Thumbprint,HasPrivateKey,Services,Status

# Enable certificate for services
Enable-ExchangeCertificate -Thumbprint "69AAD21578B43CC4E001C1C3FB142CFFA2A6ADA8" -Services "IIS,SMTP"

# Manage Transport Service
Get-Service MSExchangeTransport
Restart-Service MSExchangeTransport
Restart-Service MSExchangeFrontEndTransport

# Verify receive connectors
Get-ReceiveConnector "Default Frontend EX01" | fl Fqdn,Bindings

# Test TLS encryption
openssl s_client -connect mail.cloudwithmilu.com:25 -starttls smtp
```
All PowerShell scripts were executed directly on **EX01** within Exchange Management Shell.

---

## Challenges & Solutions
| Challenge | Description | Resolution |
|------------|-------------|-------------|
| CU12 Setup Conflict | Older Exchange module version | Installed latest CU12 and re-registered PowerShell snap-ins |
| PrivateKeyMissing | Imported cert lacked private key | Merged `.crt` + `.key` via OpenSSL → `.pfx` then re-imported |
| Cmdlets Missing | Module not loaded | `Add-PSSnapin Microsoft.Exchange.Management.PowerShell.SnapIn` |
| Transport Service Stuck | Service restart denied | Restarted IIS and Transport manually as Administrator |
| Browser Not Secure | Self-signed cert active | Imported CA-signed SSL and enabled IIS/SMTP |
| DNS Propagation Delay | DNS records not live immediately | Waited and validated with `nslookup` and MxToolbox |
| HCW Authentication Failure | MFA blocked tenant login | Created `hybridadmin`, applied Premium P2 Conditional Access policy to exclude it |
| HCW License Error | No Exchange Online mailbox available | Purchased E3 license and assigned to `hybridadmin` for successful hybrid link |

---

## What I Learned
- Deep understanding of **Exchange 2019 hybrid architecture**.  
- Practical experience in **SSL/TLS certificate management**.  
- Expertise in **SPF, DKIM, DMARC implementation**.  
- Hands-on with **Conditional Access Policies** in Entra ID.  
- Troubleshooting Exchange Transport, HCW, and authentication errors.  
- Leveraging **OpenSSL** to combine certificates and export PFX files.  
- Validating secure mail flow and TLS encryption end-to-end.

---

## Accomplishments
- Successfully deployed **Exchange 2019 on Azure** and migrated to Exchange Online.  
- Implemented enterprise-grade **SSL security** and DNS authentication.  
- Automated Exchange certificate management using PowerShell.  
- Resolved real-world hybrid auth and licensing challenges.  
- Completed a fully functional, secure **hybrid mail infrastructure**.


---

## Final Notes
This project mirrors an **enterprise Exchange hybrid deployment**, combining Azure infrastructure, Exchange Server, and Microsoft 365.  
It demonstrates advanced skills in **identity management, mail security, PowerShell automation, and hybrid migration engineering**.
