# Windows Event Monitoring with Sysmon and Splunk

A note on this document. The explanations below are written in plain sentences. The only place symbols such as backslashes and quotation marks appear is inside the actual commands and file paths themselves, since those are exact technical values that cannot be changed without breaking them. Everywhere else, plain words are used instead.

## Goal

The goal of this part of the lab was to turn a plain Windows machine into a monitored endpoint, the way a real workstation inside a company network would be watched by a security team. This meant installing a sensor that records detailed system activity, then getting that activity delivered into Splunk so it could be searched like a real analyst would search it.

## Why Sysmon

Sysmon is a free Microsoft tool that watches a Windows machine at a much deeper level than the normal Windows logs do. It records things like every new process that starts, every network connection made by a process, every file created by a process, and changes made to the registry. On its own it writes this information into Windows own Event Viewer, but that is not searchable across many machines and does not scale for real investigation work.

## Installing Sysmon

Sysmon was installed along with a community maintained configuration file that tells it exactly what to watch and, importantly, tags many of the events it records with a matching MITRE ATT&CK technique ID. This meant that once the tool was running, its own logs already pointed toward which attacker technique a piece of activity resembled.

The install command used was:

```
sysmon64.exe -accepteula -i sysmonconfig.xml
```

A successful install shows messages confirming the driver and service both started, along with confirmation that the configuration file was validated against its schema.

![Sysmon installed cleanly with schema validated](../screenshots/windows_event_monitoring/01_sysmon_clean_install.png)

## Confirming Sysmon was working

Before trusting the tool, it was tested directly. Notepad was opened and closed, then Windows own Event Viewer was checked under the Sysmon Operational log. A new event appeared straight away, Event ID 1, described as a process create event, showing notepad as the program that had just run. This confirmed the whole local chain worked, meaning real activity on the machine was actually being captured.

![Event Viewer showing the process create event for notepad](../screenshots/windows_event_monitoring/02_event_viewer_process_create_confirmed.png)

## Building a filtered view

Sysmon on its own produces a very large number of events, most of which are normal background noise from Windows itself. To make investigation realistic, a custom filtered view was built inside Event Viewer, limited to a specific set of event types that matter most for security work, covering process creation, network connections, file creation, and registry changes among others. This gave a much smaller, much more useful list to actually look through.

![Custom Core Detection View built inside Event Viewer](../screenshots/windows_event_monitoring/03_core_detection_view_built.png)

## Getting logs into Splunk

Having logs sit only in Event Viewer is not useful for a real security team, since it only covers one machine and cannot be searched at scale. To solve this, a small program called the Splunk Universal Forwarder was installed on the Windows machine. Its only job is to read specific logs and send them across the network to the Splunk server.

Two configuration files had to be written by hand, since the graphical installer does not offer an option for a custom log source such as Sysmon.

The first file, called outputs.conf, tells the forwarder where to send data. It was placed at:

```
C:\Program Files\SplunkUniversalForwarder\etc\system\local\outputs.conf
```

and contained:

```
[tcpout]
defaultGroup = default autolb group

[tcpout:default autolb group]
server = 10.0.2.15:9997

[tcpout server://10.0.2.15:9997]
```

The second file, called inputs.conf, tells the forwarder what to actually watch. It was placed at:

```
C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf
```

and contained:

```
[WinEventLog://Microsoft Windows Sysmon/Operational]
disabled = false
index = main
sourcetype = WinEventLog:Sysmon
renderXml = false
```

## Problems found and fixed along the way

Several real problems came up during setup, and working through them is part of the value of this project, since production environments rarely go smoothly on the first attempt either.

| Problem | Cause | Fix |
|---|---|---|
| Forwarder could not read the Sysmon log at all | The forwarder service was installed to run as a limited virtual account, which does not have permission to read protected event log channels | Changed the forwarder service to run as the Local System account through the Windows Services tool |
| Zero events reaching Splunk even after the fix above | Old broken checkpoint files were left behind from earlier failed attempts | Deleted the stored checkpoint files for the Windows event log input, then restarted the forwarder service |
| Files kept saving to the wrong folder | Windows quietly changes the file save location or extension when working inside a protected system folder without administrator rights | Opened Notepad and the command prompt as administrator, and used the full exact folder path every time |

The actual error that revealed the permission problem was found by reading the forwarder's own internal log file, which showed the exact reason the Sysmon channel could not be read.

![Forwarder log revealing the access denied error on the Sysmon channel](../screenshots/windows_event_monitoring/04_forwarder_log_permission_error_found.png)

## Confirming the full pipeline worked

Once the service account and checkpoint issues were resolved, fresh activity was generated on the Windows machine, then searched for on the Splunk side using:

```
index=main sourcetype="WinEventLog:Sysmon"
```

The search returned over six thousand real events, each one correctly tagged with a MITRE ATT&CK technique ID from the configuration file. This confirmed the entire chain worked correctly, from an action happening on Windows, through Sysmon recording it, through the forwarder sending it, through to Splunk making it searchable.

![Splunk search confirming over six thousand Sysmon events successfully forwarded](../screenshots/windows_event_monitoring/05_splunk_6300_events_confirmed.png)

## Summary of the pipeline

| Stage | Component |
|---|---|
| Activity happens | Windows machine |
| Activity is recorded in detail | Sysmon |
| Recorded activity is sent onward | Splunk Universal Forwarder |
| Activity becomes searchable | Splunk Enterprise on the Ubuntu machine |

## What this demonstrates

This piece of the project shows the ability to build a real endpoint monitoring pipeline from nothing, including diagnosing and fixing genuine configuration and permission problems along the way, which is exactly the kind of troubleshooting a security operations analyst is expected to be comfortable with.
