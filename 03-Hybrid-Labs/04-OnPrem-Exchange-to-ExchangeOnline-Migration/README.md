# On-Prem Exchange (AWS) to Exchange Online (Azure) Migration Lab

This lab demonstrates how to deploy an **on-prem Exchange Server on AWS** (to simulate a local data center) and **migrate mailboxes to Exchange Online in Azure** using a **hybrid configuration**.  
It also covers mail security setup, including **SPF, DKIM, and DMARC**, which are critical for preventing spoofing and ensuring email authenticity.

---

## Objectives

- Deploy Exchange Server 2019 on an **AWS EC2** instance (acting as on-premises Exchange).
- Configure **hybrid identity** with Azure using **Azure AD Connect**.
- Establish **hybrid mail flow** between on-prem and Exchange Online.
- Set up **SPF, DKIM, and DMARC** for domain authentication.
- Migrate user mailboxes to **Exchange Online (Azure)**.
- Decommission on-prem Exchange safely.
