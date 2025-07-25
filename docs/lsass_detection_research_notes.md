#  LSASS Credential Dumping Detection (T1003)

## **Objective**

Detect adversaries attempting to dump credentials from the **LSASS process**, commonly using tools like **Mimikatz**, **Procdump**, or via **direct memory access**.

---

## Technique: [T1003 – OS Credential Dumping](https://attack.mitre.org/techniques/T1003/)

Credential dumping is a post-compromise technique used by adversaries to extract plaintext passwords, hashes, or Kerberos tickets from the memory of a compromised machine.

---

## Research & Context

Adversaries may attempt to dump credentials to obtain login or credential material, typically found in memory. These credentials can be extracted from:

- OS caches
- LSASS memory
- Active Directory databases (e.g., `ntds.dit`)

The tools used for credential dumping range from **legitimate administrative tools** to **overtly malicious binaries**. Often, attackers abuse trusted Windows binaries (LOLBins) and third-party tools.

###  Common Tools Seen in the Wild

| Type | Tool | Notes |
|------|------|-------|
| Legitimate | Task Manager | Can dump LSASS memory using "Create Dump File" |
| LOLBin | Rundll32 + comsvcs.dll | Calls `MiniDump` export function |
| SysInternals | Procdump, ProcExp, SQLDumper | Commonly abused admin tools |
| Offensive Tools | Mimikatz, Cobalt Strike, Empire, Pwdump, Dumpert | Designed to target LSASS |

---

## Visibility

- Use **Sysmon** (configured to log Process Access Events - Event ID 10)
- Leverage **Event ID 11** for file creations (especially `.dmp` files)
- EDR tools or **Splunk** with Winlogbeat/Filebeat for log ingestion

---

## ⚠Detection Ideas

### 1.  LSASS Process Access via Sysmon (Event ID 10)
```spl
index=sysmon EventID=10 TargetImage="*lsass.exe" GrantedAccess="0x1010"
| stats count by SourceImage, SourceProcessId, TargetImage, TargetProcessId, ImageLoaded
```

##  Suspicious Process Dumping Behavior

**Detection Source**: Sysmon  
**Event ID**: 11 (File Create)

**Splunk Query**:
```spl
index=sysmon EventID=11 TargetFilename="*.dmp"
| stats count by Image, TargetFilename, ProcessId


