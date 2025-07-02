# IAM Lab Recommendations

This page includes a few ideas for how the IAM Lifecycle Automation Lab could be improved or expanded in the future. These suggestions are based on what I saw while working through the lab as a student.

The lab already gives a great overview of how identity and access management works in the cloud. These notes are just ways it could go even further and help others learn even more.

---

## 1. Make the automation more reliable

### What I noticed  
The lab uses automation to create users and assign access. It talks to services like Microsoft Graph, Tenable, and Google. These tools work well most of the time, but sometimes cloud services can fail or time out. When that happens, the script might stop or skip users without much warning.

### What could help  
It would be useful to show students how to:
- Retry API calls that fail the first time
- Log which users failed or were skipped
- Keep going even if one part has a problem

This would make the automation more reliable and show how to build real-world workflows.

---

## 2. Make role assignments easier to understand

### What I noticed  
The script gives access to users by assigning roles in Azure. It works, but some parts of the logic use role IDs (long strings of numbers and letters) that are hard to read. If someone new looks at the code, they might not know what those roles do.

### What could help  
A few small changes could make the role logic easier to follow:
- Use role names in the code (like "Reader" or "Contributor")
- Store role IDs in a list or dictionary
- Assign roles through groups instead of to individual users

This would make it easier for students to understand what access is being granted and how roles work in Azure.

---

## 3. Add more visibility into identity activity

### What I noticed  
The lab connects to Microsoft Sentinel, which is used to monitor what’s happening in the environment. It’s a great tool, but I didn’t see much identity-related data when I searched for things like sign-ins or role changes.

### What could help  
If sign-in logs and audit logs were connected to Sentinel, students could:
- Search for failed or risky logins
- See when users are added to groups
- Track changes to roles or access

It would also be great to include a few example KQL queries or dashboards so students know where to start when looking at logs.

---

## Final thoughts

These are just a few things that could make the lab even more useful for people learning about identity and access management. The lab is already a strong foundation, and these ideas are meant to help it grow and stay realistic for real-world cloud environments.

Big thanks to the Cyber Range team for building it and giving students the chance to learn by doing.
