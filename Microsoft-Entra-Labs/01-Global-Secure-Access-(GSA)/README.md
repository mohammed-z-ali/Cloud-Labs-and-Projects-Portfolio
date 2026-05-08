# Microsoft Entra Global Secure Access — Full Lab Implementation


## 📌 Project Overview

This project documents the end-to-end implementation of **Microsoft Entra Global Secure Access (GSA)** in a cloud-only environment. It covers the full Security Service Edge (SSE) stack including **Microsoft Entra Internet Access** (web filtering/SWG) and **Microsoft Entra Private Access** (Zero Trust Network Access / VPN replacement).

This lab demonstrates real-world enterprise skills applicable to Cloud Engineer and Cloud Administrator roles, including identity-driven network security, Zero Trust architecture, and secure remote access without a traditional VPN.

---

## 🏗️ Architecture

```
User Device (GSA Client Installed)
         │
         ▼
Microsoft Entra Global Secure Access (SSE Cloud)
         │
         ├──── Internet Access (Web Filtering / SWG)
         │              │
         │              ▼
         │     Web Content Filtering Policies
         │     Security Profiles
         │     Conditional Access Enforcement
         │
         └──── Private Access (ZTNA / VPN Replacement)
                        │
                        ▼
              Private Network Connector
                        │
                        ▼
              Azure VM (Private Resource)
              Private IP — No Public Exposure
```

---

## 🛠️ Technologies Used

| Technology | Purpose |
|---|---|
| Microsoft Entra ID (P1) | Identity platform & Conditional Access |
| Microsoft Entra Global Secure Access | SSE / SASE platform |
| Microsoft Entra Internet Access | Secure Web Gateway / Web Filtering |
| Microsoft Entra Private Access | VPN Replacement |
| Microsoft Azure | Hosted VM as private resource |
| Windows 11 (Entra Joined) | Client device for testing |
| Azure Virtual Machine (Windows Server 2022) | Private network connector host |

---

## ✅ What Was Implemented

### 1. Prerequisites & Setup
- Verified Microsoft Entra ID P1 licensing
- Assigned Global Secure Access Administrator, Conditional Access Administrator, and Global Administrator roles
- Activated Global Secure Access in the Microsoft Entra admin center

### 2. Traffic Forwarding Profiles
- Enabled **Microsoft 365 traffic profile** — routes M365 traffic through GSA for security and performance
- Enabled **Internet Access profile** — routes general internet traffic through GSA
- Enabled **Private Access profile** — routes private resource traffic through GSA tunnel
- Assigned all profiles to users via user/group assignments

### 3. GSA Client Deployment
- Downloaded and installed the Global Secure Access Windows client
- Authenticated client with Entra ID credentials on an Entra-joined Windows 11 device
- Verified all 3 channels (Microsoft 365, Internet Access, Private Access) showing Active and green

### 4. Internet Access — Web Content Filtering
- Created **Web Content Filtering policy** blocking Social Networking category
- Added **FQDN-based rules** with wildcard support (e.g. `*.facebook.com`, `*.instagram.com`) for granular blocking
- Created **Security Profile** and linked filtering policies to it
- Created **Conditional Access policy** targeting "All internet resources with Global Secure Access" and enforcing the Security Profile via Session controls
- Enabled **Baseline Profile** as safety net for traffic bypassing CA policies
- Verified blocking in traffic logs — confirmed user, device, IP, location, destination, and action all visible

### 5. Private Access — VPN Replacement
- Provisioned an **Azure Virtual Machine** (Windows Server 2022, B2s) as the private resource
- Created dedicated **Resource Group** (`GSA-Lab`) in Azure
- Downloaded and installed the **Private Network Connector** on the Azure VM
- Verified connector status shows **Active** in the Entra admin center
- Created an **Enterprise Application** in GSA representing the private resource
- Configured **Application Segment** targeting VM's private IP on port 3389 (RDP/TCP)
- Assigned users to the private app
- Created **Conditional Access policy** for Private Access with MFA requirement

### 6. Testing & Validation
- ✅ Confirmed internet traffic filtering — social media sites blocked via category and FQDN rules
- ✅ Confirmed Private Access tunnel — RDP connected to VM's **private IP** from outside Azure **without a VPN**
- ✅ Confirmed GSA dependency — disabling GSA client broke private IP connectivity; re-enabling restored it
- ✅ Reviewed Traffic Logs — confirmed visibility of user, device, IP address, geographic location, application, destination, and enforcement action
- ✅ Troubleshot real-world issues including DNS caching, QUIC protocol bypass, user assignment gaps, and CDN-based filtering inconsistency

---

## 🔍 Key Troubleshooting Scenarios Resolved

| Issue | Root Cause | Resolution |
|---|---|---|
| GSA client showed "Disabled by organization" | Users not assigned to traffic forwarding profiles | Assigned "All Users" to Internet and Private Access profiles |
| Web filtering not enforcing | CA policy missing Session control linking Security Profile | Added "Use Global Secure Access security profile" session control |
| No traffic appearing in logs | QUIC protocol bypassing GSA inspection | Disabled experimental QUIC in browser flags |
| Inconsistent social media blocking | DNS caching + CDN routing | Flushed DNS cache, added wildcard FQDN rules |
| Private Access traffic option missing in CA | Updated Entra portal UI | Used "Select specific resources" to target the Private Access enterprise app |

---

## 📊 Evidence & Proof of Work

The following screenshots are included in the `/evidence` folder of this repository:

| Screenshot | What it Shows |
|---|---|
| `01-gsa-activated.png` | GSA dashboard activated with Connect, Applications, Secure, Monitor menus |
| `02-traffic-profiles-enabled.png` | All 3 traffic forwarding profiles enabled |
| `03-gsa-client-connected.png` | GSA client showing green/Connected status with all channels active |
| `04-web-filtering-policy.png` | Block - Social Media policy with category and FQDN rules |
| `05-security-profile.png` | Security profile linked to filtering policy with block action |
| `06-ca-policy-internet.png` | CA policy for Internet Access with Session control and Security Profile |
| `07-sites-blocked.png` | Browser showing blocked pages for facebook.com / youtube.com |
| `08-connector-active.png` | Private Network Connector showing Active status in Entra portal |
| `09-private-app-config.png` | Enterprise app configured with VM private IP and port 3389 |
| `10-rdp-connected-private-ip.png` | RDP session connected to VM via private IP through GSA tunnel |
| `11-rdp-failed-gsa-disabled.png` | RDP connection failed when GSA client was disabled (proves tunnel dependency) |
| `12-traffic-logs.png` | Traffic logs showing user, device, IP, location, destination, and action |

---

## 💡 Key Concepts Demonstrated

- **Zero Trust Network Access (ZTNA)** — identity-verified, per-app access replacing broad VPN tunnels
- **Secure Web Gateway (SWG)** — cloud-delivered web filtering with category and FQDN-based rules
- **Security Service Edge (SSE)** — converged network and security delivered from the cloud
- **Conditional Access** — identity-aware policy enforcement applied to network traffic
- **Private Network Connector** — lightweight agent creating outbound-only secure tunnel (no inbound firewall rules needed)
- **Traffic Log Analysis** — monitoring user, device, location, and destination visibility across all network traffic

---

## 🧱 Environment Details

| Component | Details |
|---|---|
| Tenant Type | Cloud-only (no on-premises AD) |
| License | Microsoft Entra ID P1 |
| Client OS | Windows 11, Entra Joined |
| Connector Host | Azure VM — Windows Server 2022 Datacenter |
| VM Size | Standard B2s |
| Azure Region | East US |
| Connector Status | Active |

---

## 📁 Repository Structure

```
📦 entra-global-secure-access-lab
 ┣ 📂 evidence
 ┃ ┣ 📸 01-gsa-activated.png
 ┃ ┣ 📸 02-traffic-profiles-enabled.png
 ┃ ┣ 📸 03-gsa-client-connected.png
 ┃ ┣ 📸 04-web-filtering-policy.png
 ┃ ┣ 📸 05-security-profile.png
 ┃ ┣ 📸 06-ca-policy-internet.png
 ┃ ┣ 📸 07-sites-blocked.png
 ┃ ┣ 📸 08-connector-active.png
 ┃ ┣ 📸 09-private-app-config.png
 ┃ ┣ 📸 10-rdp-connected-private-ip.png
 ┃ ┣ 📸 11-rdp-failed-gsa-disabled.mp4
 ┃ ┗ 📸 12-traffic-logs.png
 ┣ 📄 README.md
```

---

## Evidence & Proof of Work

### 1️⃣ GSA Dashboard Activated
[![GSA Activated](evidence/01-gsa-activated.png)](evidence/01-gsa-activated.png)
*GSA dashboard activated with Connect, Applications, Secure, Monitor menus*

---

### 2-Traffic Forwarding Profile Enabled
[![Traffic Profiles](evidence/02-traffic-profiles-enabled.png)](evidence/02-traffic-profiles-enabled.png)
*All 3 traffic forwarding profiles enabled — Microsoft 365, Internet Access, Private Access*

---

### 3️⃣ GSA Client Connected
[![GSA Client](evidence/03-gsa-client-connected.png)](evidence/03-gsa-client-connected.png)
*GSA client showing green/Connected status with all channels active*

---

### 4️⃣ Web Content Filtering Policy
[![Web Filtering](evidence/04-web-filtering-policy.png)](evidence/04-web-filtering-policy.png)
*Block - Social Media policy with category and wildcard FQDN rules*

---

### 5️⃣ Security Profile Linked to Policy
[![Security Profile](evidence/05-security-profile.png)](evidence/05-security-profile.png)
*Security profile linked to filtering policy with block action enabled*

---

### 6️⃣ Conditional Access Policy — Internet Access
[![CA Policy](evidence/06-ca-policy-internet.png)](evidence/06-ca-policy-internet.png)
*CA policy for Internet Access with Session control enforcing Security Profile*

---

### 7️⃣ Sites Successfully Blocked
[![Sites Blocked](evidence/07-sites-blocked.png)](evidence/07-sites-blocked.png)
*Browser showing blocked pages for facebook.com / youtube.com*

---

### 8️⃣ Private Network Connector Active
[![Connector Active](evidence/08-connector-active.png)](evidence/08-connector-active.png)
*Private Network Connector showing Active status in Entra portal*

---

### 9️⃣ Private App Configuration
[![Private App](evidence/09-private-app-config.png)](evidence/09-private-app-config.png)
*Enterprise app configured with VM private IP and port 3389 (RDP/TCP)*

---

### 🔟 RDP Connected via Private IP Through GSA Tunnel
[![RDP Connected](evidence/10-rdp-connected-private-ip.png)](evidence/10-rdp-connected-private-ip.png)
*RDP session connected to VM via private IP — VPN replacement working*

---

### 1️⃣1️⃣ RDP Failed When GSA Disabled
[![RDP Failed](evidence/11-rdp-failed-gsa-disabled.mp4)](evidence/11-rdp-failed-gsa-disabled.mp4)
*RDP connection failed when GSA client was disabled — proves tunnel dependency*

---

### 1️⃣2️⃣ Traffic Logs — Full Visibility
[![Traffic Logs](evidence/12-traffic-logs.png)](evidence/12-traffic-logs.png)
*Traffic logs showing user, device, IP address, geographic location, destination, and enforcement action*


## 🎯 Skills This Project Demonstrates

For **Cloud Engineer / Cloud Administrator** roles:

- ✅ Microsoft Entra ID administration
- ✅ Azure Virtual Machine provisioning and management
- ✅ Zero Trust security architecture implementation
- ✅ Conditional Access policy design and enforcement
- ✅ Network security policy configuration (SWG, ZTNA)
- ✅ Cloud-native VPN replacement (Private Access)
- ✅ Security monitoring and log analysis
- ✅ Real-world troubleshooting of identity and network issues
- ✅ Microsoft SSE/SASE platform deployment

---

*This lab was built in a live Microsoft Entra tenant with real traffic, real blocking enforcement, and a real Azure VM — not a simulation.*
