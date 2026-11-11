# SOC homelab: log analysis with Sysmon and Splunk

A lab to practise log analysis and detection with Sysmon and Splunk.

All tests are meant for my own virtual machines only, on a separate LAN segment.

## Lab setup

![Lab topology](https://github.com/user-attachments/assets/9f145dd1-5c41-4abb-98d1-483fc3cd3e3a)

| Role | Operating system | IP address | Purpose |
| --- | --- | --- | --- |
| Endpoint | Windows | 192.168.20.10 | Sysmon and Splunk |
| Attacker | Kali Linux | 192.168.20.11 | Generates test activity |

## Setup

1. Built both virtual machines in VMware on a separate LAN segment, with a baseline snapshot.
2. Installed Splunk with an `endpoint` index.
3. Installed Sysmon and sent its logs to that index.

## Test scenario

A simulated reverse shell with Metasploit, using a test file named `Resume.pdf.exe`, followed by account discovery commands such as `net user` and `net group`.

## What to look for

Sysmon event ID 1 shows the process start and event ID 3 shows the network connection.

```spl
index=endpoint Image="*Resume.pdf.exe*" OR CommandLine="*net user*"
```

## Planned

Run the scenario again with more attack techniques and document the Splunk results step by step.

## Tools

VMware, Windows, Sysmon, Splunk, Kali Linux, Metasploit
