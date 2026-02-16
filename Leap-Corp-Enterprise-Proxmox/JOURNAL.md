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

---

#Week 2: Access Control & File Services

### 📅 05/02/2026 - Bulk Hiring & AD/DNS Troubleshooting
**Objective:** Prepare environment for bulk user creation and resolve AD/DNS race conditions.

* Prepared a CSV file with 50 employees sorted across departments and a bulk hiring script.
* **Issue:** Tried to RDP into DC01 but got a connection error.
* **Troubleshooting:**
  * Verified network configuration via `ipconfig` on both DC01 (Proxmox console) and IT-Admin VM.
  * Verified connectivity via `ping 192.168.50.5` (Success).
  * Checked Server Manager -> Remote Connection was Disabled. Enabled it with Network Level Authentication (NLA).
  * Checked Firewall via `Get-NetConnectionProfile` -> Showed as **Private Network** (This is a problem for pushing GPOs).
* **Fixing the Network Profile:**
  * Attempted restarting NlaSvc (`Restart-Service NlaSvc -Force`) -> Timed out.
  * Changed NLA to Automatic (Delayed Start) and rebooted -> Issue persisted.
  * Cleared specific DNS suffixes and deleted network profile signatures via Registry Editor (`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\Unmanaged`).
  * **Deep Dive:** Checked DNS and local server event logs. Found that the DNS server was waiting for AD DS to signal initial synchronization, and Disk I/O operations were retrying.
* **Root Cause:** Virtual disk was configured incorrectly (Default/no cache instead of Write-back), causing a race condition between NLA and DNS on boot.
* **The Fix:**
  * Shut down server -> Reconfigured virtual hardware -> Booted up (NLA reverted to Active).
  * Enforced Domain Controller awareness via Registry: 
    `HKLM\SYSTEM\CurrentControlSet\Services\NlaSvc\Parameters` -> Added DWORD `AlwaysExpectDomainController` = 1.
* **Result:** Server rebooted -> NetworkCategory: **DomainAuthenticated**. ISSUE RESOLVED.
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%202/solution.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%202/registry.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%202/domain%20authenticated.png)

---

### 📅 09/02/2026 - File Server (FS01) Provisioning
**Objective:** Deploy and configure a dedicated File Server for the domain.

* **Hardware:** 2 Cores, 2GB RAM, 60GB SSD (Write-back and discard enabled, not). Installed Standard CLI environment only.
* **Issue:** No network adapters found due to missing drivers.
* **The Fix:** Installed VirtIO drivers via command line (`\virtio-win-gt-x64.exe`), installed and rebooted.
* **Configuration:**
  * Assigned IP: `192.168.50.6/24`, Gateway: `192.168.50.1`.
  * **Mistake:** Forgot to set DNS. Domain join failed. 
  * Reconfigured DNS -> Preferred: `192.168.50.5` (DC01), Alternate: `8.8.8.8`.
  * Domain Join (`leapcorp.local`) -> **SUCCESS**.
* **Remote Management:**
  * Added FS01 to Server Manager on DC01.
  * **Issue:** Could not access `\\FS01\c$` (Error 0x80070035 - Network path not found). Ping failed.
  * **The Fix:** Configured Firewall rules via PowerShell on FS01:
    ```powershell
    Set-NetFirewallRule -DisplayGroup "File and Printer Sharing" -Enabled True
    Set-NetFirewallRule -DisplayGroup "Remote Desktop" -Enabled True
    Set-NetFirewallRule -DisplayGroup "Remote Management Service" -Enabled True
    ```
* **Result:** Ping successful. Accessed `\\FS01\c$` and created `CorpData` directories (Sales, HR, Marketing, Management, IT, Public).
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%202/FS01.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%202/firewall%20rules.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%202/corp%20data.png)

---

### 📅 10/02/2026 - Storage Services & GPO Drive Mapping
**Objective:** Configure SMB shares, set NTFS permissions, and deploy mapped drives via GPO.

* **Issue:** Proxmox dashboard reported IPs missing for DC01/FS01. Verified via `get-service qemu-ga` that Guest Additions weren't running properly.
* **The Fix:** Ran `msiexec /i D:\guest-agent\qemu-ga-x86_64.msi /qn` on FS01 and DC01. QEMU Guest Agent now running.
* **File Share Setup:**
  * FS01 was missing the File and Storage Services role.
  * Ran `Install-WindowsFeature -Name FS-FileServer -ComputerName FS01` from DC01.
* **Permission Management:**
  * Attempted to map shares to departments, but Security Groups were missing.
  * Ran a custom PowerShell ISE script to dynamically create groups (`SG_GroupName`) based on OUs and populate them.
  * Attempted to create the Marketing share with Modify permissions, but received "ACCESS IS DENIED".
  * **Root Cause:** The Administrator account was locked out of the Marketing folder at the NTFS level.
  * **The Fix:** Ran a script using the `takeown` command to force NTFS permission reset and regain Admin ownership.
* **GPO Drive Mapping:**
  * Created GPO: `Drive Maps` (User Configuration -> Preferences -> Drive Maps).
  * Utilized **Item-Level Targeting** to map Drive `Z:` strictly to `SG_Management`.
  * Repeated process for all departments.
* **Verification:** Logged in as CEO (Thao). Drive `Z:` mapped and operational.
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%202/qemu%20ga%20FS01.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%202/qemu%20ga%20DC01.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%202/qemu%20ga%20FS01%20proxmox.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%202/qemu%20ga%20DC01%20proxmox.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%202/FS01%20Filerserver%20Role.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%202/script%20error.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%202/drive%20map.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%202/marketing%20share%20fix.png)

---
