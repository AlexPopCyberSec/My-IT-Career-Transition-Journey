# Leap Corp: Azure Hybrid & M365 Integration

## 1. Executive Summary
The objective is to extend our existing on-premises Proxmox infrastructure (Leap Industries) into Microsoft Azure and Microsoft 365, creating a secure, unified hybrid cloud environment. Given strict FinOps constraints (out-of-pocket, Pay-As-You-Go), the Azure architecture will be highly ephemeral. We will manually build the environment to develop architectural muscle memory, then automate the lifecycle using Infrastructure as Code (Terraform) to strictly limit hourly billing.

### Success Criteria:
* **On-premises Active Directory (DC01):** Successfully synchronizes users to Microsoft Entra ID.
* **Secure Site-to-Site (S2S) VPN:** Established between the local pfSense firewall and an Azure Virtual Network.
* **Single Sign-On (SSO) and Conditional Access:** Policies deployed to secure cloud resources.
* **Endpoints:** Managed and secured via Microsoft Intune.
* **Azure Security Logs:** Routing successfully through the S2S VPN into the local Splunk SIEM.
* **Automation:** Ability to fully deploy and destroy the Azure infrastructure via Terraform in under 5 minutes.

---

## 2. Resource Inventory & Roles

| Resource Type | Name/Hostname | Role | Billing Profile |
| :--- | :--- | :--- | :--- |
| Azure VNet | VNET-HUB-01 | Primary Cloud Network | Free |
| VPN Gateway | VGW-S2S-01 | Hybrid Connectivity (Basic SKU) | ~$0.036/hr (Destroy after use) |
| Virtual Machine | AZ-MICRO-01 | B-Series Test Workload | ~$0.0104/hr (Destroy after use) |
| M365 Tenant | leapcorp1.onmicrosoft.com | Entra ID, Intune, Exchange | Business Premium 30-day Free Trial |
| Azure Storage | stdiaglogs01 | Diagnostics & Log Staging | Consumption (Micro-cents) |

---

## 3. Logical Architecture (Hybrid Strategy)
We will strictly enforce traffic separation and controlled routing across the hybrid boundary.

* **Local On-Premises:** 192.168.0.0/16 (Encompassing all existing pfSense VLANs).
* **Azure VNet (VNET-HUB-01):** 10.10.0.0/16 (The overarching cloud address space).
* **Azure Subnet (GatewaySubnet):** 10.10.1.0/24 (Dedicated strictly to the VPN Gateway).
* **Azure Subnet (Workloads):** 10.10.2.0/24 (Dedicated to AZ-MICRO-01 and future cloud servers).

---

## 4. Implementation Phases (Estimated 8-10 Weeks)

### Phase 1: Cloud Identity & M365 Foundation
**Objective:** Establish the cloud tenant and sync local identities.
* **Task:** Provision Microsoft 365 Business Premium tenant.
* **Task:** Install and configure Azure AD Connect on the local DC01 to sync domain users.
* **Task:** Configure Enterprise SSO for a third-party application (e.g., Jira).

### Phase 2: Endpoint Management & Security
**Objective:** Secure devices and enforce identity rules.
* **Task:** Enroll a test Windows 11 VM into Microsoft Intune.
* **Task:** Create Intune Configuration Profiles (e.g., BitLocker enforcement, firewall rules).
* **Task:** Build Conditional Access Policies (e.g., Block logins outside of Vietnam, require MFA for Admin roles).

### Phase 3: Hybrid Networking (ClickOps)
**Objective:** Establish the secure tunnel manually to build portal muscle memory.
* **Task:** Provision VNET-HUB-01 and GatewaySubnet in the Azure Portal.
* **Task:** Deploy the Azure VPN Gateway (Basic SKU).
* **Task:** Configure the Local Network Gateway and establish the IPsec S2S connection with the on-premises pfSense.

### Phase 4: Cloud Workloads & SIEM Integration
**Objective:** Deploy a cloud asset and centralize visibility.
* **Task:** Deploy AZ-MICRO-01 (B-Series VM) into the Workloads subnet.
* **Task:** Verify ICMP and SMB/SSH connectivity across the VPN tunnel to local Proxmox servers.
* **Task:** Configure Azure Monitor to route cloud security telemetry down the tunnel to the local Splunk instance.

### Phase 5: Automation & FinOps
**Objective:** Stop clicking and start coding to eliminate "sleep billing."
* **Task:** Translate the networking and compute infrastructure into Terraform (.tf) files.
* **Task:** Practice the `terraform apply` and `terraform destroy` lifecycle to ensure rapid daily teardowns.

## 4. Project Journal (The Engineering Log)
A raw log of the process. Real errors, real fixes.
* [View Log](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/JOURNAL.md)
