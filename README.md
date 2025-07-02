# IAM Lifecycle Automation Lab (Cyber Range)

This repository documents a student-led project focused on identity and access management (IAM) in a cloud environment. It was completed as part of the Cyber Range using Microsoft Entra ID, Azure, and third-party integrations like Tenable and Gmail.

The goal of the lab was to follow how users are provisioned, assigned access, and monitored across systems — and to make that process easier to understand for others exploring IAM for the first time.

---

## What's in this repo

- [`walkthrough.md`](./walkthrough.md): A day-by-day summary of what I explored, how the automation worked, and what I learned. Includes working KQL queries and a breakdown of the full identity lifecycle.
- [`recommendations.md`](./recommendations.md): A short set of general ideas to help improve IAM visibility and reliability for future lab iterations.

---

## Who this is for

This repo is meant for:
- Students working through the Cyber Range
- Anyone new to cloud IAM concepts
- Early-career professionals looking for a real-world example of identity automation and access control

No prior experience with Entra ID or Azure is required — the walkthrough breaks everything down step by step.

---

## What you’ll learn

By reading through this project, you’ll see how:
- IAM automation can be triggered from something as simple as a Google Sheet
- Entra ID users and groups are used to manage access in Azure
- RBAC is applied through group membership instead of direct user assignments
- Tools like Tenable and Gmail can be integrated into identity workflows
- Sign-in activity and role changes can be queried using KQL in Log Analytics

---

## Acknowledgments

Thanks to Josh Madakor and the Cyber Range team for providing access to the environment and lifecycle automation scripts. This project was completed using Reader level access at both the tenant and subscription scopes, with a focus on responsible exploration, documentation, and student learning.
