# Intune Project: Automated BitLocker Deployment for Corporate Devices

## 1. Project Summary
The goal of this project was to implement a **"Zero-Touch" Silent Encryption** policy for corporate Windows devices. The objective was to ensure that every device enrolled in Microsoft Intune is automatically encrypted using BitLocker without requiring administrative rights from the user or causing any disruption to their workflow.

## 2. Implementation Details
We configured a **Disk Encryption Profile** in Microsoft Intune targeting the `Corporate Windows Devices` group. The policy was specifically tuned to handle **Standard User** accounts.

### Key Policy Settings:
* **Silent Trigger:** Set `Allow Warning For Other Disk Encryption` to **Disabled**. This prevents the "Standard User" from seeing admin prompts they cannot clear.
* **Standard User Rights:** Enabled `Allow Standard User Encryption`, allowing non-admins to trigger the BitLocker service.
* **Hardware Auth:** Configured `TPM Startup` as **Required** while setting PIN and Key options to **Do Not Allow**. This forces the computer to use the motherboard's TPM chip silently rather than asking the user for a PIN.
* **Safety Lock:** Enabled `Do not enable BitLocker until recovery information is stored`. This ensures a device never encrypts unless the recovery key is safely backed up to Entra ID first.

## 3. Technical Learnings
During this lab, we identified and resolved several critical encryption hurdles:

* **Policy Conflicts:** Learned that overlapping "Security Baselines" and "Configuration Profiles" can cause a `Conflict` status. Resolution requires identifying the source and setting redundant policies to "Not Configured."
* **Cipher Strength:** Implemented **XTS-AES 256-bit** for internal drives (modern security standard) and **AES-CBC 256-bit** for removable drives (for maximum compatibility across different systems).
* **Escrow Logic:** Understood that the "Succeeded" status in Intune confirms policy delivery, while the "Recovery Keys" tab confirms the actual completion of the security handshake between the client and the cloud.
* **Password Rotation:** Enabled automatic rotation so that if a recovery key is ever exposed to a user, it is refreshed immediately upon use.

## 4. Verification of Success
The deployment was verified by checking the device properties in the Intune Portal. The screenshot below confirms that the **48-digit Recovery Key** has been successfully escrowed to Microsoft Entra ID.

[BitLocker Success Screenshot](./Screenshots/BitLocker Key in Recovery Key tab.png)

