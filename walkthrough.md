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

### 1. Entra ID User Accounts

All student and instructor accounts were created and managed in **Microsoft Entra ID**, serving as the central identity provider. Usernames followed a deterministic pattern derived from email addresses using SHA256 hashing.

> _Screenshot suggestion: Entra ID → Users blade showing list of provisioned accounts (blur usernames if needed)._

- Accounts were generated programmatically using hashed usernames to ensure privacy.
- Each user was given access only to their assigned Azure resources via Role-Based Access Control (RBAC).
- No administrative privileges were granted to standard student accounts.

### 2. Azure Role Assignments

Users were scoped to a single **Resource Group (RG)** named after their UPN prefix (e.g., `student-rg-abc123`). Role assignments were handled automatically through Azure’s authorization APIs.

> _Screenshot suggestion: A user’s role assignment in their resource group (Azure Portal → Access control (IAM))._

- Students were typically granted the **Contributor** or **Reader** role to their RG.
- Role assignments were handled via Microsoft Graph and Azure Resource Manager APIs in the Python automation.

### 3. Integration with Other Tools

The user identities in Entra ID were also used across other integrated services in the lab:

- **Microsoft Sentinel** for SIEM-based alerting and investigation.
- **Tenable.io** for vulnerability and asset management (user-specific account lifecycle).
- **Google Sheets** as a source of truth for tracking active and churned members.

## Summary

This preparation phase ensured that every user identity was properly provisioned and scoped, forming the foundation for secure, automated provisioning and deprovisioning workflows implemented later in the lab.

---

## Manual Provisioning Functions

As part of the IAM lifecycle automation, a manual provisioning mechanism was implemented using an **HTTP-triggered Azure Function**. This served as a self-service fallback system in case a student's account was accidentally deleted, broken, or removed due to churn misclassification.

#### 🔍 How It Worked

- The HTTP Function listened for incoming requests from the **Cyber Range frontend UI**.
- Students could input their school email address into the UI form to request a reprovision.
- The backend validated the request against the "active" worksheet in Google Sheets.
- If the email was found in the active list, the following steps occurred:
  - A new **Azure account** was (re)provisioned.
  - The user was assigned to the appropriate **roles and resource group**.
  - A **Tenable account** was created.
  - A **notification email** with credentials was sent to the student's school address.

> _Note: If the user was listed in the "churned" worksheet, provisioning was blocked to prevent abuse._

#### 🔐 IAM Relevance

This function highlights **Just-in-Time (JIT) provisioning principles**, basic **identity validation workflows**, and integration between:
- Azure Identity (Entra ID)
- Tenable.io user management
- Google Sheets (as a data source)
- Gmail API (for user notification)

#### 🧪 Testing the Provisioning Flow

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

#### ⏲️ Trigger Mechanism

- A **TimerTrigger** Azure Function ran on a schedule.
- It checked a **Google Sheets worksheet** labeled "churned" for a list of users who had left the Cyber Range.
- These accounts were then processed for removal from all critical systems.

#### 🧹 Deprovisioning Steps

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

#### 🧠 IAM Concepts Demonstrated

- **Lifecycle Management**: Clean handling of joiner/mover/leaver events
- **Principle of Least Privilege**: Ensures former users retain no access
- **Automation**: Reduces manual overhead and human error
- **Multi-system Coordination**: Tightly synchronizes Azure AD, Tenable, and Google Sheets

This system ensured the Cyber Range remained secure and scalable by keeping access strictly limited to active participants only.

---

## Role Assignments & RBAC

Each provisioned user was assigned scoped RBAC permissions to their own Azure resource group.

- `Contributor`, `Reader`, and/or custom roles via Azure Role Assignment API
- Used unique role assignment GUIDs
- Role assignment was part of both provisioning and cleanup logic

🖼️ *Screenshot of role assignments from Azure Portal (redacted)*

---

## Sentinel Alerts & Incident Response

This part of the lab focused on creating detection mechanisms and security automation:

- Microsoft Sentinel setup with custom analytics rules
- Triggered playbooks via Logic Apps
- Created alerts on simulated threats: ransomware, port scans, data exfiltration

🧠 **Scenarios handled:**

- Alerting on suspicious VM behavior
- Auto-remediation via Logic Apps (e.g., shut down VMs)
- Manual hunting using Sentinel Workbooks

🖼️ *Insert image of Sentinel alert or Workbook if allowed*

---

## Supporting APIs & Tools

- **Google Sheets API** — used for tracking active vs. churned users
- **Microsoft Graph API** — core to user creation, disabling, and updates
- **Tenable API** — used for user management and deletion
- **Azure Management API** — for provisioning and deleting resource groups

📁 *Python code is excluded due to confidentiality; pseudocode and logic are described instead*

---

## Lessons Learned

This lab was an excellent opportunity to:

- Build IAM automation across hybrid cloud platforms
- Understand user lifecycle management from a security-first perspective
- Interact with real enterprise-grade tools in a safe sandbox
- Learn practical use of REST APIs, Azure Functions, and Entra ID operations

---

## ✅ Next Steps

To improve this system further:

- Expand coverage with Logic Apps for approval-based provisioning
- Improve error handling and observability (e.g., via Application Insights)
- Draft IAM policies for onboarding/offboarding

