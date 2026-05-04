# 📔 Engineering Journal
# Week 1: Cloud Identity & SSO Implementation

### 📅 08/04/2026: Entra ID Sync & UPN Troubleshooting

**Initial Setup**
* **Subscription:** Signed up for Azure Pay-As-You-Go due to trial ineligibility.
* **UPN Suffix Configuration:** Added `leapcorp1.onmicrosoft.com` to Local AD via Active Directory Domains and Trusts.
* **Test Identity:** Created `Cloud Test` user in `OU=IT` using the new cloud UPN.
* **Agent Deployment:** Installed Entra Connect Sync on **DC01**.
    * **Configuration:** Enabled Password Hash Synchronization and Seamless Single Sign-On (SSO).
    * **Result:** Synchronization initiated successfully.
    * *Note:* Identified AD Recycle Bin is not enabled; marked for future maintenance.

**Synchronization Issues identified**
* **Mass Sync:** All 50+ local accounts synced unexpectedly. Because existing accounts still used `@leapcorp.local`, they synced as broken/unlinked identities.
* **Duplicate Identity:** Conflict found with a manually created account from a previous M365 signup due to identical naming conventions.

**THE FIX: Bulk UPN Remediation
1.  **Ghost User:** Deleted the manual duplicate account in the Entra portal.
2.  **Manual Verification:** Updated Thao Nguyen’s account UPN via ADUC and forced a delta sync: `Start-ADSyncSyncCycle -PolicyType Delta`.
3.  **Automation:** Ran the following PowerShell script to update all local user UPNs to match the cloud tenant:

$oldSuffix = "@leapcorp.local"
$newSuffix = "@leapcorp1.onmicrosoft.com"

```Get-ADUser -Filter "UserPrincipalName -like '*$oldSuffix'" | ForEach-Object {
   $newUPN = $_.UserPrincipalName -replace $oldSuffix, $newSuffix
   Set-ADUser -Identity $_.SAMAccountName -UserPrincipalName $newUPN
   Write-Host "Successfully updated $($_.SAMAccountName) to $newUPN" -ForegroundColor Green
}
```

4.  **Verification:** Forced sync again. Result: Accounts linked correctly; duplicates resolved.

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-1/entra-connect-sync-install.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-1/Entra-UPN.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-1/entra-installed.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-1/batch-user-fix.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-1/suffix-fix.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-1/user-fix.png)

---

### 📅 11/04/2026: License Provisioning & SSO GPO Deployment

**License Allocation**
* **Constraint:** Limited to Business Basic (1 License).
* **Action:** Revoked admin license and assigned it to the `Thao Nguyen` synced user in the M365 Admin Center.

**Seamless SSO GPO Configuration**
To enable automatic login without password prompts, configured a new GPO:
* **GPO Name:** `Global – Seamless SSO Enabler`
* **Path:** User Configuration > Policies > Administrative Templates > Windows Components > Internet Explorer > Internet Control Panel > Security Page > Site to Zone Assignment List.
* **Settings:**
    * **Value Name:** https://autologon.microsoftazuread-sso.com
    * **Value:** 1 (Trusted Sites/Local Intranet)

**Validation**
1.  **Password Reset:** Reset Thao Nguyen’s local AD password via ADUC.
2.  **Login Test:** Logged into the local workstation.
3.  **Web Access:** Opened Chrome and accessed `portal.office.com`.
4.  **Outcome:** Entered UPN -> Kerberos ticket was successfully handed over to Entra ID -> **SUCCESSFULLY LOGGED IN** without password prompt.

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-1/sso-gpo.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-1/thao-sso.png)

# Week 2: Entra SAML Toolkit Provisioning (External App SSO Integration Practice)

---

### 📅 18/04/2026: Entra SAML Toolkit Provisioning

**Application Setup & User Assignment**
* **Creation:** Opened Entra ID -> Enterprise Apps -> New App -> Microsoft Entra SAML Toolkit -> Create.
* **Assignment:** Users & Groups -> Add User/Group -> Thao Nguyen -> Select -> Assign.

**SSO Deployment & SAML Configuration**
* **Basic Configuration:** Single Sign-on -> SAML -> Basic SAML Configuration -> Edit.
* **URLs Added:**
  * Reply link: https://samltoolkit.azurewebsites.net/SAML/Consume
  * Sign on URL: https://samltoolkit.azurewebsites.net/
* **Third-Party App Setup:** * Registered an account at SAML Toolkit.
  * Created a new SAML configuration.
  * Pasted details from "Box 4: Set up Microsoft Entra SAML Toolkit" in Entra ID.
  * Downloaded the certificate from Entra, uploaded it to the SAML Toolkit app, and saved.

**Testing & Troubleshooting**
* **Test Execution:** Logged into Thao Nguyen’s virtual machine, opened myapps.microsoft.com, and launched SAML Toolkit. 
* **Result:** Nothing happened.
* **Troubleshooting:** Realized that SAML Toolkit generated new, specific links for my login and handshake. Updated the links in Entra ID and tested again.
* **Failure:** Received error - "Object reference not set"

---

### 📅 20/04/2026: Troubleshooting & Resolution

**The "Clean Slate" Approach**
* After about 2 hours of trial and error trying to sort out the handshake issue and token handover, I decided to nuke both configurations and perform a clean slate setup.
* **Teardown:**
  * Deleted Entra SAML Toolkit from Azure.
  * Deleted SAML Toolkit Configuration from the third-party site.
* **Reconfiguration:** Walked through all the steps again according to the official documentation.
* **Result:** Encountered the exact same "Object reference not set" error.

### **The Fix**
* Realized my AI Assistant was spinning me in loops with generic troubleshooting steps. 
* Pivoted to traditional research: Googled the specific error and found the exact solution on the Microsoft Learn community forums.
* Followed the community-provided steps to correct the token handover process.
* **Culprit:** Missed one step at the beginning, overlooked by the AI assistant as well. 
* **Outcome:** Executed the SSO integration flawlessly. **ISSUE RESOLVED.**

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-2/myapps-sso.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-2/solution.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-2/toolkit-sso.png)

# Week 3: Intune MDM & Device Enrollment

### 📅 27/04/2026: License Upgrades & Cloud Native Enrollment

**Subscription Adjustment**
* **Limitation:** Business Basic did not provide the necessary features for endpoint management.
* **Action:** Enrolled in a 1-month free trial for Microsoft 365 Business Premium (auto-renew disabled).
* **Provisioning:** Assigned Business Premium licenses to `Thao Nguyen` and the `Global Admin` account.

**Intune Configuration**
* **Path:** `intune.microsoft.com` -> Devices -> Enrollment -> Windows -> Automatic Enrollment
* **Setting:** MDM User Scope set to **ALL**.

**Enrollment Strategy & Troubleshooting**
* **Goal:** Use direct Cloud Native Entra Join for Thao's machine.
* **Issue:** Thao's machine is heavily integrated with the local `DC01`. To cleanly test Cloud Native join, the local AD connection must be severed first.
* **Pivot:** Decided to perform the Cloud Native Join on the `IT Admin` VM instead. `Thao Nguyen`'s workstation will be used to practice the **Hybrid Join + GPO** path.
* **Roadblock:** Could not sever the `DC01` connection on the `IT Admin` VM due to a lost local password. Attempts to create a new local user via standard settings yielded an 'Access Denied' error.

**Resolution: Breaking the Domain Link**
1. Logged into the `IT Admin` VM using the root domain admin: `LEAPCORP\Administrator`.
2. Created a new local user account: `LocalRescue`, gave it admin rights.
3. Verified the new account by logging in as `.\LocalRescue`.
4. Logged back in as the IT Admin and severed the domain: Search "About PC" -> Domain or Workgroup -> Change -> Workgroup -> Name: `WORKGROUP` -> Restart.
5. Logged back in as `LocalRescue`.
6. **Entra Join:** Settings -> Accounts -> Access work or school -> "Join this device to Microsoft Entra ID".
7. **Result:** Joined successfully. Device registered in Entra ID and Intune.

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-3/localrescue-account.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-3/localrescue-admin-rights.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-3/localrescue-signin.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-3/entra-join.png)

---

### 📅 28/04/2026: Intune Security Policies & Compliance

**The "Default Compliant" Security Risk**
* **Observation:** The newly enrolled device was marked as "Compliant" despite zero policies being configured.
* **Analysis:** Intune’s default setting marks devices with no assigned policies as compliant. If an attacker enrolled an unmanaged, infected device, it would bypass compliance-based Conditional Access.
* **The Fix:** Intune -> Devices -> Compliance -> Compliance Settings -> *Mark devices with no compliance policy assigned as:* **Not Compliant**.
* **Verification:** Forced a manual sync; the device flagged as non-compliant.

**Creating the Global Compliance Policy**
* **Path:** Devices -> Compliance -> Create Policy -> Platform: Windows 10 and later.
* **Name:** `Global – Windows 11 Baseline Compliance`
* **Settings Enforced:**
  * Device Health: Require BitLocker, Secure Boot.
  * System Security: Require Firewall, Antivirus, Defender Antimalware.
* **Assignment:** All Users.

**Creating the Configuration Profile**
* **Path:** Devices -> Windows -> Configuration -> Create -> New Policy -> Platform: Windows 10 and later -> Settings Catalog.
* **Name:** `Global – Windows Security Baseline`
* **Settings Applied:**
  * Device Password Enabled: `Enabled`
  * Max Inactivity Time Device Lock: `15 minutes`
* **Assignment:** All Devices.
* **Result:** Forced sync. Device successfully evaluated and marked compliant.

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-3/compliant-no-policy.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-3/compliance-settings.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-3/noncompliant.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-3/configuration-policy.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-3/compliant-device.png)

---

### 📅 30/04/2026: Hybrid Entra ID Join (Thao Nguyen)

**Azure AD Connect Object Sync**
* **Action:** Azure AD Connect -> Configure -> Customize synchronization options.
* **Issue:** Blocked by Internet Explorer Enhanced Security Configuration (IE ESC).
* **The Fix:** Server Manager -> Local Server -> IE ESC -> toggled to **OFF**.
* **Filtering:** Selected `02_Workstations` -> `Desktops` for sync.

**Configuring the Service Connection Point (SCP)**
* **Path:** Azure AD Connect -> Configure Device options -> Configure Hybrid Microsoft Entra ID Join -> Windows 10 or later domain-joined devices.
* **Settings:** Checked `leapcorp.local`, Auth Service: Microsoft Entra ID, Enterprise Admin credentials provided.

**Hybrid Join Troubleshooting**
* **Check:** Ran `dsregcmd /status` on Thao's machine -> `AzureADJoined: NO`. Device not in Entra.
* **Attempts:** Ran `dsregcmd /join` and triggered Automatic Device Join in Task Scheduler. Issue persisted.
* **The Catch-22 Fix:** Suspected the certificate propagation delay. 
  1. Rebooted the VM.
  2. Forced delta sync on DC01.
  3. Device appeared in Entra.
  4. Reran `dsregcmd /join` on the endpoint.
  5. `dsregcmd /status` now reports -> `AzureADJoined: YES`.

**GPO Push for Auto-Enrollment**
* **Path:** Group Policy Management -> `leapcorp.local` -> `02_Workstations` -> `Desktops` -> Create GPO.
* **Name:** `Global – Intune Auto-Enrollment`
* **Configuration:** Computer Configuration -> Policies -> Administrative Templates -> Windows Components -> MDM -> Enable automatic MDM enrollment -> Credential type: `User Credential`.
* **Execution:** Ran `gpupdate /force` on Thao’s workstation and rebooted.
* **Outcome:** Intune successfully took over management. The endpoint is visible in both Entra ID and Intune.

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-3/IE-ESC.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-3/IE-ESC-disabled.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-3/OU-filtering.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-3/azurejoined-no.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-3/entra%20ID.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-3/azurejoined-yes.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-3/auto-enrollment-GPO.png)

![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Cloud-Integration/images/week-3/intune-joined.png)
