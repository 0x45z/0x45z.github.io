---
layout: default
title: "Azure-based Threat Detection with Suricata IDS and Elastic Stack Integration"
permalink: /projects/networkintrusiondetection
---
# Azure-based Threat Detection with Suricata IDS and Elastic Stack Integration

This project demonstrates the process of implementing an IDS (Suricata) and SIEM (Elastic Stack) to analyse the network traffic from an Azure VM. This enables the detection and remediation of cyberattacks or other malicious activity against the Azure VM. Furthermore, the efficacy of this implementation is evaluated and tested using an industry standard penetration testing tool. 

## Development

Creating the Microsoft Azure virtual machine (VM) running Ubuntu 22.04 LTS and configured with
the Network Watcher extension for Linux.

![Creating VM](/assets/images/project01/1.png)

![Creating VM](/assets/images/project01/2.png)

![Creating VM](/assets/images/project01/3.png)


Connecting to the VM via SSH with the key file and creating a directory to store the packet captures from the Azure network watcher agent.

![Connecting via SSH](/assets/images/project01/4.png)

![Making new directory](/assets/images/project01/5.png)

As this Azure VM lacks computational resources, the intrusion detection system (IDS) and security information and event management (SIEM) will be setup on a local VM that will receive packet captures from the network watcher agent on the Azure VM. This also allows viewing the Kibana dashboard to be easier – which would require a work around if on the Azure VM. This is necessary as Suricata and Elastic Stack require fairly large amounts of RAM, which the Azure VM lacks.

Installing Suricata:

![Installing Suricata](/assets/images/project01/6.png)

![Suricata version](/assets/images/project01/7.png)

Installing and Starting Elasticsearch:

![Installing & Starting Elasticsearch](/assets/images/project01/8.png)

![Checking it is live](/assets/images/project01/9.png)

Setting up and starting Logstash:

![Logstash setup](/assets/images/project01/10.png)

![Starting Logstash](/assets/images/project01/11.png)

Starting Kibana:

![Starting Logstash](/assets/images/project01/12.png)

At this point I encountered an issue where there were no rule violations when running Suricata with a packet capture file. This was because no rules were provided in the default rule file and no other rule files were specified in the configuration file. As a result, there were no rules for Suricata to use. 

![Running Suricata](/assets/images/project01/13.png)

To fix this:

1. Find the default rule path and rule files in the configuration file “/etc/suricata/suricata.yaml”:

![Config file](/assets/images/project01/14.png)

2. In the emerging threats rules file, copy all rule files into the default rule path:

![Copying rules over](/assets/images/project01/15.png)

3. Finally, Copy all the emerging threats rules filenames into the configuration file:

![Config file](/assets/images/project01/16.png)

Setting up a packet capture in Network Watcher, with the output to the created directory on the Azure VM (/var/captures/capture.cap). This will write the packet capture file to that directory so it can later be retrieved to be analysed by Suricata.

![Adding Packet Capture](/assets/images/project01/17.png)

After the network watcher packet capture has finished, the capture file appears in the specified path on the azure VM.

![Packet Capture](/assets/images/project01/18.png)

Transfer the packet capture file to local VM using the secure copy protocol.

![SCP](/assets/images/project01/19.png)

Process the capture file with Suricata:

![Processing the capture file](/assets/images/project01/20.png)

The Kibana dashboard can then be opened in a browser to visualise the alerts from Suricata:

![Kibana Dashboard](/assets/images/project01/21.png)

For clarity, the below diagram shows the process from capturing network traffic to displaying alerts in the Kibana Dashboard.

![Kibana Dashboard](/assets/images/project01/project_diagram.png)

## Evaluation of the Implementation

The intrusion detection system (IDS) developed in this practical experiment monitors the network traffic of a Microsoft Azure virtual machine (VM) through the use of Azure Network Watcher – which provides the ability to capture both ingress and egress packets from the VM. The packet capture file can then be processed by Suricata – an open-source detection engine using signature-based detection. If the signature of a packet matches one in the emerging threats rule set, an alert is raised. Elasticsearch - which is a distributed storage, search, and analytics engine - indexes the alerts raised by Suricata, which are then displayed with charts and graphs using Kibana – creating an easy visualisation of the data. Analysts can then use the 'discover' tab on Kibana to triage the alerts.

This IDS/SIEM setup creates a dashboard for a security operations centre (SOC) analyst or network administrator to monitor network traffic and rule violations to determine an appropriate response. Other than the Azure network watcher – used to capture the traffic, this setup uses free and open- source tools, therefore making it an ideal IDS setup for a small company, organisation or individual that wants an IDS/SIEM to monitor their network assets.

The detection engine of this setup – Suricata, uses signature-based detection, which can detect known threats and indicators of attacks (IoA) by comparing network traffic to known signatures. Relatively novel threats can be detected as long as the rule set is kept up to date (emerging threats rule set in this IDS) and the attack’s signature is located in the rule set.

However, a problem with signature-based detection is that it cannot detect novel threats. Anomaly-based detection utilises machine learning to determine a normalised baseline in the network traffic; through this baseline, abnormal traffic can be identified, and an alert can be triggered. Although this increases the likelihood of false positives, it increases the chance of detecting novel threats. Therefore, this IDS setup can be improved by using anomaly-based detection in addition to the already present signature-based detection; however, Suricata does not provide anomaly-based detection.

Another issue with this IDS setup is that Azure network watcher does not support active monitoring. This is because network watcher needs to be run for a set period of time or until a specific amount of packets are captured; then, when it has finished, it will save the network traffic as a packet capture file, which then needs to be analysed by Suricata. This means that real-time monitoring of the network is not possible with this configuration.

## Testing the Implementation

In order to practically test the effectiveness of this IDS, a real-world test must be performed. To perform this test, while network watcher packet capture was running, the Azure VM was scanned
using Nmap - a tool used to enumerate networked services. The type of scan used was a SYN scan, which is generally considered to be covert and discreet as the full TCP handshake is never completed; this makes them less likely to be detected by an IDS. The `-Pn` flag was also set to ensure host discovery was disabled. The command ran was `sudo nmap -sS -Pn <azure_ip>`. After this scan was completed and the Azure network watcher packet capture had finished, the capture file was scanned by Suricata, and the results were displayed on the Kibana dashboard.

![Results of test](/assets/images/project01/22.png)

The Kibana dashboard displayed multiple detections with the signature: `ET SCAN Potential SSH Scan` on destination port 22. These alerts indicate that the IDS successfully detected the Nmap scan on port 22. In addition to the expected detection of the test scan, other alerts indicated that unauthorised scans also took place, including one from an IP address located in Hangzhou, China, with the signatures: `ET 3CORESec Poor Reputation IP` and `ET SCAN Potential SSH Scan`.

![Scans](/assets/images/project01/23.png)

Although the only scans to be detected were to port 22 (SSH), this is expected as on the Azure VM, only the SSH service (port 22) is exposed to the internet, no other service is reachable; therefore,
these other packets will not reach the VM to be logged and passed to the IDS. This test could, therefore, be improved by hosting other externally accessible services on the VM – such as a web server, and making an exception in the firewall, then performing the same or similar scan.

## Conclusion

To conclude, the open-source tools in the experiment setup work effectively and, although lacking anomaly-based detection, make a sufficient IDS/SIEM. The use of Azure network watcher for packet
capturing, however, means that real-time monitoring is not possible, and as such, this setup cannot be used as a real-world IDS/SIEM. To configure this setup into a functional IDS/SIEM, a different method of packet capturing should be used that would support real-time monitoring.


