# 📔 Engineering Journal
#Week 1: Infrastructure & Core Identity

### 📅 23/01/2026 - Network Physical Layer Setup
**Objective:** Establish a physically separate lab network using Asus RT-AX52 daisy-chained to the main ISP router.

* Plugged Asus router into the main ISP modem.
* Accessed web portal, created new SSID/Password, updated firmware.
* **Issue:** Interference detected. Main router speed: 460Mbps, Lab router: 260Mbps (placed too close).
* **Fix:** Relocated Asus router to another room using a 20m CAT6 cable.
* **Result:** Speed stabilized at **514Mbps**.
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/Active%20Directory.png)

---

### 📅 24/01/2026 - Routing & Switching Configuration
**Issue:** Daisy-chaining was temporary. Needed to replace the ISP router entirely but lacked PPPoE credentials.
**Discovery:** Logged into the ISP router interface and extracted the necessary PPPoE credentials.

**Troubleshooting Bridge Mode:**
* **Attempt 1:** Set ISP router to Bridge Mode -> Configured Asus with credentials.
* **Result:** Network Dead. Asus disconnected. Traffic not routing.
* **Root Cause:** ISP bridge mode implementation was flaky/locked. Traffic wasn't passing to the Asus WAN port.
* **The Fix:** Removed ISP router entirely. Connected Asus RT-AX52 directly to the ONT. Configured PPPoE + **MAC Address Cloning** (cloned ISP router's MAC).
* **Result:** Network UP. Speed: **529Mbps**.
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/Asus%20replaced.png)

**Switch Configuration (TP-Link ES208G):**
* Connected Port 1 to Router (VLAN 1). Internet OK.
* Connected Port 2 (Dell Optiplex) and Port 3 (Laptop).
* **Issue:** Dell Port flashing green/orange. Speed capped at **95Mbps** (Fast Ethernet) instead of Gigabit.
* **Troubleshooting:**
    * Swapped cables between Dell and Laptop -> Same issue on Laptop.
    * Reverted to original cable on Laptop -> Issue persisted (even with the cable that worked 5 mins ago).
    * **Diagnosis:** Ghost configuration or negotiation error on the switch.
* **The Fix:** Hard Reset the Switch.
* **Result:** Both devices negotiating at **1Gbps**. Speed test: ~500Mbps. Network is live.
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/TP%20Link.jpg)

---

### 📅 25/01/2026 - Hypervisor Deployment (Proxmox VE)
**Objective:** Turn Dell Optiplex into a Type-1 Hypervisor "Datacenter".

* Installed Proxmox VE successfully.
* **Post-Install Configuration:**
    * Disabled Enterprise Repositories (pve-enterprise, ceph-enterprise).
    * Added **No-Subscription Repository**.
    * Updated system packages.
* **VM Creation:** Uploaded Windows Server 2022 ISO. Created VM (2 Cores, 4GB RAM, 60GB Storage).

---

### 📅 26/01/2026 - Domain Controller (DC01) Setup
**Objective:** Promote Server 2022 to Domain Controller.

**PowerShell Configuration:**
* Renamed Computer: `Rename-Computer –NewName "DC01"`
* Static IP: `New-NetIPAddress –InterfaceIndex 5 –IpAddress 192.168.50.5 –PrefixLength 24`
* Gateway: `New-NetRoute –DestinationPrefix "0.0.0.0/0" –NextHop "192.168.50.1"`
* DNS: `Set-DnsClientServerAddress –ServerAddress "127.0.0.1"` (Loopback for AD)

**Active Directory Deployment:**
* Installed Role: `Install-WindowsFeature –Name AD-Domain-Services –IncludeManagementTools`
* Promoted DC: `Install-ADDSForest –DomainName "leapcorp.local"`

**OU Structure Implemented:**
* `00_Admins`
* `01_Employees` (Management, Marketing, Sales, CS, IT)
* `02_Workstations` (Laptops, Desktops)
* `03_Servers`
* `04_Groups` (Security, Distribution)
* `05_Service_Accounts`

**Users:** Created Admin (Aleksandar) and Director (Thao).
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/Active%20Directory.png)

---

### 📅 27/01/2026 - Driver Optimization (VirtIO & SPICE)
**Objective:** Optimize Windows VMs on Proxmox.

* **Issue:** Poor performance and missing drivers on Windows Server VM.
* **Actions:**
    * Mounted `virtio-win.iso`. Installed Guest Agents.
    * Enabled **QEMU Guest Agent** in Proxmox options.
    * Replaced Network Card: Intel E1000 -> **VirtIO (Paravirtualized)**.
    * **Audio Issue:** No sound. Added Intel HDA.
    * **Display Issue:** Console laggy. Switched Display to **SPICE**.
* **New Issue:** SPICE console in browser rendered mouse unusable.
* **Fix:** Enabled **RDP** on the server.
* **Result:** Logged in via RDP. Full Screen, Sound ON, Performance smooth.

---

### 📅 28/01/2026 - GPO Deployment (Wallpaper Policy)
**Objective:** Verify Domain Join and push visual policies.

* **Domain Join:** Joined 2 Windows 11 VMs to `leapcorp.local`.
* **GPO Implementation:**
    1.  Created Shared Folder: `C:\CorporateData\Wallpapers` (Read-only for Domain Users).
    2.  Created GPO: `Wallpaper - IT`.
    3.  Path: `User Config > Admin Templates > Desktop > Desktop Wallpaper`.
    4.  Target: `\\DC01\CorporateData\Wallpapers\IT.jpg`.
    5.  Linked GPO to `IT` OU.
    6.  Repeated for `Management` OU.
* **Verification:** Ran `gpupdate /force` on client machines. Rebooted. Wallpapers applied successfully.
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/IT%20wall.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/management%20wall.png)
