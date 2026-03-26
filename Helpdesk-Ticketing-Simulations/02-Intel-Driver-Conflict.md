# Helpdesk Simulation: Application Error Resolution (igfxem.exe Startup Crash)
### 18/03/2026 

**Ticket ID:** LH-5 (LeapCorp Helpdesk)
**Request Type:** Incident / Application Error
**Support Method:** Remote (AnyDesk)

## 📌 Scenario Overview
A user reported receiving an `igfxem.exe` error prompt upon startup. The objective was to remotely triage the system, investigate the origin of the error, rule out OS corruption or malware, and restore system stability.

## 💻 System Environment
* **OS:** Windows 10/11
* **GPU:** Intel UHD Graphics

## 🛠️ Resolution & Internal Notes
*The following notes were logged in the Jira ITSM system upon ticket resolution:*

**System Scan & Integrity Check:**
* Ran `sfc /scannow` via elevated Command Prompt -> Result: No integrity violations found.
* Checked Event Viewer -> Result: No events or critical failures logged specifically for `igfxem`.

**Malware Detection:**
* Executed a Windows Defender Offline Scan to rule out malicious spoofing of the executable.
* Result: No threats found. 

**Driver Remediation:**
* Checked Device Manager and Intel Graphics Command Center. Windows reported the drivers were "up to date."
* Performed clean installation of the Intel UHD Graphics driver

**Final Result:** Recognizing a known issue with corrupted Intel Graphics components, I proceeded to perform a clean installation of the Intel UHD graphics drivers to resolve underlying file corruption. System rebooted successfully without the `igfxem.exe` error prompt. System is stable.

* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Helpdesk-Ticketing-Simulations/images/02/sfc.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Helpdesk-Ticketing-Simulations/images/02/event-viewer.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Helpdesk-Ticketing-Simulations/images/02/drivers-update.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Helpdesk-Ticketing-Simulations/images/02/internal-notes.png)

---

## ✉️ Customer Communication
*Reassured the user about their hardware health, explained the troubleshooting process, and confirmed the resolution.*

**Transcript:**
> Dear John,
> 
> The problem has been resolved. This is a known issue with Intel Graphics cards.
> 
> I have scanned your system thoroughly and there was no system corruption or malware detected. I performed a clean reinstall of the graphics card drivers and the system is stable. This was most probably just a glitch with corrupt drivers, your workstation is stable and safe to use.
> 
> Wishing you a wonderful weekend ahead.
> 
> Kind regards,
> Aleksandar Popov
> IT Support Specialist

* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Helpdesk-Ticketing-Simulations/images/02/email.png)

---

## 🏢 Enterprise Context & Limitations
**Disclaimer on Tooling:** In this BYOD/residential scenario, local troubleshooting tools (`sfc /scannow`, Event Viewer, Windows Defender Offline) were used to manually rule out threats and perform a local driver replacement.

**Corporate Application:** In a managed Enterprise environment, I would leverage centralized tools. Instead of a local Defender scan, the endpoint's health would be verified via an **EDR solution (like CrowdStrike or Microsoft Defender for Endpoint)**. Rather than relying on local Event Viewer, logs would ideally be aggregated in a **SIEM**. Finally, if this driver corruption was identified as a known issue across multiple devices, a tested, stable Intel driver package would be pushed silently to all affected endpoints using **Microsoft Intune** or **SCCM** to prevent future helpdesk tickets.
