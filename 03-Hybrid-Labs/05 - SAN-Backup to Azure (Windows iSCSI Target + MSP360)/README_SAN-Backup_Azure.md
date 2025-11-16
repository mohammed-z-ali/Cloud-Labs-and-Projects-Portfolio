
# SAN Backup to Azure Lab (Windows iSCSI Target + MSP360)

## Summary
This lab simulates a real-world enterprise backup and disaster recovery environment using Windows Server iSCSI Target as a SAN, a dedicated file server consuming SAN storage, and a backup server sending data to Microsoft Azure.

## Objectives
- Build SAN-like storage using Windows iSCSI Target.
- Configure FS01 to consume SAN storage.
- Implement BK01 with MSP360 backup software.
- Connect to Azure Storage as backup destination.
- Perform successful backup and disaster recovery restore.

## Tools and Services Used
- Windows Server (SAN-SERVER, FS01, BK01)
- Windows iSCSI Target Server
- Windows iSCSI Initiator
- MSP360 Backup
- Azure Storage Account (Blob)
- Active Directory Domain: cloudwithmilu.com

## What We Did
1. Installed iSCSI Target Server role on SAN-SERVER.
2. Created a VHDX virtual disk and assigned it to an iSCSI target.
3. Added FS01’s IQN to authorize storage access.
4. Connected FS01 to SAN using iSCSI Initiator.
5. Initialized Disk 2 on FS01 as GPT, formatted NTFS, mounted as E:.
6. Created CompanyData folders on SAN-backed storage.
7. Installed MSP360 on BK01 and connected to Azure Storage.
8. Created and executed a backup plan for FS01 data to Azure.
9. Verified the backup in Azure Blob Storage.
10. Simulated disaster by deleting files on FS01.
11. Restored deleted data successfully using MSP360.

## What We Accomplished
- Built a functioning SAN-to-file-server storage architecture.
- Implemented Azure-based cloud backups.
- Performed a complete disaster recovery workflow.

## What We Learned
- How iSCSI SAN storage works and how servers consume external disks.
- How to manage SAN-backed storage volumes on Windows Server.
- How backup systems integrate with cloud platforms.
- How to plan and validate disaster recovery end-to-end.
