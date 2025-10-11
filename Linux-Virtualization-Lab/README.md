# Linux and Virtualization Home Lab Practice

## Overview
This is my first home lab project designed to apply and reinforce core concepts from the CompTIA A+ certification, focusing on Operating Systems, Virtualization, Networking, and Security fundamentals.

---

## Phase 1: Hardware Diagnostics and Lab Setup (Core 1 Focus)

This initial phase focused on building a stable, dedicated host machine and setting up a professional virtualization environment. I decided to use an old laptop that wasn't working properly, so hardware replacement was needed.

| Project Objective | Key Actions Performed | A+ Core 1 Skills Demonstrated |
| :--- | :--- | :--- |
| **Physical Host Machine Prep** | Diagnosed and replaced a faulty, riveted laptop keyboard. Replaced thermal paste on CPU, successfully resolving a "stuck heatsink". | Hardware identification, Safety/ESD protocols, Component replacement, Non-standard/field repairs. |
| **Virtual Environment Build** | Bypassed Windows 11 TPM/Secure Boot using Rufus for clean installation media. Installed VirtualBox and allocated 2 CPU Cores and 4 RAM. Troubleshooted graphics conflicts by disabling 3D Acceleration. | Operating System Installation, Virtualization (Hypervisor), Resource Allocation, Troubleshooting graphics drivers. |

---

## Phase 2: System Fundamentals and Connectivity (Core 1 & 2 Focus)

Practice essential command-line tools for diagnostics, networking, and controlling system services on both Windows and the Ubuntu VM.

| Project Objective | Key Commands Practiced | A+ Core 1/2 Skills Demonstrated |
| :--- | :--- | :--- |
| **Initial Diagnostics** | Linux: `lscpu`, `free -h` (System specifications and RAM utilization) <br> Windows: `ipconfig /all`, `tracert` (Network details and path verification) | Performance monitoring, Resource utilization analysis, Command-line networking. |
| **Network Configuration** | Linux: `ip a`, `sudo timedatectl set-ntp true` (Verify IP, set time synchronization) <br> VirtualBox: Switched from **NAT** to **Bridged Adapter** mode for direct network access. | DHCP/Static IP concepts, Network troubleshooting methodology, SSH setup and remote access (using PuTTY). |
| **Service and Process Management** | Linux: `htop`, `ps aux`, `kill [PID]`, `pkill -f [service]`, `sudo systemctl [start/stop/status] ssh` | Process monitoring, Service management, Terminating rogue processes and understanding service dependencies. |

---

## Phase 3: Security and Maintenance (Core 2 Focus)

Enforced security best practices, focusing on user access control, system patching, and implementing backup procedures.

| Project Objective | Key Commands Practiced (Linux) | A+ Core 2 Skills Demonstrated |
| :--- | :--- | :--- |
| **User & File Security** | `sudo adduser`, `sudo userdel -r`, `sudo gpasswd -d [user]` (Remove admin rights) <br> `chmod 700`, `chown` (File permissions and ownership) | User management, Group rights, File permissions (understanding Octal codes), Principle of Least Privilege. |
| **System Maintenance** | `tar -czvf` (Data backup) and `tar -xzvf` (Data restore) <br> `sudo apt update/upgrade` (Patching and package management) | Backup/Restore procedures, Understanding Differential vs. Full backup concepts, System patching. |
| **Security Audit** | `sudo apt install clamav`, `sudo freshclam`, `clamscan [file]` | Antivirus installation, Definition updates, Verification of security tool functionality, Endpoint security. |

---

## Phase 4: Final Project Simulation: Unresponsive File Server

This simulation synthesized knowledge from all phases into a practical troubleshooting scenario.

| Project Objective | Resolution Path | A+ Synthesis Skills |
| :--- | :--- | :--- |
| **Unresponsive File Server** (Help Desk Ticket) | 1. **Performance Check:** Used `htop` to identify RAM overload caused by a background process (Chromium) $\to$ **Fix:** `pkill -f chromium`. <br> 2. **Service Check:** Used `systemctl status samba` to find the service was **inactive (dead)** due to an incorrect package. <br> 3. **Final Fix:** Installed correct package $\to$ `sudo apt install samba`. | Full troubleshooting loop, Identifying incorrect package dependencies, Prioritizing resource management, Service restoration. |
