# 11/03/2026 Helpdesk Simulation: Legacy Hardware Optimization & Resource Reclamation

**Ticket ID:** LH-4 (LeapCorp Helpdesk)
**Request Type:** Service Request / Performance Degradation
**Support Method:** Remote (AnyDesk)

## 📌 Scenario Overview
A residential user reported severe system latency and freezing, significantly impacting their ability to work. The objective was to remotely triage the system, identify bottlenecks, reclaim system resources, and provide actionable hardware lifecycle advice.

## 💻 System Environment
* **OS:** Windows 10/11
* **CPU:** Intel Celeron 1.9GHz (Legacy/Underpowered)
* **RAM:** 4GB (80% utilization at idle)
* **Storage:** 64GB SSD (Critically low: 17GB free)

## 🛠️ Resolution & Internal Notes
*The following notes were logged in the Jira ITSM system upon ticket resolution:*

**Software Cleanup:**
* Identified and uninstalled multiple redundant antivirus software, PUPs, and OEM bloatware.
* Ran Malwarebytes -> 1 threat found and quarantined.

**OS Optimization:**
* Disabled transparency in colors to reduce GPU/CPU load.
* Optimized for performance under Advanced System Settings.
* Changed Virtual RAM page file to limit CPU usage -> Custom size -> 6144MB (1.5x available RAM).

**Storage Reclamation:**
* Identified drive bottleneck - SSD only had 17GB available.
* Ran disk cleanup, cleared system files, disabled hibernation (`powercfg -h off`), and cleared temporary files.
* **Result:** Reclaimed 12GB of space (29GB available).

**Final Result:** System responsiveness improved. Advised user that modern applications will still cause bottlenecks on this specific hardware.

* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week-6/internal-notes.png)

---

## ✉️ Customer Communication
*Applied the "Sandwich Method" to deliver system updates, manage user expectations regarding hardware limitations, and recommend future upgrade specs.*

**Transcript:**
> Dear Jack,
> 
> I’m writing to inform you that the issue with your computer has been resolved. I’ve identified the cause and implemented the solutions, optimized the system, and you should notice it working much smoother now.
> 
> I’d like to point out an important observation, however. Modern software that we use in our everyday work is demanding more and more resources, and there are limitations to how much we can actually expect from your machine. While it should work noticeably faster, I’d like to advise that you consider upgrading to a workstation with a newer CPU (at least i3 or i5) with a minimum of 8GB RAM (16GB would be even better).
> 
> Thank you for contacting the Service Desk. Wishing you a productive week.
> 
> Kind regards,
> Aleksandar Popov
> IT Support Specialist

* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week-6/communication.png)

---

## 🏢 Enterprise Context & Limitations
**Disclaimer on Tooling:** Because this was a residential/BYOD support scenario, third-party freeware (Revo Uninstaller / Malwarebytes Free) was utilized to aggressively hunt down registry keys and orphaned bloatware files. 

**Corporate Application:** In a strict Enterprise/Domain environment, unapproved third-party software would not be utilized. Instead, I would execute removals via **PowerShell**, **Registry Editor**, or centralized endpoint management tools (like **Microsoft Intune** or **SCCM**). Furthermore, rather than spending extensive billable hours optimizing a Celeron/4GB machine, an enterprise asset with these specifications would be immediately flagged for **Hardware Lifecycle Replacement**.
