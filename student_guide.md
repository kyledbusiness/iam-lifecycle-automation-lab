# Student Guide: IAM Lifecycle Automation in the Cyber Range

This guide is for fellow students participating in the IAM Lifecycle Automation lab in the Cyber Range. It walks through what to expect, how the environment is structured, and how to make the most of the hands-on experience.

---

## Table of Contents
- [1. Overview](#1-overview)
- [2. Lab Objectives](#2-lab-objectives)
- [3. Key Tools and Concepts](#3-key-tools-and-concepts)
- [4. Lab Flow Summary](#4-lab-flow-summary)
- [5. Common Gotchas](#5-common-gotchas)
- [6. Tips for Success](#6-tips-for-success)

---

## 1. Overview

This lab simulates a real-world environment for practicing Identity and Access Management (IAM) tasks, including account provisioning, privilege assignment, security alerts, and deprovisioning.

It’s part of the Cyber Range and is driven by Python-based automation, API integrations (Azure, Tenable, Google), and scheduled Azure Functions. You’ll work with real identity systems in a safe and contained setting.

---

## 2. Lab Objectives

You will:
- Learn how IAM lifecycle automation works from start to finish.
- Interact with Microsoft Entra ID (formerly Azure AD), Tenable, and other tools.
- Monitor security alerts and identity logs using Microsoft Sentinel.
- Review automation scripts that provision, disable, and clean up accounts based on spreadsheet inputs.
- Make minor updates to automation logic (where permitted).
- Understand how IAM fits into broader cloud security practices.

---

## 3. Key Tools and Concepts

| Tool | Purpose |
|------|---------|
| **Microsoft Entra ID** | Cloud-based identity provider used for account management and role assignments. |
| **Azure Resource Groups** | Created and deleted dynamically per user to simulate real cloud access. |
| **Microsoft Sentinel** | SIEM tool used for monitoring identity-based threats and alerts. |
| **Azure Monitor** | Used to track alert rules and automation triggers. |
| **Tenable.io** | External vulnerability management platform with IAM API integration. |
| **Google Sheets (service account)** | Acts as the system of record for active/churned users. |
| **Azure Functions** | Executes scheduled automation tasks like disabling churned users. |
| **Python (requests, logging, json)** | Used to build the automation logic behind provisioning and deprovisioning. |

---

## 4. Lab Flow Summary

Here's a simplified version of what happens behind the scenes:

1. **Provisioning:**
   - A student joins the range and is added to the Active Users sheet.
   - Their Azure account, Tenable user, and resource group are automatically provisioned.
   - They receive a welcome email with login info (Gmail API).

2. **Alerts & Activity:**
   - Resource use is tracked via Azure Monitor and Sentinel.
   - Alerts are triggered for common threat patterns (e.g., port scans, malware deployment).
   - Students respond to incidents and practice remediation.

3. **Deprovisioning:**
   - When a user is marked as "churned," a scheduled Azure Function disables their Azure account, deletes their Tenable account, and removes their resource group.
   - Their email is removed from the sheets to prevent reprovisioning.

*Diagram goes here showing high-level IAM flow*

---

## 5. Common Gotchas

- **Account not provisioned?** Check that your email is in the `Active` tab and not duplicated elsewhere.
- **Tenable access fails?** Your account may not have been created or could already be deleted — report to your instructor.
- **Sentinel alerts not firing?** Verify that the VM is online and performing actions that match the alert rules.
- **Azure functions not behaving?** Look in Azure Monitor logs for recent job activity or errors.

---

## 6. Tips for Success

- Treat this like a real job — document what you’re doing as if you're a security analyst.
- Experiment within your VM safely — trigger alerts and observe how automation responds.
- Explore the automation code (Python scripts) even if you’re not editing them — it’s a great way to understand IAM processes.
- Ask questions — this is a low-stakes environment meant for learning.

---

> 📘 You can also view the [Walkthrough](./walkthrough.md) for a detailed breakdown of each part of the project.

