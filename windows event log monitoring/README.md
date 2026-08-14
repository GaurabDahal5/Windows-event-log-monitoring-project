# Windows Event Log Monitoring

## What this project is

This is a home built security operations lab, centered on Windows event log monitoring. The goal was to set up a small environment where real attacker behavior could be simulated and then detected, the same way an analyst in a security operations center would find it. Everything here was built on virtual machines running on one computer, using free and open tools.

The lab covers three connected pieces of work:

1. Full endpoint monitoring on a Windows machine using Sysmon and Splunk
2. Simulated attacker techniques using Atomic Red Team, detected inside Splunk
3. Windows account activity monitoring, covering logons, failed logons and new user creation

## Why this project exists

The purpose was to build hands on proof of being able to do the actual work of a security operations center analyst. Rather than only reading about detection, this project involved building the sensors, breaking things, fixing them, generating real attack traffic, and then proving that traffic could be found again using logs and search queries. Every piece of evidence in this repository came from a real log, not a description of one.

## Lab environment

|Component|Details|
|-|-|
|Hypervisor|Oracle VirtualBox|
|Victim or monitored machine|Windows 10, hostname DESKTOP 175KBI5|
|Log collection and search platform|Splunk Enterprise, running on an Ubuntu virtual machine|
|Log forwarding agent|Splunk Universal Forwarder, installed on the Windows machine|
|Endpoint sensor|Sysmon, using a MITRE ATT\&CK mapped configuration|
|Attack simulation tool|Invoke Atomic Red Team|
|Networking|VirtualBox NAT adapters, both machines able to reach each other|

## Repository layout

|Folder or file|What it contains|
|-|-|
|README.md|This overview|
|docs/windows\_event\_monitoring.md|Setting up Sysmon and forwarding logs into Splunk|
|docs/atomic\_red\_team\_testing.md|Simulated MITRE ATT\&CK techniques and how they were detected|
|docs/windows\_authentication\_monitoring.md|Logon, failed logon and account activity monitoring|
|docs/mitre\_attack\_ioc\_mapping.md|A single reference table mapping every technique in this lab to its MITRE ATT\&CK ID, along with the exact indicators of compromise found for each one|
|incident\_response/incident\_response\_plan.md|The general incident response process used across this lab|
|incident\_response/sample\_incident\_report.md|A full written incident report based on real evidence from this lab|
|screenshots/|Real screenshots taken during the lab, organized into one folder per write up, referenced directly inside each document|

## A note on evidence

Every claim made across the documents in this repository is backed by a real screenshot taken at the time the work was done. These are stored inside the screenshots folder, split into one subfolder per write up, and each document links directly to the specific screenshot that supports the point being made at that spot in the text, rather than leaving the reader to take anything on faith.

## What is proven versus what is planned

This lab currently proves detection across PowerShell based execution techniques, remote session creation, and account related activity such as logons, failed logons and new user creation. Three further categories, data exfiltration, command and control beaconing, and lateral movement between machines, were not tested in this round of the project and are listed openly inside docs/mitre\_attack\_ioc\_mapping.md as clear next steps rather than claimed as already proven.

## High level architecture

The Windows machine generates activity. Sysmon watches that activity and writes detailed event logs. A small forwarder program reads those logs and sends them across the network to the Splunk server running on the Ubuntu machine. Once the logs arrive in Splunk, they can be searched, filtered and turned into saved reports, the same way a real analyst would investigate an alert.

For the account activity monitoring piece, Windows own built in Security log was used instead of Sysmon, since login and account events are recorded there by Windows itself once the correct audit policy is turned on.

## Skills demonstrated

* Building and troubleshooting a log pipeline across two virtual machines
* Installing and configuring Sysmon with a technique mapped configuration
* Installing and configuring a Splunk Universal Forwarder, including fixing permission and service account issues
* Writing and running Splunk search queries to find specific evidence
* Simulating real MITRE ATT\&CK techniques safely inside an isolated lab
* Enabling and interpreting Windows audit policy and the Windows Security log
* Documenting technical work in a way another analyst could follow and reproduce
* ## 

