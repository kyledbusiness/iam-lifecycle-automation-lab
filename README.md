# IAM Lifecycle Automation Lab (Cyber Range)

This repository is based on a student lab completed in the Cyber Range. The lab focused on identity and access management (IAM) in a cloud environment. It looks at how users are created, how access is assigned, and how activity is monitored using Microsoft Entra ID, Azure, and a few third party tools.

The repo includes a step-by-step walkthrough of the lab, some public-facing suggestions for improvement, and supporting notes. It is meant for students, entry level professionals, or anyone learning how cloud identity systems work in practice.

---

## Repository Contents

[`walkthrough.md`](./walkthrough.md) - A detailed review of the lab experience. Includes provisioning logic, group based access control, role assignment, monitoring with Microsoft Sentinel, and example KQL queries.

[`recommendations.md`](./recommendations.md) - Suggestions for making the lab even more helpful for students, with ideas for improving reliability, access clarity, and identity visibility.

---

## Tools and Platforms Used

- Microsoft Entra ID  
- Microsoft Graph API  
- Azure role based access control  
- Microsoft Sentinel  
- Log Analytics  
- Tenable.io  
- Google Sheets and Gmail API  
- Python

---

## Learning Outcomes

This project provided an opportunity to practice key identity concepts in a cloud environment, including:

- Understanding how users are provisioned in Microsoft Entra ID  
- Assigning access through groups and scoped roles  
- Extending the identity lifecycle to other platforms like Tenable and Google  
- Writing queries to investigate access behavior in Microsoft Sentinel  
- Reviewing the impact of role assignments and user provisioning in context

---

## Acknowledgments

Thanks to Josh Madakor and the Cyber Range team for providing access to the environment and lifecycle automation scripts. This project was completed using Reader level access at both the tenant and subscription scopes, with a focus on responsible exploration, documentation, and student learning.
