#  LSASS Credential Dump Detection – MITRE ATT&CK T1003

This detection engineering project focuses on identifying malicious or suspicious attempts to dump credentials from the LSASS (Local Security Authority Subsystem Service) process. Techniques covered here are mapped to [MITRE ATT&CK Technique T1003](https://attack.mitre.org/techniques/T1003/).

---

## Objective

Detect adversaries attempting to dump credentials from the LSASS process using tools like:

- Mimikatz
- ProcDump
- Rundll32
- Task Manager
- Process Explorer
- Empire
- Dumpert
- SQLDumper.exe

---

## Detection Strategy

The approach is built on identifying anomalies and misuse of process memory access using:

- **Sysmon Event ID 10** – `ProcessAccess`: When a process attempts to access LSASS.
- **Sysmon Event ID 11** – `FileCreate`: For `.dmp` files often generated during LSASS dumps.
- **Child Processes from LSASS** – LSASS spawning child processes is highly anomalous.

