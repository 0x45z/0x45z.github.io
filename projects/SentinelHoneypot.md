---
layout: default
title: "Building a Cloud-Based SOC with Microsoft Sentinel"
permalink: /projects/sentinel
---
# Building a Cloud-Based SOC with Microsoft Sentinel

This project involves the development and deployment of a cloud-based Security Operations Centre (SOC) using Microsoft Azure. Two honeypot VMs are deployed, one Linux VM with an exposed SSH service and one Windows VM with an exposed RDP service and SQL database. Logs from these honeypot VMs are ingested into a Log Analytics Workspace, along with logs from other Azure-based sources such as Entra ID (Azure Active Directory), Blob Storage, Key Vault, and Activity Log. These logs are queried by Microsoft Sentinel to trigger real-time alerts and build maps of ongoing and past cyberattacks. The result of this project is a realistic cloud-based SOC environment that safely enables real-time monitoring, analysis, and response to real cyberattacks. 


![Cloud SOC Diagram](/assets/images/project02/Cloud_SOC_Diagram.png)

This project enables aspiring and junior analysts to understand how Microsoft Sentinel works, practice using its interface and KQL queries, and practice triaging and remediating cyberattacks in a safe environment.


## Demonstration with Simulated Cyberattacks




## Demonstration with Real Cyberattacks

The Windows and Linux VM were left running for approximately 24 hours.  As a result, 252 alerts were triggered from brute force attacks against the RDP and SSH service on the Windows and Linux VM respectively.

![Alert Dashboard](/assets/images/project02/2.png) 

The attack maps illustrate the attacks by plotting them on a map according to the geolocation of the attacking IP address.

![RDP Attack Map](/assets/images/project02/3.png) 

![SSH Attack Map](/assets/images/project02/4.png) 

From this data, it can be seen that (for this specific case) there are a greater number and distribution of attacks against SSH than RDP. However, the attacks against RDP are performed with greater intensity, with the highest number of attempts from a single IP for RDP being 6,600, whereas the highest number of attempts against SSH is only 876.

## Triaging an Alert

Triaging the most recent alert, it can be seen to be a brute force attempt against RDP from the IP `57[.]129[.]140[.]32`.

![RDP Alert](/assets/images/project02/5.png) 

There is a total failure count of 19 attempts for this alert. However, this rule queries the Log Workspace every 5 minutes for logs in the last 5 minutes, meaning that the 19 attempts occurred in the last 5 minutes from the alert. Querying the logs, it can be seen that this IP is responsible for 2427 brute force attempts.

![KQL Query](/assets/images/project02/6.png) 

The malicious IP is an OVH Virtual Private Server (VPS) located in London. 

![IP Lookup](/assets/images/project02/7.png) 

Despite the large number of brute force attempts, this IP has only been reported once on AbuseIPDB, with the report stating the IP was responsible for an RDP attack. At the time of writing, this report was submitted 20 days ago, which suggests that the threat actor has been performing RDP brute force attacks with this IP for at least 20 days. 

![IP Reputation](/assets/images/project02/8.png) 

This threat actor is attempting the brute force the accounts `\ADMINISTRATOR` and `\ADMIN only`.

![Targets](/assets/images/project02/9.png) 

Although this is a honeypot, in the real-world remediation steps could include:
- Block the IP
- Report IP to abuse database such as AbuseIPDB
- Report abuse to cloud/service provider (in this case OVH)
- Assess infrastructure -> Does this service need to be exposed?
- Implement auto-remediation for brute force attacks via SOAR or systems such as Fail2Ban

For this example, I took the time to submit an abuse report to OVHCloud as this IP has likely been used for RDP brute force attacks for multiple weeks, going largely unreported. 

![Report](/assets/images/project02/10.png) 

Given the quantity of brute force attacks, however, reporting every single malicious IP in this way would not be feasible. However, sites like AbuseIPDB have APIs to set up automatic reporting.




