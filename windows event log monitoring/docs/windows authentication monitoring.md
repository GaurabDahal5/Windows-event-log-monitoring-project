# Windows Authentication Monitoring

A note on this document. As with the other technical documents in this repository, explanations are written in plain sentences, and symbols such as quotation marks only appear inside actual commands and code, since those are exact values.

## Goal

The goal of this part of the lab was to monitor account related activity on the Windows machine, covering successful logons, failed logons, and new user account creation. This is a different kind of monitoring from the Sysmon work covered earlier, since account activity is recorded by Windows itself inside its own Security log, rather than by Sysmon.

## Why this matters

A huge amount of real world attacker activity eventually touches an account somewhere, whether that is someone trying to guess a password, an attacker creating a new account to keep access after being discovered, or a legitimate account being used at an unusual time. Being able to find and understand this kind of activity is a core skill for anyone working in a security operations role.

## Key event types tracked

| Event ID | What it means |
|---|---|
| 4624 | A successful logon |
| 4625 | A failed logon |
| 4634 | An account logged off |
| 4720 | A new user account was created |
| 4738 | A user account was changed |
| 4740 | An account was locked out after too many failed attempts |
| 4767 | A locked account was unlocked |

## Checking and enabling audit policy

By default, Windows does not record every one of these event types. The current settings were checked using:

```
auditpol /get /category:*
```

This showed that logon and logoff auditing was already turned on, but account management auditing, which covers new user creation and account changes, was not fully enabled. The missing categories were turned on so that user account management events would actually be recorded going forward.

![Logon and logoff audit categories confirmed enabled](../screenshots/windows_authentication_monitoring/01_auditpol_logon_logoff.png)

![Account management audit categories checked and enabled](../screenshots/windows_authentication_monitoring/02_auditpol_account_management.png)

## Generating real test activity

A test account was created using:

```
net user testuser Passw0rd! /add
```

![Test account created and account lockout policy settings confirmed](../screenshots/windows_authentication_monitoring/03_test_user_created.png)

After the account existed, a small number of deliberate failed logon attempts were made against it, followed by a successful logon, to generate a realistic mix of both successful and failed authentication events rather than only one or the other.

## Confirming the events in Event Viewer

Windows own Security log, viewable inside Event Viewer, showed the expected results clearly. Successful logons appeared as event 4624, failed logons appeared as event 4625, the logoff appeared as event 4634, the new account creation appeared as event 4720, and account changes appeared as event 4738.

![Security log showing account management events](../screenshots/windows_authentication_monitoring/04_security_log_events_confirmed.png)

![Security log showing logon, failed logon and logoff events together](../screenshots/windows_authentication_monitoring/05_security_log_more_events.png)

## Extending log forwarding to cover the Security log

The Sysmon work covered earlier only forwarded Sysmon's own log. To also capture account activity, the forwarder's input configuration was extended to include Windows own Security log as an additional source, using the same forwarder that was already running on the machine.

## Confirming the events in Splunk

Once the forwarder was updated, a search was run in Splunk:

```
index=main sourcetype="WinEventLog:Security" earliest=-1h
```

This returned well over one hundred real Security log events, including the successful logons, failed logons, logoffs, and account management activity generated during testing. A saved report was then built inside Splunk, named Windows Auth Detection, Failed and Successful Logons, which pulls together the key authentication event types into one reusable search that could be reused going forward without needing to rebuild it each time.

![Splunk search confirming Security log events successfully forwarded](../screenshots/windows_authentication_monitoring/06_splunk_security_events_confirmed.png)

![Expanded field detail for a forwarded Security log event](../screenshots/windows_authentication_monitoring/07_splunk_security_fields_expanded.png)

![Saved Splunk report combining the key authentication event types](../screenshots/windows_authentication_monitoring/08_saved_splunk_report.png)

## What was intentionally left for future work

Account lockout, event 4740, and account unlock, event 4767, were not triggered during this round of testing, since doing so requires deliberately failing a logon enough times in a row to cross the configured lockout threshold. This is a natural next step for extending this part of the lab further, but was not required to demonstrate the core skill being shown here, which is enabling audit policy correctly, generating real authentication activity, and proving that activity is fully searchable end to end.

## What this demonstrates

This piece of the project shows the ability to work with Windows own built in Security log, correctly diagnose which audit categories need to be turned on, generate real evidence rather than only reading documentation, and extend an already working log forwarding pipeline to bring in a brand new data source.
