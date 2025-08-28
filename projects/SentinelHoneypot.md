---
layout: default
title: "Building a Cloud-Based SOC with Microsoft Sentinel"
permalink: /projects/sentinel
---
# Building a Cloud-Based SOC with Microsoft Sentinel

This project involves the development and deployment of a cloud-based Security Operations Centre (SOC) using Microsoft Azure. Two honeypot VMs are deployed, one Linux VM with an exposed SSH service and one Windows VM with an exposed RDP service and SQL database. Logs from these honeypot VMs are ingested into a Log Analytics Workspace, along with logs from other Azure-based sources such as Entra ID (Azure Active Directory), Blob Storage, Key Vault, and Activity Log. These logs are queried by Microsoft Sentinel to trigger real-time alerts and build maps of ongoing and past cyberattacks. The result of this project is a realistic cloud-based SOC environment that safely enables real-time monitoring, analysis, and response to real cyberattacks. 

![Cloud SOC Diagram](/assets/images/project02/Cloud_SOC_Diagram.png)

This project enables aspiring and junior analysts to understand how Microsoft Sentinel works, practice using its interface and KQL queries, and practice triaging and remediating cyberattacks in a safe environment.



