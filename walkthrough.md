# IAM Lifecycle Lab – Walkthrough

This walkthrough captures my hands-on experience in the Cyber Range IAM Lifecycle Automation Lab. The lab focused on cloud-based Identity and Access Management (IAM), including user provisioning, RBAC enforcement, third-party integrations, and monitoring via Microsoft Sentinel.

I was assigned Reader-level access at both the tenant and subscription scopes. Within those permissions, I explored how identities are created, assigned roles, and monitored — while also identifying where visibility, automation, or structure could be enhanced.

---

## Day 1 – IAM Recon & Environment Discovery

**Goals:**
- Understand the lab’s identity architecture
- Identify key services involved in lifecycle automation

**Activities:**
- Navigated the Azure Portal to identify resource groups and connected services
- Reviewed the Microsoft Entra ID tenant: users, groups, and role assignments
- Explored Microsoft Defender for Cloud recommendations related to identity posture
- Opened Microsoft Sentinel to see what data connectors were active

**Key Takeaways:**
- Entra ID users were managed through group-based access — a scalable model
- Sentinel had connectors enabled for Defender for Endpoint, Activity Logs, and others
- A group-to-role mapping approach was in use for RBAC, which supports least privilege
- No sign-in or audit logs were visible in Sentinel at this point

---

## Day 2 – IAM Automation Flow Analysis

**Goals:**
- Trace the identity lifecycle from user input to access delivery
- Understand how the automation logic fit together

**Activities:**
- Identified that user data originated from a structured Google Sheet
- Observed that new users were provisioned in Entra ID via Microsoft Graph
- Tracked the process of users being added to Azure security groups
- Noted how those groups were mapped to RBAC roles (like Reader, Contributor) at the resource group level
- Noticed integration touchpoints with Tenable (external vulnerability platform) and Gmail (for notifications)

**IAM Flow Summary:**
1. User metadata (name, role, email) appears in the input sheet
2. The automation flow creates the user in Entra ID
3. The user is added to a group corresponding to a specific role
4. Access is granted via the group’s RBAC assignment at scope
5. Third-party accounts are created (Tenable, Google)
6. A success message or confirmation is sent via Gmail

**Reflections:**
- The flow modeled a real-world lifecycle: trigger → provision → assign → notify
- Group-based RBAC helped separate identity from permissions — a good best practice

---

## Day 3 – Sentinel & Access Visibility

**Goals:**
- Explore how identity events and access changes were monitored
- Practice writing queries to trace IAM activity

**Activities:**
- Reviewed Sentinel’s data connectors and workbooks
- Checked for sign-in logs and role assignment audit data (none were flowing yet)
- Ran KQL queries against AzureActivity and SecurityEvent logs
- Tested detections for privilege assignment, unusual login behavior, and new user creation

**Example KQL Queries Used:**
```kql
AzureActivity
| where ResourceProvider == "Microsoft.Authorization"
| where ActivityStatus == "Succeeded"
| project Caller, ActivitySubstatusValue, AuthorizationAction, _TimeGenerated

SigninLogs
| summarize count() by AppDisplayName, UserPrincipalName, Location
```

Reflections:

Identity visibility was decent but could be enhanced with Entra diagnostic logs

Sentinel had strong baseline detection coverage (e.g., failed sign-ins, risky behavior)

Identity events are crucial for auditability and detection — not just logging infrastructure activity

---

## Day 4 – Documentation & Recommendations
**Goals:**

- Summarize lessons learned and propose improvements

- Reflect on how the lab’s IAM setup could evolve

**Activities:**

- Documented my understanding of the IAM automation pipeline

- Wrote up non-sensitive recommendations for improving visibility, error handling, and role clarity

- Mapped out an architecture flow to show how systems interacted

**IAM Lifecycle Architecture (Public-Safe Summary):**

- Trigger: Google Sheet entry acts as onboarding signal

- Provisioning: Entra ID user is created using Microsoft Graph

- Group Assignment: User is added to group based on their role/department

- Access Delivery: Group maps to a built-in or custom RBAC role

- Third-Party Extension: Accounts created in Tenable and Gmail

- Notification/Logging: Email sent, Sheet updated, audit event (where supported)

For general improvement suggestions, see recommendations.md.

---

## Final Thoughts

This lab provided a realistic and hands-on view into how cloud IAM actually works — from provisioning to monitoring. I gained experience in:

- Interpreting access architecture and permissions structure

- Understanding how RBAC and group membership scale IAM policy

- Investigating identity signals in Microsoft Sentinel

- Reflecting on how IAM automation can break, succeed, or be made more secure

These skills are directly relevant to working with real-world cloud environments where access governance and lifecycle hygiene are critical.

Thanks to the Cyber Range team for enabling this experience!
