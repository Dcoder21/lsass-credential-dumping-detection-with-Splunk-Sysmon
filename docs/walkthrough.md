# 🔍 LSASS Credential Dump Detection - Walkthrough

## 🌟 Goal

Simulate credential dumping from LSASS using Atomic Red Team and validate the corresponding detection rule via Sysmon and Splunk.

---

## 🧰 Prerequisites

- ✅ Sysmon installed and configured with proper `sysmon-config.xml`
- ✅ Splunk (or any SIEM) with Sysmon logs being ingested
- ✅ Atomic Red Team installed (`Invoke-AtomicTest`)
- ✅ Administrative privileges on the test machine

---

## 🚀 Step-by-Step Walkthrough

### 1. Setup Atomic Red Team

```powershell
# Install Atomic Red Team if not done
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing);
Install-AtomicRedTeam -getAtomics

# Import Atomic Red Team functions
Import-Module AtomicRedTeam

# Initialize Atomics Folder
Set-AtomicRedTeamConfig -AtomicRedTeamPath "C:\AtomicRedTeam"
```

---

### 2. Run LSASS Dump Test

Atomic test for LSASS dumping using **Procdump**:

```powershell
Invoke-AtomicTest T1003.001 -TestNumbers 1 -GetPrereqs -Force
Invoke-AtomicTest T1003.001 -TestNumbers 1
```

This uses Sysinternals ProcDump to dump LSASS memory.

> 🔐 Note: You must run PowerShell as Administrator.

---

### 3. Monitor Sysmon Event ID 10 and 11

You’re looking for:

- **Event ID 10** (Process Access): A process accessing `lsass.exe`
- **Event ID 11** (File Create): Creation of `.dmp` file

Sample SPL for Event ID 11:

```spl
index=sysmon EventID=11 TargetFilename="*.dmp"
| stats count by Image, TargetFilename, ProcessId
```

Sample SPL for Event ID 10 (LSASS Access):

```spl
index=sysmon EventID=10 TargetImage="*lsass.exe"
| stats count by SourceImage, TargetImage, GrantedAccess, CallTrace
```

---

### 4. Analyze Output in Splunk

- Confirm that the detection rule triggered.
- Identify the process that accessed or dumped LSASS (e.g., `procdump.exe`)
- Look at time, user, and command line arguments.

---

### 5. Cleanup

To remove the dumped file:

```powershell
Remove-Item C:\Users\Public\lsass.dmp -Force
```

---

## 📌 Notes

- Consider adding command-line logging via Sysmon’s `ProcessCreate` event (Event ID 1).
- Tag your detection rules with MITRE ATT&CK ID: `T1003.001`.
- You can expand by testing other dumpers (e.g., `rundll32`, `comsvcs.dll` method).

---

## ✅ Outcome

You should now be able to:

- Simulate LSASS dumping with Atomic Red Team
- Detect it using custom SPL/Sigma logic
- Correlate detection with visibility from Sysmon logs
