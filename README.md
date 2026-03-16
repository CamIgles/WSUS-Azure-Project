# Windows Server 2022: Install, Configure, and Deploy Windows Server Update Services (WSUS) on Microsoft Azure

---

## Project Summary

**What This Project Is**
I deployed a Windows Server 2022 virtual machine in Microsoft Azure and installed/configured Windows Server Update Services (WSUS) to centrally manage Microsoft updates. This project demonstrates how to set up an update server in the cloud, synchronize with Microsoft Update, approve updates for testing, and configure client computers to receive updates. This simulates what IT professionals do to control patching and save bandwidth in enterprise environments.

**Languages Used**
- PowerShell - Used for server configuration and cleanup scripts

**Environments Used**
- Microsoft Azure (Cloud Platform)
- Windows Server 2022 (WSUS Server)
- Windows 10 Pro (Client Machines)
- Azure Virtual Network

**Technologies/Applications/Services Used**
- Microsoft Azure Virtual Machines
- Windows Server Update Services (WSUS)
- Internet Information Services (IIS)
- Windows Update Service
- Group Policy
- Azure Networking (Virtual Networks, Network Security Groups)
- PowerShell

---

## Demonstration

### Phase 1: Azure Environment Setup

**Step 1: Create Azure Free Account**
I signed up for an Azure free account at portal.azure.com. The free account includes $200 credit for the first 30 days and 12 months of free services.

**Step 2: Create Resource Group**
In the Azure portal, I created a new resource group named "WSUS-Lab-RG" to organize all resources for this project.

**Step 3: Create Virtual Network**
I created a virtual network with:
- Name: WSUS-VNet
- Address space: 10.0.0.0/16
- Subnet: Default (10.0.0.0/24)

**Step 4: Create Windows Server 2022 VM**

In the Azure portal, I created a virtual machine with these settings:
- Resource group: WSUS-Lab-RG
- Virtual machine name: WSUS-Server
- Image: Windows Server 2022 Datacenter
- Size: Standard_B2ats_v2 (2 vCPUs, 1 GB RAM) - Free tier eligible
- Administrator account: azureadmin / [password]
- Inbound ports: Allow RDP (3389)

![Azure VM Creation](screenshots/01-azure-vm-creation.png)
*Creating the Windows Server 2022 VM in Azure portal*

**Step 5: Configure Disk for Free Tier**
I ensured the OS disk was set to 64 GB to stay within free tier limits.

**Step 6: Review and Create**
Clicked "Review + create" and then "Create" to deploy the VM.

![VM Deployment](screenshots/02-vm-deployment.png)
*VM deployment in progress and complete*

---

### Phase 2: Connect to Azure VM

**Step 7: Connect via RDP**

After deployment, I connected to the VM using Remote Desktop:
- Downloaded the RDP file from Azure portal
- Opened the file and clicked "Connect"
- Entered the administrator credentials

![RDP Connect](screenshots/03-rdp-connect.png)
*Connecting to the Azure VM via Remote Desktop*

---

### Phase 3: WSUS Server Installation

**Step 8: Add WSUS Server Role**
In Server Manager, I clicked "Manage" → "Add Roles and Features" and selected "Windows Server Update Services".

![Add WSUS Role](screenshots/04-add-wsus-role.png)
*Selecting WSUS role in Add Roles wizard*

**Step 9: Select Role Services**
I selected:
- WID Database (Windows Internal Database)
- WSUS Services

![Role Services](screenshots/05-role-services.png)
*Selecting WID Database and WSUS Services*

**Step 10: Complete Installation**
I completed the wizard and clicked "Install". The installation took about 10 minutes.

![Install Complete](screenshots/06-install-complete.png)
*WSUS role installation completed successfully*

---

### Phase 4: WSUS Configuration

**Step 11: Launch WSUS Configuration Wizard**
After installation, the WSUS configuration wizard opened automatically.

![Configuration Wizard](screenshots/07-config-wizard.png)
*WSUS configuration wizard after installation*

**Step 12: Choose Upstream Server**
I selected "Synchronize from Microsoft Update" to download updates directly from Microsoft.

**Step 13: Choose Languages**
I selected "English" only to save disk space and bandwidth.

**Step 14: Select Products**

I chose these products to sync:
- Windows 10 and later updates
- Windows Server 2022
- Microsoft 365 Apps
- .NET Framework

![Products](screenshots/08-products.png)
*Selecting which products to sync updates for*

**Step 15: Select Classifications**

I selected these update types:
- Critical Updates
- Security Updates
- Update Rollups
- Service Packs

![Classifications](screenshots/09-classifications.png)
*Selecting update classifications to download*

**Step 16: Begin Initial Synchronization**
I clicked "Begin initial synchronization" and waited for updates to download. This took approximately 45 minutes.

![Initial Sync](screenshots/10-initial-sync.png)
*Initial synchronization in progress*

---

### Phase 5: Post-Sync Configuration

**Step 17: Review Downloaded Updates**
After sync completed, I opened the WSUS console and saw over 1,200 updates available.

![WSUS Console](screenshots/11-wsus-console.png)
*WSUS console showing downloaded updates*

**Step 18: Create Computer Groups**

I created target groups for different deployment rings:
- TEST - For testing updates first
- WORKSTATIONS - For general employee computers
- SERVERS - For server systems

![Computer Groups](screenshots/12-computer-groups.png)
*Creating computer groups in WSUS*

---

### Phase 6: Azure Networking for WSUS

**Step 19: Open WSUS Port in Azure Firewall**

For clients to connect to WSUS, I opened port 8530 in Azure:
- Went to VM → Networking
- Added inbound port rule
- Destination port: 8530
- Protocol: TCP
- Name: Allow-WSUS

![Azure Networking](screenshots/13-azure-networking.png)
*Adding inbound rule for WSUS port 8530 in Azure*

---

### Phase 7: Group Policy Configuration

**Step 20: Create WSUS Group Policy Object**
I opened Group Policy Management Console and created a new GPO named "WSUS-Client-Settings".

**Step 21: Configure WSUS Server Settings**

I configured these settings:
- Specify intranet Microsoft update service location: http://[WSUS-Server-IP]:8530
- Configure Automatic Updates: Enabled
- Set schedule: Install every Sunday at 3:00 AM

![Group Policy](screenshots/14-group-policy.png)
*Configuring WSUS settings in Group Policy*

---

### Phase 8: Client Testing

**Step 22: Force Group Policy Update on Client**
On a Windows 10 client, I ran:
