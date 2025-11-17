# SAN Backup to Azure Lab  
Windows iSCSI Target • FS01 File Server • MSP360 • Azure Blob Storage

## Overview
This lab demonstrates how to design and implement an on-premises style storage and disaster recovery workflow using Windows Server iSCSI Target (acting as a SAN), a file server consuming SAN storage, and a backup server sending data to Microsoft Azure. It simulates a small-scale enterprise environment and provides hands-on experience with hybrid cloud backup architecture.

---

# Phase 1: Build the SAN Storage Layer (SAN-SERVER)

### Objective
Create a SAN-like storage backend using Windows iSCSI Target to provide centralized storage for FS01.

### Steps
1. Install the **iSCSI Target Server** role on SAN-SERVER.  
2. Create a **VHDX virtual disk** to act as SAN storage.  
3. Create an **iSCSI Target** and map the disk to it.  
4. Retrieve FS01’s **IQN** from iSCSI Initiator.  
5. Authorize FS01’s IQN on SAN-SERVER.  
6. SAN-SERVER is now ready to serve disk storage over iSCSI.

### Purpose
To simulate SAN storage (like Nimble) by exposing block storage over the network for consumption by a file server.

---

# Phase 2: Configure FS01 and Attach SAN Storage

### Objective
Connect FS01 to the SAN virtual disk and configure it as a usable data volume.

### Steps
1. Install and open **iSCSI Initiator** on FS01.  
2. Add SAN-SERVER’s IP as a discovery portal.  
3. Connect to the exposed **FS01 target**.  
4. Open **Disk Management**, initialize the new disk as **GPT**.  
5. Create a **New Simple Volume**, assign drive letter **E:**, and format NTFS.  
6. Create data folders under:
   ```
   E:\CompanyData
   ```

### Purpose
FS01 now uses storage that physically resides on SAN-SERVER, replicating enterprise shared-storage architecture.

---

# Phase 3: Deploy BK01 (Backup Server)

### Objective
Introduce a dedicated backup server for managing backup operations without affecting production performance.

### Steps
1. Provision BK01 and join it to the domain.  
2. Install **MSP360 Backup**.  
3. Ensure BK01 can access:
   ```
   \\FS01\CompanyData
   ```

### Purpose
Separate compute and backup roles to reflect standard enterprise backup design patterns.

---

# Phase 4: Configure Azure Storage as Cloud Backup Target

### Objective
Create durable offsite cloud storage for backup data.

### Steps
1. Create an Azure **Resource Group**.  
2. Create an **Azure Storage Account** (LRS).  
3. Create a **Blob container** named `backups`.  
4. Retrieve **Access Key** for MSP360 authentication.

### Purpose
Provide a cloud-based destination for MSP360 to store backup data securely and durably.

---

# Phase 5: Implement Backup Workflow with MSP360

### Objective
Configure and run an end-to-end backup from FS01 to Azure via BK01.

### Steps
1. In MSP360, add Azure using:
   - Storage account name  
   - Access key  
2. Create a **File Backup Plan** targeting FS01’s:
   ```
   \\FS01\CompanyData
   ```
3. Configure:
   - Compression  
   - Encryption (optional)  
   - Retention  
   - Schedule  
4. Execute the backup plan.  
5. Confirm that backup data appears in the Azure Blob container.

### Purpose
Establish a reliable backup pipeline from on-premises storage to Azure cloud.

---

# Phase 6: Disaster Recovery (DR) Test

### Objective
Validate that backups can be used to restore data during a loss event.

### Steps
1. On FS01, delete sample files or entire subfolders under:
   ```
   E:\CompanyData
   ```
2. On BK01, create a **Restore Plan** in MSP360.  
3. Select Azure as the source.  
4. Restore deleted data back to FS01.  
5. Verify data integrity after restoration.

### Purpose
Demonstrate full DR capability and validate restore readiness.

---

# Tools and Services Used

### Windows Server Components
- iSCSI Target Server (SAN-SERVER)  
- iSCSI Initiator (FS01)  
- File Server Role  
- NTFS Volume Management  
- Active Directory Domain Services (cloudwithmilu.com)

### Backup & Cloud
- MSP360 Backup for Windows Server  
- Azure Storage Account (Blob container)  
- Azure Access Keys  
- Network File Shares (SMB)

---

# Timeline Summary (Quick Reference)

1. **Build SAN Storage**  
   Create VHDX → expose via iSCSI Target → authorize FS01.

2. **Attach SAN Disk to FS01**  
   Connect via iSCSI Initiator → initialize disk → mount as E:.

3. **Configure BK01**  
   Install MSP360 → validate FS01 share access.

4. **Build Azure Backup Target**  
   Create Storage Account → container → access key.

5. **Backup Workflow**  
   MSP360 backup plan → full run → verify in Azure.

6. **Disaster Recovery Test**  
   Delete data → restore via MSP360 → validate.

---

# What We Accomplished
- Constructed a SAN-like storage backend using Windows Server.  
- Provisioned a file server consuming centralized block storage.  
- Implemented cloud backups using MSP360 and Azure.  
- Validated complete recovery of lost data.  
- Built a realistic, enterprise-style DR workflow.

---

# What We Learned
- How SAN/iSCSI architecture works in practice.  
- How file servers consume remote block storage.  
- How backup servers orchestrate cloud backups.  
- How Azure Blob Storage fits into hybrid DR strategy.  
- How to perform reliable backups and restores in real environments.

This lab demonstrates a full on-premises-to-cloud backup and DR pipeline using pure Windows Server components and Azure Blob Storage.
