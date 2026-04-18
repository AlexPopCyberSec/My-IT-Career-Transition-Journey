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
