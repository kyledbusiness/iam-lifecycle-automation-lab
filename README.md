# 🔐 Azure IAM + Security Automation Lab (Cyber Range)

This project documents a hands-on lab conducted in a Microsoft Azure Cyber Range focused on Identity and Access Management (IAM), security monitoring with Microsoft Sentinel, and lifecycle automation through Python and Azure Functions.

The lab simulates a real-world environment where users (students) are onboarded into Azure, assigned access via roles and resource groups, monitored for security threats, and offboarded using automated workflows.

---

## 🧩 Key Highlights

This lab was divided into three core phases:

### 🔐 1. Identity & Access Management in Azure
- Created and managed users in Entra ID (formerly Azure AD)
- Assigned users to groups and applied Azure RBAC for resource scoping
- Deployed resource groups and provisioned role assignments dynamically
- Explored principal types, directory objects, and administrative units

### 🔔 2. Security Monitoring & Alerting
- Created and tuned log-based alerts with Azure Monitor
- Integrated Microsoft Sentinel as a SIEM
- Detected unauthorized role escalations and suspicious activities
- Explored automated incident generation and visibility

### ⚙️ 3. IAM Lifecycle Automation
- Reviewed and dissected a production-grade Python-based Azure Function used to automate account deprovisioning
- Gained understanding of Microsoft Graph API, Tenable API, and Google Sheets API integrations
- Walked through the offboarding flow including Azure account disablement, Tenable user deletion, and resource group cleanup
- Analyzed logic to prevent reprovisioning of churned users by modifying spreadsheet-based identity sources

---

## 🛠️ Tools & Technologies Used

| Category                    | Tool/Service                     | Purpose |
|----------------------------|----------------------------------|---------|
| **Cloud Platform**         | Microsoft Azure                  | Primary cloud environment |
| **Identity Provider**      | Entra ID (Azure AD)              | Central IAM directory |
| **Access Management**      | Azure RBAC                       | Role assignment and access scoping |
| **Security Monitoring**    | Microsoft Sentinel + Log Alerts  | Alerts, log-based detection |
| **Automation**             | Azure Function App (Python)      | Lifecycle automation via TimerTrigger |
| **IAM Scripting**          | Python + Microsoft Graph API     | User/account management logic |
| **Vuln Management**        | Tenable.io API                   | Churned user cleanup |
| **Data Source**            | Google Sheets API                | Used as external identity source |
| **Notification System**    | Gmail API (optional scope)       | For provisioning notifications (future scope) |

---

## 📂 Repository Contents

| File/Folder              | Description |
|--------------------------|-------------|
| `README.md`              | This file, high-level summary of the lab |
| `walkthrough.md`         | Step-by-step breakdown of all hands-on tasks |
| `recommendations.md`     | Suggestions for improving IAM, automation, and policy |

> _Note: The actual Python source code used in the Cyber Range is proprietary and not included in this repository. Instead, pseudocode, logic explanations, and process flow are documented in `walkthrough.md`._

---

## 🚀 Next Steps

- Jump into [`walkthrough.md`](./walkthrough.md) to see how the lab unfolded
- Review [`recommendations.md`](./recommendations.md) for strategic IAM improvements
- Clone or fork this repo if you're building your own version of this lab

---
## 📧 Contact

For any questions, feedback, or collaboration — feel free to reach out via [GitHub](https://github.com/kyledbusiness/) or [LinkedIn](https://www.linkedin.com/in/kyledbusiness/).
