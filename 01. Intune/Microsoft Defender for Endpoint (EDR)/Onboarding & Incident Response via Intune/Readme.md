# Lab: Microsoft Defender for Endpoint (MDE) Onboarding & Incident Response via Intune

## Project Overview
This project demonstrates the end-to-end process of onboarding an Entra ID (AAD) joined device to **Microsoft Defender for Endpoint (MDE)** using **Microsoft Intune**. The goal was to move beyond simple configuration and perform active security validation, including triggering alerts, investigating device timelines, and performing remote network isolation.

## Technical Tasks Accomplished
* **Service-to-Service Integration:** Enabled the handshake between the Microsoft Defender portal and Intune Admin Center.
* **EDR Policy Deployment:** Configured and deployed an **Endpoint Detection and Response (EDR)** profile to automate sensor activation on Windows devices.
* **Local Administrator Management:** Overcame "Standard User" restrictions on an AAD-joined device by deploying a custom Intune policy to create a **Local Administrator account**, enabling advanced troubleshooting and manual alert triggering.
* **Manual Alert Validation:** Successfully executed a PowerShell-based "Canary" attack string to verify the EDR sensor's ability to capture and report suspicious behavior.
* **Device Isolation & Remediation:** Performed a remote **Network Isolation** of the compromised host via the Defender Portal to contain a simulated threat, followed by a successful restoration of connectivity.

## Investigation & Validation
### The Defender Portal Timeline
After triggering the alert, I utilized the **Device Timeline** feature in the Microsoft Defender portal. This allowed me to:
* Identify the exact timestamp at which the PowerShell script was executed.
* Observe the process tree showing `cmd.exe` spawning `powershell.exe`.
* Verify the **Logged-on Users** list, which accurately reflected both the **Standard User** and the newly created **Local Admin** account used for the lab.

### Network Isolation Test
To test the "Containment" capability:
1.  **Isolation Triggered:** Network traffic was cut off at the driver level.
2.  **Observation:** The device maintained "Bars" (signal), but all browser requests and pings failed, proving the Defender firewall was active.
3.  **Restoration:** Connectivity was instantly restored once "Release from Isolation" was confirmed in the portal.


## Tools Used
* **Microsoft Intune:** Policy deployment and Local Admin creation.
* **Microsoft Defender for Endpoint:** Security monitoring and Incident Response.
* **PowerShell:** Advanced scripting for threat simulation.
* **Microsoft Entra ID:** Identity and device management.
