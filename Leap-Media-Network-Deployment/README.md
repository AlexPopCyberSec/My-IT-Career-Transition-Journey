# Project: Leap Media Small Business Network Deployment

## **Overview**

This project simulates the deployment and security configuration of a 6-device, multi-operating system (OS) network environment for a fictional digital marketing agency, "Leap Media". The goal is to showcase practical proficiency in network deployment, system administration, and security principles (CompTIA A+ Core 1 & 2 skills).

**Key Skills Demonstrated:** Static IP addressing, Bridged Networking, Web Server Deployment (Nginx), File Sharing (Samba), User/Group Management, and File Permissions (`chmod`/`chown`).

***

## **1. Network Architecture and Roles**

The network operates on a single physical network using **Bridged Adapter** mode for all Virtual Machines (VMs).

| Device Name | OS / Package | Host Machine | Network Role | User Role (The Human Element) |
| :--- | :--- | :--- | :--- | :--- |
| **SRV-01 (SERVER)** | Ubuntu Server | Main Laptop | **Static IP** (`192.168.100.200` Recommended) | Central File/Web Server |
| **CLI-01 (DESIGNER)** | Windows Host | Main Laptop | **DHCP** | Creative Director (Needs fast, reliable access to large design files). |
| **CLI-02 (SALES)** | Ubuntu Host | Main Laptop | **DHCP** | Sales Manager (Requires secure, remote access via SSH for report generation). |
| **CLI-03 (MAC-SIM)** | Windows VM | Laptop B | **DHCP** | Graphic Designer (Simulates a Mac workstation). |
| **CLI-04 (FINANCE)** | Linux VM | Laptop B | **DHCP** | Finance Manager (Needs highly secure, restricted access for sensitive data). |

***

## **2. Deployment Phases and Execution Plan**

### Phase 1: Network Stabilization and Core Services

**Objective:** Establish fixed addressing for the server and deploy the essential web platform.

| Task | Command / Action | Verification |
| :--- | :--- | :--- |
| **Set Server Static IP** | Edit the Netplan YAML file on SRV-01 (e.g., `sudo nano /etc/netplan/*.yaml`). | Run `ip a` on SRV-01 to confirm the new, static IP is active. |
| **Verify Bridging** | Ensure all client VMs are set to **Bridged Adapter** mode in VirtualBox. | Run `ip a` on clients; verify they received a `192.168.100.x` address from the physical router. |
| **Deploy Web Server** | Install and start Nginx on SRV-01. | `sudo apt install nginx`, then `sudo systemctl start nginx`. Access `http://192.168.100.200` from any client's web browser. |

### Phase 2: File Sharing, Users, and Security

**Objective:** Install the Samba service, create user accounts, and implement granular file access permissions.

| Task | Command / Action | Security Principle |
| :--- | :--- | :--- |
| **Install File Sharing**| Install the core Samba file-sharing service. | `sudo apt install samba` |
| **Create Directories** | Create the two shared folders on SRV-01. | `sudo mkdir /srv/marketing_share`, `sudo mkdir /srv/finance_secure` |
| **Create Users** | Create the user accounts on SRV-01. | `sudo adduser design`, `sudo adduser sales`, `sudo adduser finance` |
| **Implement Security** | Set secure permissions on the Finance folder. | `sudo chown finance:finance /srv/finance_secure` | **Ownership** |
| **Lock Down Finance** | Restrict access to the Finance folder. | `sudo chmod 700 /srv/finance_secure` | **Least Privilege** |

### Phase 3: Final Verification and Security Test

**Objective:** Prove the network is functional and secure from all workstations.

| Client Test | Access Requirement | Command / Action |
| :--- | :--- | :--- |
| **CLI-01 Designer** | **Operational:** Must save a file to the Marketing Share. | Access `\\192.168.100.200\marketing_share` from File Explorer. |
| **CLI-02 Sales** | **Security:** Must connect remotely. | Run `ssh sales@192.168.100.200` successfully. |
| **CLI-04 Finance** | **Security:** Must access the secure folder. | `cd /srv/finance_secure` and `ls -l` must work. |
| **CLI-03 / CLI-02 Test**| **Security:** Must be denied access to the Finance folder. | Attempting to `cd` into `/srv/finance_secure` must return **"Permission denied"** for non-finance users. |

***

## **3. Project Journal **

A space to document the journey - thought process, challenges, and debugging as I complete each phase.

### **Oct 2025 - Phase 1: Network Stabilization**

* **Goal:** Set up the server's fixed IP and verify client connectivity.
* **Process** 

12th Oct 2025 - Ubuntu Server 24.04.3 LTS downloaded and installed using Oracle Virtual Box. The server has 25gb of storage, 2 cores and 4Gb ram. The initial installation went well. I configured the **Bridged Adapter** prior to installing, so we’ve received an IPv4 address successfully. Our server is online and connected to the Internet.

12 Oct 2025 - Successfully assigned the static IP to my server. First, I ran `ip a` to verify the address, then `ip route show default` to find my default gateway address. I ran `ls /etc/netplan/` to see the file name, which I then edited, assigning it the $\mathbf{192.168.100.200/24}$ address. **Minor obstacles with YAML text alignment were sorted quickly**, reinforcing the need for attention to detail in configuration files. Our server now has a static IP address, which was verified using the `ip a show enp0s3` command.

12 Oct 2025 - Nginx successfully installed. Performed `sudo apt update` to update the list, then `sudo apt install nginx -y`, `sudo systemctl status nginx` verified it was active, received the `Welcome to Nginx` message in the browser.


* **Result:** Server succesfully installed, static IP assigned, nginx service deployed.

### **October 2025 - Phase 2: Service and Samba Deployment**

* **Goal:** Get file sharing and user accounts ready.
* **Process**

* 13.10.2025 – I realized we have insufficient VMs configured so I configured another Linux VM assigning it 25gb of storage, 8gb ram and 2 CPUs. I then proceeded to group and user creation using `sudo group add [name]` and `sudo adduser [name] –ingroup [name]`, setting up separate passwords for each user. Checked using `id [username]` and verified the users are in correct groups.

The next step was setting up Samba, so first we installed the Samba service using `sudo apt install samba –y`, and verified the service is active - `sudo systemctl status smbd`. I added our 3 linux users to samba using `sudo smbpasswd –a [name]` and setting unique passwords. Our Samba service is now installed, active, and has 3 users. We have succesfully completed the first 2 phases of our network deployment and will now proceed to phase 3 – Security and Permissions.

* **Result:** Groups and user accounts added, Samba installed, functionality verified.

### **[Date] - Phase 3: Security and Final Verification**

* **Goal:** Implement the security policy and solve any final access issues.
* **Process** 
* **Result:** 
