# Incident Response Plan

## Purpose

This document describes the general process followed whenever a security event was found or simulated inside this lab. It is written the way a real incident response plan would be written for a small environment, and it is the same process applied throughout every part of this project.

## The six stages followed

| Stage | What happens at this stage |
|---|---|
| Preparation | Sensors, logging, and monitoring tools are already installed and working before anything happens, so evidence exists to look at when needed |
| Identification | Unusual or suspicious activity is noticed, either through an alert, a search, or a manual review of logs |
| Containment | The threat is stopped from continuing or spreading further while investigation continues |
| Eradication | The root cause of the problem is removed, not just the immediate symptom |
| Recovery | Normal, safe operation is restored, and protections are put in place to reduce the chance of the same thing happening again |
| Lessons learned | What happened is written down clearly, along with what worked, what did not, and what should change going forward |

## Preparation used in this lab

Before any attack was simulated, monitoring was already built and tested. This included Sysmon collecting detailed endpoint activity, Splunk making that activity searchable, and Windows own Security log recording account related activity. Without this step already done, none of the later detection work would have been possible.

## Identification used in this lab

Identification came from actively searching Splunk immediately after running a known Atomic Red Team technique, to confirm the resulting activity could be found. In the encoded PowerShell command technique specifically, identification meant recognizing the parent to child process relationship between cmd.exe and powershell.exe as a known evasion pattern, rather than treating it as two unrelated events.

## Containment used in this lab

Containment in this lab was demonstrated at the account level. Once repeated failed logon activity, event 4625, was identified against a test account, the same account activity monitoring that surfaced the failed attempts would, in a real environment, support locking or disabling the account before further attempts succeeded.

## Eradication used in this lab

Eradication in this lab meant identifying the specific technique or gap that allowed suspicious activity to go unnoticed in the first place, such as an audit policy category that was not yet enabled, and correcting it so the same activity would be reliably caught going forward.

## Recovery used in this lab

Recovery meant confirming the fix actually worked by generating the same kind of activity again and checking that it was now correctly detected end to end, in both Event Viewer and Splunk, rather than only trusting that a configuration change had been applied correctly.

## Lessons learned, applied across the whole project

A few points came up repeatedly across every part of this lab and are worth carrying forward into any future security work.

* Permissions matter as much as configuration. Several problems in this project were not caused by wrong settings, but by the tool involved not having the right permission to actually do what it was configured to do.
* Confirm with real evidence, not assumptions. At several points a step looked successful based on a message on screen, but checking the actual resulting log or file showed the real state was different.
* Antivirus and security tools will react to security testing tools. This is expected behavior, not a bug, and needs to be planned for rather than fought against blindly.
* A working pipeline should always be proven with a real test, not just trusted because it was configured correctly. The Atomic Red Team testing stage in this project is exactly that kind of proof.