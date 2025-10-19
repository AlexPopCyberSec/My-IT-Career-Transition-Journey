# Project: Leap Media Small Business Network Deployment

## **Overview**

This project simulates the deployment and security configuration of a 6-device, multi-operating system (OS) network environment for a fictional digital marketing agency, "Leap Media". The goal is to showcase practical proficiency in network deployment, system administration, and security principles (CompTIA A+/Network+/Security+).

**Key Skills Demonstrated:** Static IP addressing, Bridged Networking, Web Server Deployment (Nginx), File Sharing (Samba), User/Group Management, and File Permissions (`chmod`/`chown`).

***

## **1. Network Architecture and Roles**

The network operates on a single physical network using **Bridged Adapter** mode for all Virtual Machines (VMs).

| Device Name | OS / Package | Host Machine | Network Role | User Role (The Human Element) |
| :--- | :--- | :--- | :--- | :--- |
| **SRV-01 (SERVER)** | Ubuntu Server | Laptop A | **Static IP** (`192.168.100.200` Recommended) | Central File/Web Server |
| **u_director (DIRECTOR)** | Windows Host | Laptop A | **DHCP** | Creative Director (Needs fast, reliable access to large design files). |
| **u_sales (SALES MANAGER)** | Windows Host | Laptop B | **DHCP** | Sales Manager (Requires secure, remote access via SSH for report generation). |
| **u_designer (DESIGNER)** | Windows VM | Laptop A | **DHCP** | Graphic Designer (Simulates a Mac workstation). |
| **u_finance (FINANCE MANAGER)** | Linux VM | Laptop B | **DHCP** | Finance Manager (Needs highly secure, restricted access for sensitive data). |

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
| **u_designer Designer** | **Operational:** Must save a file to the Marketing Share. | Access `\\192.168.100.200\marketing_share` from File Explorer. |
| **u_sales Sales** | **Security:** Must connect remotely. | Run `ssh sales@192.168.100.200` successfully. |
| **u_finance Finance** | **Security:** Must access the secure folder. | `cd /srv/finance_secure` and `ls -l` must work. |
| **u_designer / u_sales Test**| **Security:** Must be denied access to the Finance folder. | Attempting to `cd` into `/srv/finance_secure` must return **"Permission denied"** for non-finance users. |

***

## **3. Project Journal **

A space to document the journey - thought process, challenges, and debugging as I complete each phase.

### **Oct 2025 - Phase 1: Network Stabilization**

* **Goal:** Set up the server's fixed IP and verify client connectivity.
* **Process** 

* 12th Oct 2025 - Ubuntu Server 24.04.3 LTS downloaded and installed using Oracle Virtual Box. The server has 25gb of storage, 2 cores and 4Gb ram. The initial installation went well. I configured the **Bridged Adapter** prior to installing, so we’ve received an IPv4 address successfully. Our server is online and connected to the Internet.

* 12 Oct 2025 - Successfully assigned the static IP to my server. First, I ran `ip a` to verify the address, then `ip route show default` to find my default gateway address. I ran `ls /etc/netplan/` to see the file name, which I then edited, assigning it the $\mathbf{192.168.100.200/24}$ address. **Minor obstacles with YAML text alignment were sorted quickly**, reinforcing the need for attention to detail in configuration files. Our server now has a static IP address, which was verified using the `ip a show enp0s3` command.

* 12 Oct 2025 - Nginx successfully installed. Performed `sudo apt update` to update the list, then `sudo apt install nginx -y`, `sudo systemctl status nginx` verified it was active, received the `Welcome to Nginx` message in the browser.


* **Result:** Server succesfully installed, static IP assigned, nginx service deployed.

### **October 2025 - Phase 2: Service and Samba Deployment**

* **Goal:** Get file sharing and user accounts ready.
* **Process**

* 13.10.2025 – I realized we have insufficient VMs configured so I configured another Linux VM assigning it 25gb of storage, 8gb ram and 2 CPUs. I then proceeded to group and user creation using `sudo group add [name]` and `sudo adduser [name] –ingroup [name]`, setting up separate passwords for each user. Checked using `id [username]` and verified the users are in correct groups.

* The next step was setting up Samba, so first we installed the Samba service using `sudo apt install samba –y`, and verified the service is active - `sudo systemctl status smbd`. I added our 3 users to Samba using `sudo smbpasswd –a [name]` and setting unique passwords. Our Samba service is now installed, active, and has 3 users. We have succesfully completed the first 2 phases of our network deployment and will now proceed to phase 3 – Security and Permissions.

* **Result:** Groups and user accounts added, Samba installed, functionality verified.

### **October 2025 - Phase 3: Permission and Security**

* **Goal:** Implement the security policy and solve any final access issues.
* **Process**

* 14.10.2025 – Time for user permissions and security. First I created 2 folders, marketing_share (which will be a shared file everyone can access, and finance_secure (only available to the finance user). `sudo mkdir /srv/[name]` used to create folders. I then proceeded to changing the group owners using `sudo chown :g_marketing /srv/marketing_share` and `sudo chown u_finance:g_finance /srv/finance_secure`. Permission changed using `sudo chmod 775 /srv/marketing_share` and `sudo chmod 770 /srv/finance_secure` resulting in the marketing folder being available to all in the group, the marketing group being the owner and having the read/write permissions, while the finance folder is only available to the finance user. Principle of Least Privilege has just been succesfully implemented.

* Samba Share Configuration – we now need to configure the Samba service to recognize and enforce the file system permissions across the network. First, I performed a backup of the Samba configuration file - `sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak`. I then added the following commands to the end of the configuration file:

[Marketing_Share] 
    comment = Leap Media General Access Files 
    path = /srv/marketing_share 
    browseable = yes 
    read only = no 
    valid users = @g_marketing 
    create mask = 0775 
    directory mask = 0775 

[Finance_Secure] 
    comment = Leap Media Highly Restricted Finance Data 
    path = /srv/finance_secure 
    browseable = no 
    read only = no 
    valid users = @g_finance, u_finance 
    force group = g_finance 
    create mask = 0770 
    directory mask = 0770 

resulting in file permission configuration being applied across the network.


* **Result:** Principal of Least Privilege implemented, permissions and Samba Service configured successfully. 


### **October 2025 - Phase 4: Security and Final Verification**

* **Goal:** Test and verify full network functionality.
* **Process:**

* 16th October - Setting up local users on my Windows hosts and Linux VMs. I realized I am missing one more Windows machine so tried installing another VM and got stuck in the loop of creating (and proving I'm not a robot) new Microsoft account. Bypassed the issue by disabling the internet connection (VM Settings  ->  Network  ->  Adapter  ->  Not Attached). Windows installed successfully with a local user. This led to further issues such as the drivers not being installed (no sound, no full hd display option and not running smoothly). I installed VirtualBox Guest Additions but it didn't seem to have fixed the issues. I updated the drivers manually, still no change, so I poceeded to check Windows Update and ran into another problem. Windows update could not update my system because I'd allocated too little storage when I installed it (only 20 GB - beginner's mistake). This was fixed by adding more storage using the CLI on my host machine and the `VBoxManage modifymedium command`. I then extended the drive in my VM and ran the update. Issue persisted. I reinstalled Guest Additions, issue still unresolved.

* 18th October - After multiple failed attempts to sort the issues I decided to remove the machine and perform a clean install. Prior to installing, I configured the graphics adapter to VBoxSVGA, allocating 128mb of video memory, and enabling 3D Acceleration. I also made sure the audio controller was set to Inter HD Audio. After the installation, I noticed the sound issue was fixed, but the display issue persisted. I installed the Guest Additions, however,  the display still couldn’t be set to 1920x1080 Full HD.  I then proceeded to forcing the resolution fix outside the VM using the command `VBoxManage controlvm "machinename" setvideomodehint 1920 1080 32` and the issue was finally resolved.

* 19th October – Final user check and verification. All users on all machines have been added, PoLP applied, local security ensured using `net localgroup users [name] /delete` `net localgroup guests [name] /add`. Each user has their own IP address and they all ping the server 192.168.100.200. Along the way I realized I had only added 3 users to the server and actually need 4, so I added the last user and proceeded to full network functionality test.

FINAL STEP – Full network functionality verification:

1. Director can access the shared marketing file, read and write.
2. Sales manager can access the shared marketing file, read and write.
3. Finance manager cannot access the shared marketing file, he has access to the secure finance folder, no other users can access it.
4. Designer had issues accessing the secure folder even though he was already in the g_marketing group. The issue was resolved by running `sudo smbpasswd –e u_designer` and got a message `Failed to find user u_designer in passdb backend`. Somehow, the user was added to the group, but his password was unknown to the Samba server. Issue resolved by running `sudo smbpasswd –a u_designer` and adding the password. Result: Designer can access the shared marketing folder.
5. Remote access verified – OpenSSH installed on the server using `sudo apt update && sudo apt install openssh-server –y`, enabled using `sudo systemctl enable --now ssh`, ran `sudo systemctl status ssh` to verify it’s active. Sales manager remotely accessed the server using `ssh u_sales@192.168.100.200`.


* **Result:** PROJECT COMPLETED SUCESSFULLY

* **Project Success Summary (Leap Marketing Network)**

1. Network Foundation (CompTIA Network+)
    *Static IP Configuration: Set a permanent address (192.168.100.200) for the server (SRV−01), ensuring stability.
    *Bridged Networking: Configured all VMs to operate as independent devices on the same physical network.
    *Connectivity: All four client devices successfully pinged and connected to the server.

2. Service Deployment & Administration (CompTIA A+/Network+)
    *Web Services: Installed and verified Nginx, making the server publicly accessible via HTTP.
    *File Services: Installed, configured, and restarted the Samba service to facilitate cross-OS file sharing.
    *Remote Access: Installed and enabled the OpenSSH Server on SRV−01 and verified secure remote login from a Windows client.

3. Security and Permissions (CompTIA Security+)
    *Principle of Least Privilege (PoLP): Successfully implemented and verified the PoLP.
    *Local Security: Used net localgroup guests to set Windows client users as standard, low-privilege accounts.
