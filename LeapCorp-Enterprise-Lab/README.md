# Project: The Enterprise Sandbox (LeapCorp)

## **Overview**

This project demonstrates the design, deployment, and management of a fully functional, isolated Enterprise Network environment. By utilizing **Windows Server 2022** and **Active Directory Domain Services (AD DS)**, this lab simulates a small-sized corporate infrastructure. The goal is to showcase proficiency in identity management, centralized governance through Group Policy, and the automation of administrative tasks via **PowerShell**.

**Key Skills Demonstrated:** Active Directory (AD DS) Implementation, DNS/DHCP Configuration, Group Policy Objects (GPO), PowerShell Scripting, Identity & Access Management (IAM), and Cross-Platform (Linux/Windows) Integration.

***

## **1. Network Architecture and Roles**

The environment is built on an **Isolated Internal Network** to simulate a secure corporate perimeter.

| Device Name | OS / Package | Network Role | IP Configuration | User Role (The Human Element) |
| :--- | :--- | :--- | :--- | :--- |
| **DC-01** | Windows Server 2022 | **Primary Domain Controller** | Static: `10.0.0.1` | **The Brain:** Handles all Identity, DNS, and DHCP handshakes. |
| **ITAdmin** | Windows 11 | **Domain Admin (Workstation)** | DHCP Assigned | **IT User:** Domain Administrator. |
| **PC-01** | Windows 11 | **Domain Member (Workstation)** | DHCP Assigned | **Management User:** Standard employee requiring domain-authenticated login. |
| **PC-02** | Windows 11 | **Domain Member (Workstation)** | DHCP Assigned | **Marketing User:** Standard employee requiring domain-authenticated login. |
| **PC-03** | Windows 11 | **Domain Member (Workstation)** | DHCP Assigned | **Sales User:** High-privilege workstation with restricted GPO settings. |
| **PC-04** | Windows 11 | **Domain Member (Workstation)** | DHCP Assigned | **Customer Support:** High-privilege workstation with restricted GPO settings. |
| **PC-05** | Windows 11 | **Domain Member (Workstation)** | DHCP Assigned | **Finance:** High-privilege workstation with restricted GPO settings. |
| **WEB-01** | Ubuntu Server | **Linux Integration** | Static: `x.x.x.x` | **Web Server:** Simulates an internal company portal or dev environment. |

***

## **2. Deployment Phases and Execution Plan**

### Phase 1: Core Infrastructure and Identity Services

**Objective:** Promote the server to a Domain Controller and establish the `leapcorp.local` forest.

| Task | Action / Tool | Security Principle |
| :--- | :--- | :--- |
| **OS Hardening** | Install Win Server 2022 with minimal necessary features. | **Attack Surface Reduction** |
| **AD DS Promotion** | Use Server Manager to promote to Domain Controller. | **Centralized Identity** |
| **IP Schema Setup** | Configure Static IP and DNS Loopback (`127.0.0.1`). | **Network Stability** |
| **DHCP Scope** | Create a scope for `10.0.0.100 - 10.0.0.200`. | **Automated Resource Management** |

### Phase 2: User Governance and Policy Implementation

**Objective:** Organize the directory structure and enforce security via Group Policy Objects (GPO).

| Task | Action / Tool | Administrative Goal |
| :--- | :--- | :--- |
| **OU Structure** | Create Organizational Units (OUs) for Sales, Finance, and IT. | **Logical Administration** |
| **Bulk User Creation** | Import users into AD using **PowerShell** scripts. | **Efficiency & Automation** |
| **GPO Deployment** | Create "Screen Lock" and "Disable USB" policies for Finance OU. | **Security Hardening** |
| **IAM Setup** | Assign users to Security Groups (e.g., `G_Finance_Full`). | **Least Privilege Access** |

### Phase 3: Domain Integration and Validation

**Objective:** Join client workstations to the domain and verify policy enforcement.

| Test Case | Verification Action | Expected Result |
| :--- | :--- | :--- |
| **Domain Join** | Join PC-01 to `leapcorp.local`. | Success message: computer object appears in AD. |
| **DHCP Handshake** | Run `ipconfig /renew` on PC-01. | Client receives IP from the DC-01 DHCP scope. |
| **GPO Enforcement** | Attempt to use a USB drive on PC-02 (Finance). | Access is denied per Group Policy. |
| **DNS Resolution** | `ping web-01.leapcorp.local` from any PC. | Successful name resolution to `172.16.0.10`. |

***

## **3. Project Journal**

A space to document the journey - thought process, challenges, and debugging.

### **[30th December 2025] - Server installation**

* **Process:** 	- Installed Windows Server 2022 in a virtual environment (VirtualBox,) assigning it 1 Intel i5 10300H CPU 2.5GHz and 4GB RAM.
	
	- ISSUE - Display settings max 1600x1200, sound not working.
	- SOLUTION - Installed VM GuestAdditions, the issue persisted. Updates available - Tried installing updates, the server crashed/froze multiple times, after about an hour of trying, I decided to delete everything and perform a clean install of the Server.

SECOND ATTEMPT

- Installed Windows Server 2022 - i5 2.5GHz and 4GB RAM, allocated 50GB of SSD storage.
- This time I decided to perform the Update first then install the Guest Additions, and the system is stable, working well, but the resolution issue persisted.
- SOLUTION - I tried restarting the server multiple times (so the guest addition changes take place), examined the display drivers (they were up to date). Ultimately, the solution was just swapping between host screen 1 and back to host screen 2 (I have 2 monitors on the host machine) and the changes took place immediately - 1920x1080 resolution is in place, system stable and working well.
- Renamed the machine to DC1.
- Changed the network adapter to Internal Network (creating an isolated enterprise environment for the lab)
- Setting up static IP:
		- `Get-NetIPInterface (finding the IPv4 interface)`
		- `New-NetIPAddress -InterfaceIndex 5 -IPAdress 10.0.0.1 -PrefixLength 24 (assigning static IP)`
    - `Set-DnsClientServerAddress -InterfaceIndex 5 -ServerAddress "127.0.0.1"`
* **Challenge/Lesson Learned:** The solution is sometimes easier than it seems (host monitor swap). Clean install was a good idea, prioritizing system stability.
* **Result:** Server is live and healthy.

### **[2nd January 2026] - Active Directory installation and user configuration**

* **Process:** * Installing Active Directory - Using PowerShell to practice commands and get used to the CLI.
 
	- Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
	- Install-ADDSForest -DomainName "leapcorp.local"
	- server successfully promoted to Domain Controller.

At this moment switching from PowerShell to Active Directory (need to be familiar with the software as well, plenty of room to practice PowerShell later in the project.AD Configuration:
	- added OUs as follows:
	* 1.  Employees:     
    *   1. Management
	*     2. Marketing
	*     3. Sales
	*     4. Customer Support
	*     5. IT
	* 2. Hardware:
	*    1. Desktops
	*    2. Laptops
	*    3. Servers
	* 3. Groups:
	

Joining the domain:
- Windows 11 Pro VM (ITAdmin) installed and configured, setting up network adapter to Internal Network (EnterpriseLab), static IP assigned, Subnet assigned, DNS assigned (server), server successfully pinged.
- Joined leapcorp.local domain through This PC>Domain or workgroup>Change>Member of Domain>leapcorp.com
- IT user added in AD Users and Computers>IT>New>User> assigned name, logon name, password, and added to Domain Admins.

* **Challenge/Lesson Learned:** **Configuration went without major issues.
* **Result:** First user created, promoted to Domain Admin, secure password set up, ready to start batch adding other users.

### **[5th January 2026] - DHCP installation and configuration**

* **Process:** * Installing DHCP

	- Installing DHCP from the IT Admin machine using RDP (for the sake of practice).
	- Remote Desktop Connection enabled on the server.
	- Connection attempt 1: unsuccessful. Firewall likely blocking us since we haven't configured anything yet and it defaults to 'Public Network'.
	- Firewall disabled (for troubleshooting purposes), RDP connection attempt 2: unsuccessful.
	- Now checking the TCP handshake on the IT Admin Windows 11 machine using Test-NetConnection 10.0.0.1 -Port 3389, result: FALSE.
	- Checking if the server is listening to TCP 3389 using netstat -ano | findstr 3389, result: only showing UDP 0.0.0.0:63389, no TCP port open.
	- Ran Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0 to force RDP on, restarted the service using Restart-Service TermService -Force, cechek with netstat -ano again, still only seing UDP 0.0.0.0:63389.
	- Final check Test-NetConnection 127.0.0.1 -Port 3389, result: loopback TCPTestSucceded: false. This indicates a deeper issue with the RDP which I will come back to at a later time.
    - Firewall settings turned back on ensuring security best practices.
	- Now configuring DHCP directly on the server console Install-WindowsFeature DHCP -includeManagementTools.
	- DHCP scope configured 10.0.0.100 - 10.0.0.200, network setting on the Windows machine reverted to automatic, ran ipconfig, new IP - 10.0.0.100.
Wasted about an hour trying to troubleshoot RDP connection error, finally installed and configured the DHCP server directly on the server, working well, will return to troubleshoot RDP connection issue at a later time.

* **Challenge/Lesson Learned:** **Spent an entire hour troubleshooting remote desktop connection issues, decided to install and configure it directly on the server, prioritizing stability and functionality. RDP to be fixed in the near future.
* **Result:** DHCP operational, successfully assigning IPs to devices on the network.

### **[12th January 2026] - First GPO upade - Wallpaper push**

* **Process:** * Creating GPO for the default wallpaper and linking it to the right groups.

	- Task completed successfully by creating a shared folder on the server and adding a wallpaper image, creating a GPO and linking it to the right user group.
	- Created a folder  C:\Data and shared it
	- added an image wallpaper.jpg
	- Created a GPO for it - Opened Group Policy Management > right-click Group Policy Objects > New > Name: Wallpaper 
	- Right-click the new GPO > Edit > User Configuration > Policies > Administrative Templates > Desktop > Dekstop > Double-click Desktop Wallpaper > wallpaper name \\DC01\Data\wallpaper.jpg > wallpaper style – fill
	- Linking – Group Policy Management Domain Tree > Employees > IT > right-click IT > Link and existing GPO > select Wallpaper 
	- Verification - logged into the virtual machine, forced updates (gpoupdate /force), rebooted, wallpaper in place.
	

* **Challenge/Lesson Learned:** At this moment, I realize the project reached a level of complexity where running critical infrastructure (Active Directory/DNS) on a transient host (my personal laptop inside VirtualBox) became a bottleneck. To simulate a true Always-On enterprise environment, I am migrating the stack to a dedicated Type-1 Hypervisor on bare metal (Proxmox). 
* **Result:** Realized the impact resource constraints have on the functionality of a system. Investing in new equipment and upgrading. PROJECT ARCHIVED

