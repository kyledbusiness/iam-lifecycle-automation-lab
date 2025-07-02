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
- Group names reflected both access level and resource group (for example, names like `RG1-Contributor-Group`).
- Microsoft Sentinel was connected to several sources like AzureActivity and Defender. Based on what I could see, there were no sign-in or audit logs from Entra ID in the connected workspace.


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
   - The group already had a built-in role (like Reader or Contributor) assigned at the resource group level.
   - When a user was added to the group, they automatically received access to that scope.
   - Here is an example of how that role assignment might be structured:
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

---

## Day 3 – Reviewing identity activity in logs

**What I wanted to do:**  
Check how identity activity was being tracked in the environment and practice writing KQL queries to investigate user access and sign-in behavior.

**What I found:**
- Microsoft Sentinel was deployed and connected to log sources like AzureActivity, Defender for Endpoint, and SecurityEvent
- I used the Log Analytics Workspace linked to Sentinel to write and run KQL queries
- Entra ID sign-in and audit logs were not available in the workspace I had access to

> Tip: Sign-in and audit logs from Entra can be added by setting up Diagnostic Settings in Entra ID and forwarding them to the Log Analytics Workspace used by Sentinel.

### KQL queries I used

These are some of the KQL queries I tested while working in the Log Analytics Workspace. Some returned useful results, while others helped me understand what data was (and wasn’t) visible in this lab setup. Each example includes a short explanation for anyone trying to learn from the same lab.

#### Role assignment activity

*Shows who or what assigned a role in Azure. Useful for tracking when RBAC changes happen and who initiated them.*

```kql
AzureActivity
| where OperationNameValue has "roleAssignments"
| project TimeGenerated, Caller, OperationNameValue
| sort by TimeGenerated desc
```

> In this query, `Caller` is often a user, service principal, or script that made the change. If it shows a GUID, it’s likely a service principal used by automation.

#### Failed sign-in attempts

*Lists accounts with failed sign-ins and how often they failed. This can show misconfigured scripts, password spray attempts, or risky behavior.*

```kql
SigninLogs
| where ResultType != 0
| summarize FailCount = count(), LatestAttempt = max(TimeGenerated) by UserPrincipalName, IPAddress, AppDisplayName
| order by FailCount desc
```

> This is useful for identifying accounts that are repeatedly targeted or failing to authenticate. You can also filter by time range or specific IP addresses.

#### Sign-ins from outside the US

*Highlights accounts that signed in from countries other than the US. Helpful for spotting unexpected access.*

```kql
SigninLogs
| extend Location = tostring(LocationDetails["countryOrRegion"])
| where Location != "US"
| summarize SigninCount = count() by UserPrincipalName, Location
| order by SigninCount desc
```

> This relies on IP-based geolocation and may include VPNs or travel. Still useful as a visibility check.

#### Account first seen and last seen

*Shows when each user account was first used and most recently active.*

```kql
SigninLogs
| summarize FirstSeen = min(TimeGenerated), LastSeen = max(TimeGenerated) by UserPrincipalName
| order by FirstSeen desc
```

> Great for finding unused accounts or tracking recent login patterns.

**What I learned:**  
Running these queries helped me understand how user activity shows up in log data. I was able to track failed sign-ins, role assignment events, and login history for different accounts. Even though some data wasn't available (like Entra sign-in logs), the exercise gave me a better idea of what to look for in a real environment.

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
| 2 | Python script created a new user in Entra ID |
| 3 | User was added to an access group |
| 4 | Group was already mapped to a role at the resource group level |
| 5 | External accounts were created in Tenable and Gmail |
| 6 | Google Sheet was updated and a confirmation email was sent |
| 7 | Activity was logged (where available) in the Log Analytics Workspace connected to Sentinel |

---

### IAM lifecycle visual summary

To help visualize how the whole system worked, here’s a high-level diagram of the identity flow:

![IAM Lifecycle Architecture](https://github.com/user-attachments/assets/056c015b-36b8-4b06-b351-978455a755a1)

---

## Final thoughts

This lab gave me a chance to explore IAM in a way that felt realistic. I got to follow how users move through a cloud environment, from being created to being given access and showing up in security logs.

For other students going through this lab, I recommend focusing on:
- How groups are used to manage access
- How role assignments are scoped to different resources
- How to use KQL in Sentinel to investigate what users are doing

I'd like to give a thank you to Josh Madakor and the Cyber Range team for putting the Cyber Range together and making this a strong learning experience.
