# Automated Wi-Fi Provisioning & Policy Deployment

## Project Overview
This project demonstrates the automated deployment of Wi-Fi configuration profiles using Microsoft Intune (Endpoint Manager). 
The goal of this lab was to eliminate manual network onboarding by ensuring managed devices automatically connect to the appropriate SSID with the correct security credentials upon entering range.

This repository covers:
1.  **Hands-on Lab:** Deployment of a Personal/Small Office (PSK) Wi-Fi profile.
2.  **Validation:** Live testing on a BYOD (Bring Your Own Device) endpoint.
3.  **Architecture:** Professional roadmap for Enterprise (Certificate-based) Wi-Fi.

---

## Lab 1: Personal/Home Wi-Fi Configuration (Completed)
In this phase, I configured a policy to connect managed devices to a private network (Optimum WiFi) using a Pre-Shared Key (PSK). This simulates a small office or remote-work environment.

### Technical Configuration:
- **Platform:** Windows 10/11
- **Profile Type:** Configuration Profile (Wi-Fi Template)
- **Security Type:** WPA2-Personal
- **Connectivity:** - **Auto-Connect:** Enabled.
  - **Hidden SSID Support:** Configured to connect even if the network is not broadcasting.

### Steps Performed:
1.  **Policy Creation:** Navigated to `Devices > Configuration` and created a new Wi-Fi template.
2.  **Parameter Definition:** Defined the exact SSID and entered the Pre-Shared Key (PSK).
3.  **Targeting:** Assigned the policy to a specific Entra ID Device Group.
4.  **Verification:** Forced a manual sync on the endpoint to pull the new configuration.

### Validation & Results (BYOD Test)
To confirm the policy was effective, I performed a live test using an enrolled BYOD laptop:
- **Scenario:** The device was brought within range of the home Wi-Fi after the policy was synced.
- **Result:** ✅ **Success.** The device identified the SSID and auto-connected immediately.
- **Evidence:** The Wi-Fi settings on the device displayed the "Managed by your organization" status, and no manual password entry was required. (View [Screenshots](./Screenshots/)).

---

## Lab 2: Corporate/Enterprise Architecture (Research & Planning)
While I utilized a PSK for the initial lab, I researched and documented the requirements for a **Zero-Trust Corporate environment**. In a professional setting, individual passwords are replaced by digital identities.

### The 3-Tier Deployment Strategy:
To implement Enterprise Wi-Fi (**EAP-TLS**), the following infrastructure is required:

1.  **Trusted Root Certificate:** A policy to deploy the Organization's Root CA certificate so the device trusts the network.
2.  **SCEP/PKCS Certificate Profile:** Issuing a unique digital "ID Card" to the device for authentication.
3.  **Enterprise Wi-Fi Profile:** - **EAP Type:** EAP-TLS.
    - **Authentication:** Certificate-based (no passwords).
    - **Cloud RADIUS:** Integration with providers like SCEPman or RADIUSaaS for a serverless experience.

---

## Learning Outcomes
- **MDM Proficiency:** Gained experience navigating the Microsoft Intune Admin Center.
- **Policy Life Cycle:** Learned the process of creating, assigning, and verifying configuration profiles.
- **Identity Management:** Understand the shift from PSK (Shared Passwords) to Certificate-based (Enterprise) security.
- **Troubleshooting:** Learned to verify policy arrival via Windows Sync settings and Event Viewer.

---

## Future Roadmap
- [ ] Implement a Cloud RADIUS trial to test full EAP-TLS handshakes.
- [ ] Configure "Compliance-based" Wi-Fi (only allow Wi-Fi access if the device is encrypted and up-to-date).
- [ ] Automate profile assignment using Dynamic Device Groups based on device naming conventions.
