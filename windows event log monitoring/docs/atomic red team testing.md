# Atomic Red Team Testing

A note on this document. As with the other technical documents in this repository, the explanations are written in plain sentences, and symbols such as backslashes and quotation marks only appear inside actual commands, file paths and code, since those are exact values that must stay as written.

## Goal

Once the Sysmon and Splunk pipeline was proven to work, the next step was to actually simulate real attacker behavior and prove it could be caught. Building a monitoring pipeline is only half the job. The other half is proving it actually catches something.

## What Atomic Red Team is

Atomic Red Team is a library of small, safe test scripts, each one built to copy the exact behavior of a real attacker technique, mapped directly to a MITRE ATT&CK technique ID. Running one of these tests does not cause any real harm, but it produces the same kind of system activity a real attack would produce, which is exactly what is needed to test whether detection actually works.

## Setting it up

The tool is installed and run through PowerShell. Several real obstacles came up while installing it, which are worth recording since they are common problems anyone doing this kind of work will run into.

| Problem | Cause | Fix |
|---|---|---|
| A required supporting module would not load | PowerShell script execution was blocked by the default security policy | Changed the execution policy to allow locally approved scripts to run |
| The test library folder did not exist even though the tool reported a successful install | The install command only sets up the tool itself, and needs a separate flag to also download the actual attack technique test definitions | Reran the install with the flag that pulls down the technique library |
| Windows Defender deleted parts of the downloaded test library | Many Atomic Red Team test files intentionally resemble real attacker tools, which triggers antivirus detection | Added a folder exclusion for the tool's install location, then restored the quarantined files |

## Preparing the machine

One of the tests required PowerShell remoting to be enabled on the machine, which in turn required the network connection to be set to a private network type rather than a public one, since Windows will not allow certain remote management features on a network it considers untrusted. Once the network type was changed, remoting was enabled successfully.

![PowerShell remoting prerequisites confirmed met after fixing the network type](../screenshots/atomic_red_team_testing/02_winrm_enabled_prereqs_met.png)

## Technique one: PowerShell session creation and use

This test opens a PowerShell session on the machine and runs a small command inside it, which mirrors how an attacker might open a remote session to explore or control a system after gaining initial access.

Before running it, the available test options for this technique were listed to see exactly what would run.

![List of available T1059.001 test variations](../screenshots/atomic_red_team_testing/01_t1059_001_test_list.png)

It was run using:

```
Invoke-AtomicTest T1059.001-12
```

![Test twelve executing and completing successfully](../screenshots/atomic_red_team_testing/03_test_12_execution_output.png)

### How it was detected

Searching Splunk for recent process creation events on the Windows machine first surfaced an unrelated event, which was a good reminder to always check that a found event actually lines up with the test that was run before treating it as evidence.

![First Splunk search result, later confirmed to be unrelated background activity](../screenshots/atomic_red_team_testing/04_first_splunk_search_result.png)

Widening the search to look specifically for the WinRM host process surfaced the correct events, timestamped exactly when the test was run.

![Correct wsmprovhost process creation events found](../screenshots/atomic_red_team_testing/05_wsmprovhost_events_found.png)

Expanding the matching event showed the full set of fields needed to document the detection properly.

![Full field detail for the wsmprovhost detection event](../screenshots/atomic_red_team_testing/06_wsmprovhost_fields_expanded.png)

| Field | Value |
|---|---|
| Detected process | wsmprovhost.exe |
| Parent process | svchost.exe |
| Description | Host process for WinRM plug ins |
| Simulated technique | T1059.001, PowerShell Session Creation and Use |
| Technique tag applied by Sysmon itself | T1021.006, Windows Remote Management |

Both technique numbers are worth keeping, since one describes the test that was run, and the other describes what Sysmon itself recognized the resulting activity as, which shows the same event can honestly be described from two connected angles.

## Technique two: PowerShell command execution using an encoded command

This test runs a command through PowerShell using an encoded, or obfuscated, format rather than plain text. Real attackers frequently use this trick specifically to avoid being noticed by anyone glancing at process activity, since the actual command is hidden inside a block of encoded text rather than shown in plain words.

It was run using:

```
Invoke-AtomicTest T1059.001-17
```

### How it was detected

A Splunk search was built to lay out the process name, the full command line, and the parent process side by side for every process creation event in the recent time window. This revealed the full attack chain in one view.

![Table view showing the full cmd.exe to powershell.exe command chain](../screenshots/atomic_red_team_testing/07_technique_two_command_chain_table.png)

| Field | Value |
|---|---|
| Parent process | cmd.exe |
| Parent command line | cmd.exe /c powershell.exe -e (followed by an encoded block of text) |
| Child process | powershell.exe |
| Child command line | powershell.exe -e (the same encoded block of text) |
| Technique | T1059.001, PowerShell Command Execution |

This is a stronger and more realistic piece of detection than the first technique, since it shows both the parent and child process together, which is exactly the kind of evidence a real analyst would use to justify escalating an alert.

## Splunk queries used across this testing

```
index=main sourcetype="WinEventLog:Sysmon" EventCode=1 Image="*wsmprovhost*"
```

```
index=main sourcetype="WinEventLog:Sysmon" EventCode=1 earliest=-10m | table _time, Image, CommandLine, ParentImage
```

## What this demonstrates

This piece of the project proves the monitoring pipeline built earlier actually works against real attacker behavior, not just theoretical activity. It also shows the ability to write targeted search queries that pull out exactly the evidence needed, rather than only reading raw logs line by line.
