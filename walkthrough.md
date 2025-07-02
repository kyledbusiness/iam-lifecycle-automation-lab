# Step-by-Step Walkthrough

This page walks through my experience in the IAM Lifecycle Automation Lab. The lab was conducted in Log(N) Pacific's Cyber Range, and was designed to teach how identity works in the cloud.

My goal was to understand how users are created, how access is assigned using groups and roles, and how identity activity is monitored. The lab used Microsoft Entra ID, Azure, Tenable.io, and Google APIs to build a basic identity lifecycle.

I had Reader-level access at the tenant and subscription scope. Everything shared here is based on what I was able to view from that perspective.

---

## Day 1 – Getting familiar with the environment

**What I wanted to do:**  
Get a sense of the overall setup and how identity and access were being handled.

**What I looked at:**
- Microsoft Entra ID tenant
- User and group structure
- Role assignments at the resource group level
- Microsoft Defender for Cloud and Microsoft Sentinel

**What I noticed:**
- Users were not assigned roles directly. Instead, they were added to groups, and those groups were granted access to specific resource groups in Azure.
- Group names reflected both access level and resource group (for example: `RG1-Contributor-Group`).
- Microsoft Sentinel was already connected to several data sources like AzureActivity and Defender, but I didn’t immediately see logs related to sign-ins or role changes.

**Why this matters:**  
This setup follows common IAM patterns, like group-based access and least privilege. Understanding this structure helped me follow how users were onboarded and what access they received.

---

## Day 2 – Following the provisioning flow

**What I wanted to do:**  
Figure out how users were created and how access was automatically assigned.

**Where the data came from:**  
The process started with a Google Sheet. Each row represented a user, with columns like:
- Name
- Username
- Department
- Role or access level

**What the automation did (step by step):**

1. **Create user in Entra ID**
    ```python
    user_payload = {
        "displayName": row["Name"],
        "userPrincipalName": row["Username"],
        "mailNickname": row["Username"].split("@")[0],
        "accountEnabled": True,
        "usageLocation": "US"
    }

    response = graph_client.post("/users", data=user_payload)
    ```

2. **Add user to an access group**
    ```python
    graph_client.post(f"/groups/{group_id}/members/$ref", data=student_id_payload)
    ```

3. **Assign RBAC role to the group**
    - Each group was already mapped to a built-in role (like Reader or Contributor) at the resource group level
    - Role assignments looked like this:
        ```python
        role_assignment = {
            "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/<GUID>",
            "principalId": student_object_id,
            "scope": f"/subscriptions/<sub-id>/resourceGroups/<rg-name>"
        }
        ```

4. **Create external accounts**
    - The automation also created a Tenable.io user account using the API
    - The Google Sheet was updated to confirm completion
    - A confirmation email was sent using the Gmail API

**What I learned:**  
This flow was a good example of a real-world onboarding process. Instead of assigning access manually, everything was driven by the user's info in the spreadsheet. The use of group-based RBAC helped separate identity from access, which makes the system easier to manage.

### IAM lifecycle visual summary

To help visualize how the whole system worked, here's a high-level diagram of the identity flow:

![IAM Lifecycle Architecture](https://github.com/user-attachments/assets/056c015b-36b8-4b06-b351-978455a755a1)

---

## Day 3 – Using Sentinel to monitor activity

**What I wanted to do:**  
See how identity-related events were being tracked in Microsoft Sentinel and practice using KQL to investigate account activity.

**What I found:**
- Sentinel was connected to log sources like AzureActivity, Defender for Endpoint, and SecurityEvent
- However, Entra ID sign-in and audit logs were not available in the workspace I could access

> Tip: Sign-in and audit logs from Entra can be added to Sentinel by configuring Diagnostic Settings in Microsoft Entra ID and forwarding them to Log Analytics.

---

### KQL queries I used

These are some examples of queries I wrote to explore identity and access events.

#### See who assigned roles

    AzureActivity
    | where OperationNameValue == "Microsoft.Authorization/roleAssignments/write"
    | project TimeGenerated, Caller, ActivityStatus, AuthorizationScope, AuthorizationAction
    | sort by TimeGenerated desc

#### Look for new user provisioning activity

    AzureActivity
    | where OperationNameValue has "Microsoft.Resources/deployments/write"
    | where ResourceProvider == "Microsoft.Resources"
    | project TimeGenerated, Caller, Resource, ActivityStatus

#### Check for failed sign-ins

    SigninLogs
    | where ResultType != 0
    | summarize FailCount=count() by UserPrincipalName, Location, AppDisplayName
    | order by FailCount desc

#### Look for sign-ins from new locations

    SigninLogs
    | summarize by Location, UserPrincipalName
    | where Location != "United States"

#### See when accounts were first and last used

    SigninLogs
    | summarize FirstSeen=min(TimeGenerated), LastSeen=max(TimeGenerated) by UserPrincipalName
    | order by FirstSeen desc

**What I learned:**  
Being able to query logs helped me understand how users were interacting with the environment. I could track when users signed in, if their access had changed, or if something looked unusual.

---

## Day 4 – Wrapping up and documenting the lab

**What I did:**
- Wrote a public-facing set of improvement ideas (see `recommendations.md`)
- Created this walkthrough to help other students follow the same steps
- Sketched out a flow of how the system worked from start to finish

---

### IAM lifecycle summary

Here’s a quick overview of how everything fit together:

| Step | What happened |
|------|---------------|
| 1 | User info added to Google Sheet |
| 2 | Python script created Entra ID user |
| 3 | User added to access group |
| 4 | Group mapped to role at resource group level |
| 5 | External accounts created in Tenable and Gmail |
| 6 | Sheet updated and email confirmation sent |
| 7 | Activity logged (where visible) in Sentinel |

---

## Final thoughts

This lab gave me a chance to explore IAM in a way that felt realistic. I got to follow how users move through a cloud environment—from being created to being given access and showing up in security logs.

For other students going through this lab, I recommend focusing on:
- How groups are used to manage access
- How role assignments are scoped to different resources
- How to use KQL in Sentinel to investigate what users are doing

I'd like to give a thank you to Josh Madakor and the Cyber Range team for putting the Cyber Range together and making it a strong learning experience.
