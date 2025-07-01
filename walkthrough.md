Thanks again for your patience — you're absolutely right. Triple backticks (` ``` `) are causing issues when wrapped in a full Markdown block, especially in this chat interface.

Here’s the **clean, copy-paste-ready version** of `walkthrough.md` with **no wrapping triple backticks**, and all inner code blocks, tables, and headings correctly formatted.

---

### ✅ Final Clean Markdown (Copy into `walkthrough.md`)

# IAM Lifecycle Lab – Detailed Walkthrough

This walkthrough captures my experience and technical findings from the Cyber Range IAM Lifecycle Automation Lab. The lab focused on simulating identity provisioning, access control, third-party integrations, and monitoring — all using Microsoft Entra ID, Azure RBAC, Tenable, Microsoft Sentinel, and Google APIs.

The repo is designed to help students and entry-level professionals understand real-world IAM concepts in the context of Azure cloud environments. While some of the specifics here are more technical than you'd typically publish, this write-up was encouraged by the lab’s instructor for future students.

---

## 🗓️ Day 1 – IAM Recon & Architecture Discovery

![IAM Lifecycle Architecture](https://github.com/user-attachments/assets/056c015b-36b8-4b06-b351-978455a755a1)

**Goals:**

* Map out the cloud identity architecture
* Identify how users, groups, and roles were structured

**What I Explored:**

* Entra ID > Users: Discovered pre-provisioned test identities
* Entra ID > Groups: Groups were logically named (e.g., `RG1-Contributor-Group`)
* Entra ID > Roles and Administrators: Verified that no Global Admin roles were assigned directly to users
* Azure > Subscriptions: Noted Reader-level access assigned to students
* Azure > Resource Groups: Identified that access was controlled through group assignments at the RG level

**Architecture Pattern Identified:**

* Identities are provisioned in Entra ID via automation
* Users are added to groups
* Groups are assigned built-in RBAC roles (e.g., Reader, Contributor) at specific resource group scopes
* Least privilege is enforced through scoping and grouping, not direct role binding

---

## ⚙️ Day 2 – IAM Lifecycle Flow & Automation Analysis

**Goals:**

* Trace the provisioning process from start to finish
* Understand how access was automatically granted

**Provisioning Trigger:**

* Input came from a **Google Sheet** containing columns like:

  * Name
  * Username
  * Department
  * Desired Access Level
  * Notes or Expiry

**What the Automation Did:**

```
# Simplified pseudocode
user_payload = {
    "displayName": row["Name"],
    "userPrincipalName": row["Username"],
    "mailNickname": row["Username"].split("@")[0],
    "accountEnabled": True,
    "usageLocation": "US"
}

response = graph_client.post("/users", data=user_payload)
```

* Created users using Microsoft Graph

* Added users to security groups using:

  graph\_client.post(f"/groups/{group\_id}/members/\$ref", data=student\_id\_payload)

* Assigned RBAC at resource group level using raw roleDefinitionIds like:

  role\_assignment = {
  "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/<GUID>",
  "principalId": student\_object\_id,
  "scope": f"/subscriptions/<sub-id>/resourceGroups/<rg-name>"
  }

* Created Tenable user accounts via Tenable.io API

* Logged provisioning actions back into the Google Sheet

* Sent confirmation emails through the Gmail API

**Reflections:**

* The IAM flow was simple but mirrored real-world onboarding
* Using group-based RBAC reduced the need for direct user-to-role assignments
* Mapping access based on spreadsheet attributes aligned with basic HR-driven provisioning

---

## 📊 Day 3 – Identity Monitoring & Sentinel KQL

**Goals:**

* Explore Sentinel’s ability to track IAM-related activity
* Use KQL to search for sign-ins, privilege changes, and access behavior

### Log Sources Available:

* **AzureActivity** (via default connector)
* **Defender for Cloud**
* **Microsoft Defender for Endpoint**
* **SecurityEvent**
* **(No Entra Sign-in or Audit logs initially)**

---

### 🔎 Sample Queries I Used

#### 🔹 Role Assignment Visibility

```
AzureActivity
| where OperationNameValue == "Microsoft.Authorization/roleAssignments/write"
| project TimeGenerated, Caller, ActivityStatus, AuthorizationScope, AuthorizationAction
| sort by TimeGenerated desc
```

#### 🔹 New User Provisioning Audit (via Graph)

```
AzureActivity
| where OperationNameValue has "Microsoft.Resources/deployments/write"
| where ResourceProvider == "Microsoft.Resources"
| project TimeGenerated, Caller, Resource, ActivityStatus
```

#### 🔹 Failed Sign-Ins or Suspicious Access Attempts

```
SigninLogs
| where ResultType != 0
| summarize FailCount=count() by UserPrincipalName, Location, AppDisplayName
| order by FailCount desc
```

#### 🔹 Unusual Location Logins

```
SigninLogs
| summarize by Location, UserPrincipalName
| where Location != "United States"
```

#### 🔹 Account Churn / New Account Trends

```
SigninLogs
| summarize FirstSeen=min(TimeGenerated), LastSeen=max(TimeGenerated) by UserPrincipalName
| order by FirstSeen desc
```

---

## 🧠 Day 4 – Wrap-Up & Student Reflections

**What I Delivered:**

* A realistic IAM automation map
* KQL queries and usage examples
* Public-safe recommendations to improve identity visibility and automation resilience
* An architecture diagram showing the full IAM flow from Google Sheet to Azure + Tenable

### IAM Lifecycle Architecture (Public-Safe Summary)

| Stage                          | Description                                                 |
| ------------------------------ | ----------------------------------------------------------- |
| **Trigger**                    | Entry in a Google Sheet                                     |
| **Provisioning**               | Microsoft Graph API call to create Entra ID user            |
| **Group Assignment**           | User added to group based on department/access level        |
| **RBAC Access**                | Group assigned scoped role on Azure RG                      |
| **Tenable/Google Integration** | User added to Tenable; Gmail confirmation sent              |
| **Logging**                    | Sheet updated + logs surfaced in Sentinel (where available) |

---

## 🔚 Final Thoughts

This lab gave me a hands-on opportunity to explore IAM architecture, provisioning flows, and access control design in a secure environment.

Through this project I was able to:

* Analyze lifecycle automation without admin access
* Investigate real security monitoring patterns using Sentinel + KQL
* Reflect on how real IAM systems are built, broken, and improved

This repo and write-up are intended to help future students understand IAM not just as a set of tools — but as a system of design decisions.

> Special thanks to the Cyber Range team for making this lab possible!
