# IAM Lifecycle Automation Lab (Cyber Range)

This repository documents a student-led Identity and Access Management (IAM) lab completed in the Cyber Range environment. The goal of the lab was to simulate a lightweight identity lifecycle system using cloud-native tools and custom automation.

Throughout the lab, I explored how to provision users, assign roles using Azure RBAC, and integrate identity workflows with third-party platforms like Tenable.io and Google Workspace — all while maintaining secure, least-privilege access principles.

## 🔍 What This Repo Includes

- `walkthrough.md`: A day-by-day public-facing walkthrough of the lab environment and my investigative process
- `recommendations.md`: Non-sensitive recommendations for improving access visibility, reliability, and IAM hygiene
- *(optional)* `assets/`: Visuals, diagrams, or sanitized screenshots to support documentation

---

## 🔧 Tools & Technologies Used

- **Microsoft Entra ID** (formerly Azure Active Directory)
- **Azure RBAC** (custom and built-in role assignments)
- **Microsoft Sentinel** (log analysis and security monitoring)
- **Microsoft Graph API**
- **Tenable.io** (external vulnerability management)
- **Google Sheets & Gmail API** (as provisioning source and notification channel)
- **Python** (used to automate provisioning logic)

---

## ⚙️ Architecture Overview

![IAM Architecture Diagram](https://github.com/user-attachments/assets/190b5749-807c-49c8-8d9a-23f7d4a87ef6)

This project simulates a streamlined identity lifecycle system across Microsoft Entra ID, Azure RBAC, and third-party services.

### Key Components

- **Provisioning Trigger**: Google Sheet containing target user data
- **Automation Logic**: Python script using Microsoft Graph, Tenable API, and Google APIs
- **Identity Creation**: New users provisioned in Entra ID
- **Access Assignment**:
  - Users added to Azure security groups
  - Groups linked to RBAC roles at the resource group level
- **Third-Party Integration**:
  - Tenable accounts automatically created for security exposure tracking
  - Updates logged in Google Sheets
- **Notifications**: Emails sent to admins/users confirming provisioning actions

> This mirrors a real-world cloud IAM lifecycle — just scaled down for learning.

---

## 🎯 Learning Objectives

- Understand how IAM concepts translate into real cloud environments
- Use Microsoft Graph to automate identity provisioning
- Assign and scope RBAC roles via group membership
- Monitor access behavior and account lifecycle events
- Integrate identity systems with security and productivity tools

---

## 🤝 Acknowledgments

Thanks to the Cyber Range team for designing this lab and providing access. This project was completed using Reader-level access at both the tenant and subscription scopes, with a focus on responsible exploration and documentation.
