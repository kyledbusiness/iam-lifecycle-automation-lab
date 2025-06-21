# IAM Lifecycle Automation Lab — Walkthrough

This document walks through the key tasks, logic, and learning outcomes from the Cyber Range IAM lab. It complements the main `README.md` by providing a deeper look at what was implemented and how it all connects across Microsoft Azure, Entra ID, Tenable, and Google APIs.

## Table of Contents

- [1. Lab Setup & Environment](#lab-setup--environment)
- [2. Identity & Access Preparation](#identity--access-preparation)
- [3. Manual Provisioning Functions](#manual-provisioning-functions)
- [4. Automated Deprovisioning Logic](#automated-deprovisioning-logic)
- [5. Role Assignments & RBAC](#role-assignments--rbac)
- [6. Sentinel Alerts & Incident Response](#sentinel-alerts--incident-response)
- [7. Supporting APIs & Tools](#supporting-apis--tools)
- [8. Lessons Learned](#lessons-learned)

---

## Lab Setup & Environment

This lab was hosted inside a Cyber Range environment that simulated an enterprise cloud environment with multi-user IAM scenarios.

**Key services used:**

- Microsoft Azure (multiple subscriptions)
- Entra ID (formerly Azure AD)
- Microsoft Sentinel & Log Analytics
- Tenable.io (cloud-based vulnerability management)
- Google Sheets API (account tracking)
- Azure Functions (Python)

🖼️ *Insert screenshot here of the Azure subscription/resource group layout if allowed*

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

