# IAM Lifecycle Automation Lab — Walkthrough

This document walks through the key tasks, logic, and learning outcomes from the Cyber Range IAM lab. It complements the main `README.md` by providing a deeper look at what was implemented and how it all connects across Microsoft Azure, Entra ID, Tenable, and Google APIs.

## Table of Contents

- [Part 1: Lab Setup & Environment](#lab-setup--environment)
- [2. Identity & Access Preparation](#identity--access-preparation)
- [3. Manual Provisioning Functions](#manual-provisioning-functions)
- [4. Automated Deprovisioning Logic](#automated-deprovisioning-logic)
- [5. Role Assignments & RBAC](#role-assignments--rbac)
- [6. Sentinel Alerts & Incident Response](#sentinel-alerts--incident-response)
- [7. Supporting APIs & Tools](#supporting-apis--tools)
- [8. Lessons Learned](#lessons-learned)

---

## Part 1: Initial Azure IAM and Security Setup

The lab began with a foundational configuration of Azure to enforce security boundaries and proper access control for each student.

### Resource Group Isolation
Each student was provisioned with an isolated Azure Resource Group (RG), automatically named using the format `student-rg-<username>`. This structure ensured that students could deploy and manage resources without impacting one another.

- These RGs were created by automation using the ARM REST API and secured via role-based access control.
- Each student only had access to their own RG.
- A reader role was assigned at the subscription level so students could inspect resources, but not alter them outside their RG.

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

Before automation, the lab involved manually managing cloud identities and access:

- Student UPNs were SHA256-hashed and appended with an internal domain
- Azure users were created manually for testing
- Basic Entra ID tasks included:
  - Resetting passwords
  - Enabling/disabling accounts
  - Assigning roles

🖼️ *Insert screenshot of Entra ID users screen (sanitized)*

---

## Manual Provisioning Functions

Provisioning functionality was exposed via HTTP-triggered Azure Functions for internal use:

- Create a user in Entra ID
- Provision a Tenable.io user
- Create a resource group per user
- Assign user-specific IAM roles

The logic used Microsoft Graph API, Tenable’s REST API, and Azure Management APIs.

📌 **Interesting logic choices:**

- Email-based account tracking through Google Sheets
- SHA256 hashing for usernames (to anonymize identities)
- Auto-generated passwords with force-change flag

🖼️ *Screenshot of provisioning HTTP Function GUI in Azure Portal*

---

## Automated Deprovisioning Logic

A scheduled function (`timer_user_account_management`) was deployed to automatically:

- Identify churned users via a Google Sheet
- Disable their Entra ID accounts
- Delete their Tenable.io accounts
- Delete their Azure resource groups
- Remove them from the "active" and "joined" tracking sheets

🧠 **Technologies used:**

- Azure Functions (timer trigger)
- Google Sheets API (Python)
- Microsoft Graph API (user disabling)
- Tenable.io API (account deletion)

🖼️ *Optional: Flow diagram of deprovisioning logic*

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

