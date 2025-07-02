# IAM Lifecycle Automation Lab (Cyber Range)

This repository documents a student project completed in the Cyber Range environment. The goal of the lab was to explore identity and access management in a cloud environment by reviewing how users are created, granted access, and monitored using Microsoft Entra ID, Azure, and third party services.

The repository includes a walkthrough of the lab, a set of public facing recommendations, and supporting documentation. It is intended for students, entry level professionals, or anyone learning how cloud identity systems are structured and maintained.

---

## Repository Contents

[`walkthrough.md`](./walkthrough.md) - A detailed review of the lab experience. Includes provisioning logic, group based access control, role assignment, monitoring with Microsoft Sentinel, and example KQL queries.

[`recommendations.md`](./recommendations.md) - General recommendations for improving identity reliability and access visibility.

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

Thanks to the Cyber Range team for providing access to the environment and lifecycle automation scripts. This project was completed using Reader level access at both the tenant and subscription scopes, with a focus on responsible exploration, documentation, and student learning.
