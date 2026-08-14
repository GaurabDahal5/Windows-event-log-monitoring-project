# MITRE ATT&CK Mapping and Indicators of Compromise

## Purpose

This document pulls together every technique that was actually simulated or observed across this project into one single reference table, mapped to its matching MITRE ATT&CK technique ID, along with the specific indicators of compromise, meaning the exact evidence, that proved each one happened. A note near the end also covers a few well known technique categories that were not part of this lab, explained honestly rather than left out silently.

## What an indicator of compromise means in plain terms

An indicator of compromise, often shortened to IOC, is a specific, concrete piece of evidence that shows something happened on a system. This could be a process name, a full command line, a file path, an account name, or a network address. A good IOC is specific enough that searching for it again would reliably find the same activity if it happened a second time.

## Mapping table

| MITRE ATT&CK ID | Technique name | Where it happened in this lab | Key indicator of compromise |
|---|---|---|---|
| T1059.001 | Command and Scripting Interpreter, PowerShell | Atomic Red Team testing, technique one | Process wsmprovhost.exe created by svchost.exe, described as the host process for WinRM plug ins |
| T1021.006 | Remote Services, Windows Remote Management | Atomic Red Team testing, technique one | Same wsmprovhost.exe event above, which Sysmon itself tagged with this technique ID directly inside the log |
| T1059.001 | Command and Scripting Interpreter, PowerShell, encoded command variant | Atomic Red Team testing, technique two | Process powershell.exe launched with the minus e flag followed by a base64 encoded block of text, itself launched by cmd.exe using the /c flag |
| T1078 | Valid Accounts | Windows authentication monitoring | Successful logon event 4624 for the test account, immediately following several failed logon attempts, event 4625, against the same account |
| T1136 | Create Account | Windows authentication monitoring | New user account event 4720 for the account named testuser, followed by account change event 4738 |

## Full indicator of compromise reference list

| Indicator type | Value | Related technique |
|---|---|---|
| Process name | wsmprovhost.exe | T1059.001 and T1021.006 |
| Parent process | svchost.exe running with the DcomLaunch service group | T1021.006 |
| Process name | powershell.exe | T1059.001 |
| Command line pattern | powershell.exe followed by the minus e flag and a long base64 encoded string | T1059.001 |
| Parent process and command | cmd.exe run with the /c flag, launching the encoded PowerShell command above | T1059.001 |
| Windows Security event ID | 4624, successful logon | T1078 |
| Windows Security event ID | 4625, failed logon | T1078 |
| Windows Security event ID | 4720, new user account created | T1136 |
| Windows Security event ID | 4738, user account changed | T1136 |
| Account name | testuser | T1136 and T1078 |
| Hostname | DESKTOP 175KBI5 | All Windows based findings in this table |

## Technique categories not covered in this lab

Three well known categories of attacker behavior were not simulated as part of this project, and are listed here openly rather than being quietly left out.

| Category | MITRE ATT&CK tactic | Status |
|---|---|---|
| Exfiltration | Data taken out of the environment, for example over a command and control channel or an alternative protocol | Not tested. Would require setting up an outbound transfer of a test file from the Windows machine and confirming the resulting network activity is visible in Sysmon and Splunk |
| Command and control beaconing | Regular, repeated outbound connections a compromised machine makes back to an attacker controlled server | Not tested. Would require standing up a safe, local command and control simulation and confirming the pattern of repeated connections is detectable through Sysmon network events |
| Lateral movement | An attacker moving from one compromised machine to another inside the same network | Not tested, since this lab currently runs a single monitored Windows machine. Would require a second Windows machine on the same network and a technique such as remote service creation or pass the hash between them |

These are listed as a clear next step for extending this lab further, rather than claimed as already done, since no real evidence for any of the three exists yet inside this project.

## Closing note

Every row in the two tables above ties back to a real command that was run and a real log entry that was found afterward, inside either Event Viewer or Splunk. Nothing in this document describes something that was only planned or assumed, apart from the three categories explicitly marked as not tested in the final section.
