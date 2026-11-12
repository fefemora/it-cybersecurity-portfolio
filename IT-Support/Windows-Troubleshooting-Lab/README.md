
# Windows Troubleshooting Lab
## INC-001: Windows Update & Application Update Failure

**Category:** IT Support / Windows Troubleshooting  
**Difficulty:** Intermediate  
**Status:** Resolved  
**Environment:** Windows 11 Pro, Version 25H2  
**Tools:** PowerShell, Event Viewer, Windows Update, Microsoft Store

---

## 1. Overview

This project documents a real troubleshooting scenario involving recurring Windows and application update failures.

The workstation repeatedly requested updates, but some application updates failed to install successfully.

The investigation focused on identifying the errors, reviewing system health, checking Windows Update services, and applying a safe troubleshooting procedure.

The issue was resolved following disk cleanup, a system restart, and another update attempt.

## 2. Problem Description

The user reported that Windows repeatedly requested updates and did not successfully complete the update process.

No significant performance issues, Wi-Fi connectivity problems, or other system failures were reported.

The main objective was to identify the cause of the update failures and restore normal update functionality.

## 3. Technical Environment

| Component | Details |
|---|---|
| Operating System | Windows 11 Pro |
| Version | 25H2 |
| OS Build | 26200.9711 |
| Architecture | 64-bit |
| Storage | KIOXIA 512 GB NVMe SSD |
| File System | NTFS |
| Storage Health | Healthy |
| Primary Diagnostic Tool | Windows PowerShell |

## 4. Initial Assessment

The troubleshooting process started with a general system assessment.

### 4.1 System Information

```powershell
Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsBuildNumber, OsArchitecture
```

An additional registry query was used to verify the Windows version:

```powershell
Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion' |
Select-Object ProductName, DisplayVersion, CurrentBuild, UBR
```

**Findings:**

- Windows 11, version 25H2
- OS build 26200.9711
- 64-bit architecture

Some system information fields displayed a legacy Windows 10 product name, while the build number and version identified Windows 11.

### 4.2 Storage Assessment

```powershell
Get-Volume |
Select-Object DriveLetter, FileSystem, HealthStatus, SizeRemaining, Size
```

Initial storage results:

| Metric | Initial Value |
|---|---|
| Total C: volume capacity | 475.9 GiB |
| Available storage | 18.6 GiB |
| Free space | 3.9% |
| Volume health | Healthy |

The system drive had limited available storage, potentially affecting update installation and general system maintenance.

Physical storage health was also checked:

```powershell
Get-PhysicalDisk |
Select-Object FriendlyName, MediaType, HealthStatus, OperationalStatus, Size
```

The NVMe SSD reported a Healthy status and normal operational state.

## 5. Windows Update Investigation

### 5.1 Checking Update Services

```powershell
Get-Service wuauserv, bits, cryptsvc, usosvc |
Select-Object Name, Status, StartType
```

Results:

| Service | Status | Startup Type |
|---|---|---|
| BITS | Running | Automatic |
| Cryptographic Services | Running | Automatic |
| Update Orchestrator Service | Running | Automatic |
| Windows Update | Stopped | Manual |

The Windows Update service was stopped but configured for manual startup, which is not necessarily abnormal because Windows can start this service on demand.

No obvious service configuration issue was identified.

### 5.2 Reviewing Installed Updates

```powershell
Get-HotFix |
Sort-Object InstalledOn -Descending |
Select-Object -First 10 HotFixID, Description, InstalledOn
```

The system showed recently installed security updates, indicating that at least some Windows updates had been installed successfully.

### 5.3 Reviewing Event Logs

The following PowerShell command was used to identify recent Windows Update errors:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='System'
    ProviderName='Microsoft-Windows-WindowsUpdateClient'
    Level=2
    StartTime=(Get-Date).AddDays(-14)
} -MaxEvents 15 |
Select-Object TimeCreated, Id, Message |
Format-List
```

Five update failures were identified.

**Common error code:** `0x80073D02`

Affected applications included:

| Application | Failed Events |
|---|---|
| Windows Web Experience Pack | 2 |
| Microsoft Office Hub | 2 |
| Microsoft Phone Link | 1 |

The error code is commonly associated with application package updates that cannot complete because required resources are currently in use.

This suggested a possible application update conflict rather than a complete Windows Update service failure.

## 6. Troubleshooting Hypotheses

Two primary hypotheses were considered:

**Hypothesis 1: Application resources in use**

Background processes or applications could have prevented Windows from updating the affected application packages.

**Hypothesis 2: Limited available storage**

The system drive had only 3.9% free space, potentially affecting update operations.

Although limited storage was a relevant risk factor, the available evidence did not establish it as the direct cause of error 0x80073D02.

## 7. Resolution Procedure

A conservative troubleshooting approach was selected to avoid unnecessary changes to Windows services or system components.

### Step 1: Free Up Disk Space

Unnecessary files were removed to recover storage capacity.

No manual deletion of critical Windows system files was required.

### Step 2: Restart Windows

The workstation was restarted to allow running processes to close and release potentially locked application resources.

### Step 3: Retry Updates

After restarting, application updates were retried through Microsoft Store, and pending updates were checked through Windows Update.

### Step 4: Validate Storage

```powershell
Get-Volume -DriveLetter C |
Select-Object DriveLetter, HealthStatus, SizeRemaining, Size
```

Final storage results:

| Metric | Before | After |
|---|---|---|
| Available storage | 18.6 GiB | 63.1 GiB |
| Free space | 3.9% | 13.3% |
| Volume health | Healthy | Healthy |

Approximately **44.5 GiB** of storage was recovered.

## 8. Results and Validation

Following disk cleanup and a system restart:

- Application updates completed successfully.
- Windows Update no longer reported the previous recurring issue.
- Available storage increased significantly.
- The system continued operating normally.

**Final status: Resolved**

The issue was considered functionally resolved based on successful update attempts and the absence of the previously reported problem.

The exact root cause was not conclusively isolated. Both the release of application resources during restart and the increased available storage may have contributed to the successful outcome.

## 9. Technical Evidence

The following files contain sanitized technical evidence collected during the troubleshooting process.

| Evidence | Description |
|---|---|
| [System Information](evidence/system-information.txt) | Initial and post-remediation Windows version and build information. |
| [Storage Validation](evidence/storage-validation.txt) | Storage capacity, available space, and volume health verification. |
| [Update Error Events](evidence/update-errors.txt) | Historical Windows Update error events associated with error 0x80073D02. |
| [Validation Notes](evidence/validation-notes.md) | Functional validation, findings, root cause assessment, and limitations. |

All evidence has been reviewed to avoid exposing personal information or sensitive system identifiers.

The evidence includes historical diagnostic findings and post-remediation observations. It does not establish a single confirmed root cause.


## 10. Security Considerations

The troubleshooting process followed several security and operational principles:

- Avoided unnecessary modifications to Windows services.
- Did not disable security controls.
- Avoided manually deleting critical system files.
- Used built-in Windows diagnostic tools.
- Prioritized low-risk and reversible troubleshooting actions.
- Avoided exposing sensitive system or user information.

## 11. Lessons Learned

This lab reinforced several important troubleshooting practices:

1. Always investigate error messages and event logs before applying fixes.
2. Correlate technical events with the symptoms reported by the user.
3. A stopped Windows service does not automatically indicate a failure.
4. Low disk space can create operational risks even when storage health is normal.
5. Start with low-risk troubleshooting actions before attempting invasive repairs.
6. Verify the outcome and distinguish confirmed findings from technical hypotheses.
7. Document both successful actions and diagnostic limitations.

## 12. Skills Demonstrated

- Windows 11 troubleshooting
- PowerShell diagnostics
- Windows Event Log analysis
- Windows Update troubleshooting
- Service status verification
- Storage health assessment
- Root cause investigation
- Technical documentation
- Incident resolution and validation

---

**Project Status:** Completed  
**Incident ID:** INC-001  
**Category:** IT Support / Windows Troubleshooting
