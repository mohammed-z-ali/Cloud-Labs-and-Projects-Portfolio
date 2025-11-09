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
## 🧭 Implementation Timeline & Detailed Steps

### **Phase 1 – Azure VM & Domain Setup**
**Goal:** Prepare the base environment for Exchange installation.  
**Steps Performed:**
1. **Created Azure Resource Group** → *RG-Exchange-Lab*  
2. **Deployed VM:**
   - Name: `EX01`
   - Image: Windows Server 2022 Datacenter  
   - Size: Standard D2s_v3  
   - Static IP + Public IP  
3. **Configured NSG Rules:**
   | Protocol | Port | Source | Purpose |
   |-----------|------|---------|----------|
   | RDP | 3389 | Admin IP only | Remote management |
   | HTTPS | 443 | *Any* | OWA/ECP access |
   | SMTP | 25 | *Any* | Mail transport |
   | ICMP | Allow | *Any* | Ping test |
4. Connected via **RDP** → renamed machine → joined to **cloudwithmilu.com** domain.
5. Installed Windows updates, rebooted.

---

### **Phase 2 – Install Exchange Server 2019 CU12**
**Goal:** Deploy Exchange Mailbox role on the VM.  
**Steps Performed:**
1. Installed **Exchange prerequisites**:
   ```powershell
   Install-WindowsFeature Server-Media-Foundation, RSAT-ADDS, Web-Server, Web-Mgmt-Console, NET-Framework-45-Features
   ```
   Then installed:
   - UCMA 4.0 Runtime  
   - Visual C++ 2013 Redistributable (x64)  
   - IIS URL Rewrite Module  
2. Prepared AD schema:
   ```powershell
   Setup.exe /PrepareSchema /IAcceptExchangeServerLicenseTerms_DiagnosticDataON
   Setup.exe /PrepareAD /OrganizationName:"CloudWithMilu"
   ```
3. Ran setup wizard → selected **Mailbox Role** only → finished setup.
4. Validated installation:
   ```powershell
   Get-ExchangeServer
   Get-Service *exchange* | Where {$_.Status -eq "Running"}
   ```

---

### **Phase 3 – SSL Certificate Configuration**
**Goal:** Secure all Exchange services (IIS, SMTP, IMAP, POP) with valid SSL/TLS.  
**Steps Performed:**
1. Purchased SSL for **mail.cloudwithmilu.com** from *SSLS.com* and downloaded `.crt` + `.ca-bundle`.  
2. Combined into `.pfx` using **OpenSSL**:
   ```bash
   openssl pkcs12 -export -out mail_cloudwithmilu_com_fixed.pfx -inkey private.key -in certificate.crt -certfile ca-bundle.crt
   ```
3. Copied `.pfx` to Exchange VM (`C:\Certs`).  
4. Imported into Exchange:
   ```powershell
   Import-ExchangeCertificate -FileData ([Byte[]](Get-Content "C:\Certs\mail_cloudwithmilu_com_fixed.pfx" -Encoding byte -ReadCount 0)) -Password (ConvertTo-SecureString "XXXX" -AsPlainText -Force)
   ```
5. Assigned to services:
   ```powershell
   Enable-ExchangeCertificate -Thumbprint <Thumb> -Services IIS,SMTP -Force
   iisreset /noforce
   Restart-Service MSExchangeTransport
   Restart-Service MSExchangeFrontEndTransport
   ```
6. **Verified padlock:**
   - Browser → `https://mail.cloudwithmilu.com/owa` → ✅ Padlock shows valid CA.
   - Checked certificate chain → CA root trusted.
7. Tested SMTP TLS:
   ```bash
   openssl s_client -connect mail.cloudwithmilu.com:25 -starttls smtp
   ```
   Output showed `Verify return code: 0 (ok)` → TLS working.

---

### **Phase 4 – DNS & Domain Authentication**
**Goal:** Configure SPF, DKIM, and DMARC for outbound mail trust.  
**Steps Performed:**
1. Logged into **GoDaddy DNS** → selected domain `cloudwithmilu.com`.  
2. Added:
   - **SPF**:  
     `v=spf1 include:spf.protection.outlook.com -all`
   - **DKIM** (after enabling in M365):  
     ```
     selector1._domainkey → selector1-cloudwithmilu-com._domainkey.cloudwithmilu.onmicrosoft.com
     selector2._domainkey → selector2-cloudwithmilu-com._domainkey.cloudwithmilu.onmicrosoft.com
     ```
   - **DMARC**:  
     `v=DMARC1; p=none; rua=mailto:dmarc@cloudwithmilu.com; ruf=mailto:dmarc@cloudwithmilu.com; pct=100`
3. Validated propagation:
   ```powershell
   nslookup -type=txt _dmarc.cloudwithmilu.com
   ```
4. Verified all 3 with [MxToolbox.com](https://mxtoolbox.com).

---

### **Phase 5 – Microsoft 365 & Entra ID Configuration**
**Goal:** Prepare Microsoft 365 tenant for hybrid connection.  
**Steps Performed:**
1. Added custom domain **cloudwithmilu.com** in M365 Admin Center → verified via TXT record.  
2. Created **hybridadmin** account:
   - Roles: Global Admin + Exchange Admin  
   - License: Microsoft 365 E3  
3. Enabled **Entra ID Premium P2**, then created Conditional Access Policy:
   - Require MFA for all users  
   - Exclude `hybridadmin`  
4. Verified login for `hybridadmin` → no MFA prompt.

---

### **Phase 6 – Hybrid Configuration Wizard (HCW)**
**Goal:** Establish hybrid trust and mail flow between on-prem and Exchange Online.  
**Steps Performed:**
1. Downloaded **Hybrid Configuration Wizard** from Microsoft.  
2. Signed in as `hybridadmin`.  
3. Selected:
   - **Full Hybrid**  
   - Mail flow both ways  
   - Automatically configure send/receive connectors  
4. Validated connectivity to both Exchange environments.  
5. HCW failed initially → fixed by:
   - Assigning mailbox license to `hybridadmin`  
   - Re-running wizard  
6. Post-success:
   - Verified connectors:  
     `Outbound to Office 365` & `Inbound from Office 365`  
   - Tested mail flow both directions.  
   - Verified TLS headers in message source.

---

### **Phase 7 – Testing & Validation**
**Goal:** Confirm complete hybrid functionality.  
**Steps Performed:**
1. Test user mailbox creation (on-prem + online).  
2. Sent cross-environment emails (both directions).  
3. Verified:
   - SPF → Pass  
   - DKIM → Pass  
   - DMARC → Pass  
4. Opened OWA and ECP URLs → confirmed SSL padlock and successful login.  
5. Used:
   ```powershell
   Get-ReceiveConnector
   Get-SendConnector
   Get-HybridConfiguration
   ```
6. Ran `Get-MessageTrackingLog` to trace mail flow.
7. Completed end-to-end TLS and hybrid trust validation.

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
