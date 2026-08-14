# Sample Incident Report

## Report summary

| Field | Detail |
|---|---|
| Incident type | Obfuscated PowerShell command execution, launched via cmd.exe |
| Affected system | Windows 10 virtual machine, hostname DESKTOP 175KBI5 |
| Detected by | Splunk search of Sysmon process creation events |
| Severity | Medium |
| Status | Detected and confirmed |

## What happened

A PowerShell command was executed using the encoded command flag, meaning the actual command was hidden inside a block of base64 encoded text rather than shown as plain, readable text. The command was launched indirectly, through cmd.exe using the /c flag, rather than PowerShell being run directly. This chain, cmd.exe launching an encoded PowerShell command, is a well known technique real attackers use to avoid detection by anyone glancing at process activity in plain text.

## How it was found

A Splunk search was built against Sysmon process creation events, laid out as a table showing process name, full command line, and parent process side by side. This revealed the complete chain in one view: cmd.exe spawning powershell.exe a fraction of a second later, both carrying the same encoded command block.

## Immediate action taken

The encoded command block was identified as belonging to a known, safe Atomic Red Team test technique rather than a real threat, since this activity was generated deliberately as part of testing the detection pipeline. In a real environment, the next action would be to decode the base64 block to see exactly what the hidden command was attempting to do, and to treat the parent cmd.exe process as equally suspicious as the child PowerShell process.

## Root cause

The underlying pattern being tested was a gap that exists on any system where process command line logging is not enabled or not being reviewed. Without visibility into full command lines and parent to child process relationships, an encoded PowerShell command launched this way would blend into normal system activity.

## Fix put in place

Sysmon was already configured to log full command line detail for every process creation event, and Splunk was already receiving that data through the Universal Forwarder. This meant the detection did not require a new tool or a new configuration change, only a correctly written search query that pulled process name, command line, and parent process together, rather than reviewing events one at a time.

## Evidence collected

* Sysmon process creation event for cmd.exe, showing the full command line including the /c flag and the call to powershell.exe
* Sysmon process creation event for powershell.exe, showing the same encoded command block, timestamped a fraction of a second after the parent event
* Splunk table view showing both events side by side with their timestamps

## Recommendation for the future

Beyond confirming this specific technique can be found, a longer term improvement would be building a saved Splunk alert that automatically flags any process creation event where a command line contains the encoded command flag, so this kind of activity is surfaced immediately rather than needing someone to search for it manually after the fact.

## Timeline

| Time | Event |
|---|---|
| T plus zero seconds | cmd.exe launched with a command line calling powershell.exe with an encoded command block |
| A fraction of a second later | powershell.exe launched as a child process, carrying the same encoded command block |
| Shortly after | Splunk search built to lay out process name, command line, and parent process together |
| Following this | Full parent to child chain identified and confirmed to match the test technique that was run |

## Lessons learned

The first Splunk search run during this investigation returned a result that looked relevant but turned out to be unrelated background activity once checked against the actual timestamp of the test. This was a useful reminder not to treat the first matching event as confirmed evidence without cross checking it against when the activity actually happened. Having a process chain query, showing process name, command line, and parent process together, ready ahead of time made the real detection fast once the search was corrected, rather than requiring the chain to be pieced together manually from separate events.

## Closing note

This report was written using real evidence generated inside a personal lab environment, built specifically to practice the full process a security analyst follows when investigating suspicious process activity, from first finding it in a search through to documenting it clearly enough for another analyst to follow.
