# IAM Lifecycle Automation Lab — Walkthrough

This document walks through the key tasks, logic, and learning outcomes from the Cyber Range IAM lab. It complements the main `README.md` by providing a deeper look at what was implemented and how it all connects across Microsoft Azure, Entra ID, Tenable, and Google APIs.

## Table of Contents

- [Part 1: Initial Azure IAM and Security Setup](#part-1-initial-azure-iam-and-security-setup)
- [Part 2: Entra ID User Management](#part-2-entra-id-user-management)
- [Part 3: Microsoft Sentinel Alert Rules](#part-3-microsoft-sentinel-alert-rules)
- [Part 4: Role-Based Access Control](#part-4-role-based-access-control)
- [Part 5: IAM Automation System (Python)](#part-5-iam-automation-system-python)
- [Part 6: Google Sheets + Gmail API Integration](#part-6-google-sheets--gmail-api-integration)
- [Part 7: Azure Functions + Key Vault Integration](#part-7-azure-functions--key-vault-integration)

---

## Part 1: Initial Azure IAM and Security Setup

The lab began with a foundational configuration of Azure to enforce security boundaries and proper access control for each student.

### Resource Group Isolation
Each student was provisioned with an isolated Azure Resource Group (RG), automatically named using the format `student-rg-<username>`. This structure ensured that students could deploy and manage resources without impacting one another.

- These RGs were created by automation using the ARM REST API and secured via role-based access control.
- Each student only had access to their own RG.
- Access to Azure resources was tightly scoped. Students had Contributor access only to their own resource group. They could not access other students’ RGs or view broader Azure resources outside their sandbox environment.

> _Screenshot suggestion: Azure portal view of multiple `student-rg-*` resource groups._

### IAM Role Assignment
Access to resources was managed through Azure RBAC (Role-Based Access Control), where students were assigned Contributor, Reader, or other custom roles scoped to their RG.

- Role assignments were done via the Microsoft Graph API and ARM API in automation scripts.
- Student accounts were assigned based on hashed usernames derived from their school emails.
- Automation ensured idempotency: roles were not re-applied if already present.

> _Screenshot suggestion: “Access control (IAM)” tab of a student RG showing user role assignments._

### Sentinel Workspace Configuration
A centralized Microsoft Sentinel workspace was configured at the start of the lab to monitor student environments.

- Each student RG was onboarded to the Sentinel workspace via Data Connectors.
- Sentinel received telemetry and security signals from VMs, NSGs, and other resources.
- Alerts triggered based on KQL-based analytic rules, signaling port scans, malware, ransomware, and data exfiltration attempts.

> _Screenshot suggestion: Sentinel workspace with active Data Connectors._

### Summary
This setup phase demonstrated best practices in IAM scoping, access control delegation, and centralized monitoring in a multi-tenant environment. These configurations formed the base upon which all later automation and response scenarios were built.

---

## Identity & Access Preparation

This phase of the lab focused on establishing a foundational identity and access environment using **Microsoft Entra ID (Azure AD)** and Azure-native access control. These configurations ensured users could securely participate in the lab while respecting least privilege principles.

### Entra ID User Accounts

All student and instructor accounts were created and managed in **Microsoft Entra ID**, serving as the central identity provider. Usernames followed a deterministic pattern derived from email addresses using SHA256 hashing.

> _Screenshot suggestion: Entra ID → Users blade showing list of provisioned accounts (blur usernames if needed)._

- Accounts were generated programmatically using hashed usernames to ensure privacy.
- Each user was given access only to their assigned Azure resources via Role-Based Access Control (RBAC).
- No administrative privileges were granted to standard student accounts.

### Azure Role Assignments

Users were scoped to a single **Resource Group (RG)** named after their UPN prefix (e.g., `student-rg-abc123`). Role assignments were handled automatically through Azure’s authorization APIs.

> _Screenshot suggestion: A user’s role assignment in their resource group (Azure Portal → Access control (IAM))._

- Students were typically granted the **Contributor** or **Reader** role to their RG.
- Role assignments were handled via Microsoft Graph and Azure Resource Manager APIs in the Python automation.

### Integration with Other Tools

The user identities in Entra ID were also used across other integrated services in the lab:

- **Microsoft Sentinel** for SIEM-based alerting and investigation.
- **Tenable.io** for vulnerability and asset management (user-specific account lifecycle).
- **Google Sheets** as a source of truth for tracking active and churned members.

## Summary

This preparation phase ensured that every user identity was properly provisioned and scoped, forming the foundation for secure, automated provisioning and deprovisioning workflows implemented later in the lab.

---

## Manual Provisioning Functions

As part of the IAM lifecycle automation, a manual provisioning mechanism was implemented using an **HTTP-triggered Azure Function**. This served as a self-service fallback system in case a student's account was accidentally deleted, broken, or removed due to churn misclassification.

#### How It Worked

- The HTTP Function listened for incoming requests from the **Cyber Range frontend UI**.
- Students could input their school email address into the UI form to request a reprovision.
- The backend validated the request against the "active" worksheet in Google Sheets.
- If the email was found in the active list, the following steps occurred:
  - A new **Azure account** was (re)provisioned.
  - The user was assigned to the appropriate **roles and resource group**.
  - A **Tenable account** was created.
  - A **notification email** with credentials was sent to the student's school address.

> _Note: If the user was listed in the "churned" worksheet, provisioning was blocked to prevent abuse._

#### IAM Relevance

This function highlights **Just-in-Time (JIT) provisioning principles**, basic **identity validation workflows**, and integration between:
- Azure Identity (Entra ID)
- Tenable.io user management
- Google Sheets (as a data source)
- Gmail API (for user notification)

#### Testing the Provisioning Flow

As part of testing the IAM automation:
- My email was added to the **active users sheet**.
- I triggered the function via the Cyber Range web UI form.
- I verified:
  - Azure account creation in Entra ID
  - Resource group assignment
  - Tenable user creation
  - Receipt of a welcome email with credentials

---

## Automated Deprovisioning Logic

A key part of IAM lifecycle hygiene is revoking access when users are no longer active. This lab implemented an **automated deprovisioning workflow** using an Azure Timer Function to periodically identify and remove churned users from the system.

#### Trigger Mechanism

- A **TimerTrigger** Azure Function ran on a schedule.
- It checked a **Google Sheets worksheet** labeled "churned" for a list of users who had left the Cyber Range.
- These accounts were then processed for removal from all critical systems.

#### Deprovisioning Steps

For each churned user, the following occurred:

1. **Azure AD (Entra ID)**  
   - User account was disabled via Microsoft Graph API.
   - If the user had a personal **resource group**, it was deleted using the Azure Resource Manager API.

2. **Tenable.io**  
   - User account was disabled (or deleted, depending on config).
   - API keys and credentials associated with the user were removed.

3. **Google Sheets Cleanup**  
   - The user's email was removed from:
     - `churned`
     - `active`
     - `cyber-range-joined-members`
   - This prevented accidental reprovisioning via the HTTP self-service function.

> _Screenshot suggestion: Azure user account for a churned user showing "Account Disabled = true"_  
> _Alternative: Log output from the Azure Function execution confirming user deprovisioning._

#### IAM Concepts Demonstrated

- **Lifecycle Management**: Clean handling of joiner/mover/leaver events
- **Principle of Least Privilege**: Ensures former users retain no access
- **Automation**: Reduces manual overhead and human error
- **Multi-system Coordination**: Tightly synchronizes Azure AD, Tenable, and Google Sheets

This system ensured the Cyber Range remained secure and scalable by keeping access strictly limited to active participants only.

---

## Role Assignments & RBAC

Azure’s **Role-Based Access Control (RBAC)** model allows precise control over who can access what, and this lab provided hands-on experience assigning and managing roles through both the Azure portal and code.

#### Student-Level Access

Each student in the Cyber Range was granted limited access by:

- Creating a **dedicated Resource Group** using the student’s UPN.
- Assigning specific roles scoped **only to that Resource Group**.
- Preventing lateral movement or access to tenant-wide resources.

This separation ensured that students could work in isolated environments without interfering with each other or impacting shared infrastructure.

#### ⚙Role Assignment Automation

A provisioning function (Python, Azure Function) automatically:

- Queried Azure AD to fetch the object ID of the student account
- Generated a **unique role assignment ID** (UUID)
- Assigned a built-in role (like Contributor or Reader) to the user within their Resource Group
- Used the Azure Resource Manager API for secure, repeatable deployment

> _Screenshot suggestion: Azure Resource Group → Access control (IAM) blade showing the assigned role for a student UPN_

#### IAM Concepts Demonstrated

- **Least Privilege**: Access confined to scope of responsibility
- **Scoped RBAC**: Roles assigned at the Resource Group level, not tenant-wide
- **Automation**: Role assignment done via API, not manual clicks
- **Separation of Duties**: Prevented students from accessing peer environments or admin controls

This structure not only improved security but also helped simulate real-world enterprise IAM best practices in a learning environment.

---

## Sentinel Alerts & Incident Response
### Sentinel Alerts & Incident Response

As part of the lab environment, Microsoft Sentinel was pre-configured to collect security events and surface suspicious behavior across various data sources. This allowed us to view how alerts and incidents are generated, triaged, and responded to within Azure’s SIEM platform.

#### Key Features Observed:
- **Alerts Generated by Defender for Endpoint**  
  Integration with Microsoft Defender XDR enabled real-time alerting for suspicious activities such as port scans, malware detections, and lateral movement attempts.
  
- **Incident Grouping**  
  Sentinel automatically grouped related alerts into single incidents for streamlined investigation and response.

- **Built-in Playbooks (Logic Apps)**  
  While not editable from a student view, Sentinel included playbooks (automated response actions) for common alert types.

- **Investigate Blade**  
  Each alert provided a graph-based view of the attack timeline and impacted resources, helping visualize scope and progression.

- **Detection Rules**  
  Custom scheduled and built-in analytics rules were in place to detect common attack behaviors.

#### Actions Taken:
- Viewed real-time alerts generated during lab scenarios (e.g., port scan, malware deployment).
- Opened incidents and explored alert details, entities involved, and timeline of the attack.
- Observed how alerts were linked to Defender for Endpoint and Entra ID logs.

---

## Supporting APIs & Tools

To support provisioning, deprovisioning, and automation tasks across Azure and Tenable, this lab made extensive use of APIs and backend services. While students interacted primarily with pre-built Azure Functions and blueprints, inspecting the logic offered insight into real-world IAM automation strategies.

#### Microsoft Graph API
- Used to manage Entra ID (Azure AD) accounts programmatically.
- Enabled:
  - Disabling user accounts.
  - Resetting passwords.
  - Checking account status.
  - Assigning RBAC roles to resource groups.

#### Azure Resource Manager (ARM) API
- Used to manage Azure resources like Resource Groups and role assignments.
- Enabled automated deletion of student resource groups during deprovisioning.

#### Tenable Cloud API
- Enabled:
  - Retrieval and deletion of Tenable user accounts.
  - Optional ability to disable Tenable accounts during churn handling.

#### Google Sheets API
- Served as the source of truth for student membership status (Active, Churned, Joined).
- Python automation read and modified Google Sheets to:
  - Mark students as churned.
  - Prevent reprovisioning by removing them from “active” sheets.
  - Clean up “joined” and “churned” entries during offboarding.

#### Azure Functions
- All IAM lifecycle processes were triggered on a schedule using serverless Azure Functions.
- The core function `timer_user_account_management` performed a nightly cleanup:
  - Disabled Azure & Tenable accounts.
  - Deleted Azure resource groups.
  - Updated Google Sheets.

> _Note: The backend code and logic were not editable by students but were viewable for educational purposes._

---

## Lessons Learned

This lab provided hands-on exposure to several critical IAM principles and security automation workflows in a real Azure environment. Some major takeaways included:

- **Lifecycle Automation is Crucial**  
  Manual account cleanup is error-prone and time-consuming. Automated deprovisioning ensures consistency and reduces risk.

- **Data Consistency is Key**  
  Google Sheets served as a data source of truth. Keeping IAM automation aligned with that source ensured smooth operations.

- **RBAC Can Be Enforced Granularly**  
  Assigning least-privilege roles at the resource group level maintained a secure student experience while allowing experimentation.

- **SIEM Integration Adds Context**  
  Microsoft Sentinel alerts gave visibility into malicious behavior and connected IAM events with real-time detections.

- **APIs Power Modern IAM**  
  The Microsoft Graph API and other cloud APIs are essential for scaling IAM programs. Learning their structure and auth flows is a must for IAM engineers.

- **Cross-Platform Identity Control**  
  Managing accounts across Azure, Tenable, and Google Sheets simulated the real-world complexity of hybrid IAM environments.

- **Read-Only Access Still Offers Deep Learning**  
  Even without write permissions, being able to inspect backend scripts, cloud functions, and RBAC policies helped connect theory to practice.

---

## Suggestions for Students

- **Explore where you have access.**  
  Use your Reader role to examine resource groups, RBAC assignments, alert rules, Sentinel incidents, and VM security-related logs.

- **Pay close attention to identity flows.**  
  Trace how identities are created, managed, and disabled. Even if you don’t have access to external integrations like Google Sheets or Tenable, reviewing how Entra ID and Azure roles are used internally is valuable.

- **Take notes on what’s automated and what’s manual.**  
  Understand which actions require HTTP triggers and which are handled by background jobs. Knowing what needs a human touch vs. what’s auto-handled is a core IAM concept.

- **Use Azure Portal filters to stay focused.**  
  Apply filters to view only your assigned resource group, subscriptions, or services. It’s easy to get lost otherwise.

- **Document everything you touch.**  
  Your future self (and hiring managers) will thank you. Write down what resources you used, what the function names imply, and what you observed in the IAM, networking, and security layers.

- **Explore Sentinel even if it’s not your main focus.**  
  Investigate the alert rules and KQL logic where you can. Understanding detection and response is just as valuable as provisioning.

- **Ask questions and suggest improvements.**  
  Even if you’re new, your outside perspective can spot gaps or opportunities others missed. Bring them up in a respectful way to help improve the range for everyone.


