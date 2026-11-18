# Security Baseline — Assessment Register

[Project overview](../README.md) · [Baseline guide](baseline-guide.md) · [Validation plan](validation-plan.md)

**Assessment phase:** Active. **CHG-001 Security log size change implemented and immediate after-state observed; seven-day retention target remains unverified.**

| Control domain | Objective | Directly observed in this project | Provisional next step | Status |
|---|---|---|---|---|
| Firewall | Confirm native protection across network profiles | Initial local query: all three `Enabled=True`, action fields `NotConfigured`; `ActiveStore`: all three `Enabled=True`, default inbound `Block`, outbound `Allow` | If required, examine scoped rule exceptions and active network profile without exposing addresses | **Profile defaults verified; no issue identified in checked fields** |
| Antivirus | Review on-device malware protection status | Service, antivirus, real-time, Tamper Protection, NIS: `True`; signatures age `0`, updated `2026-11-17 09:41:14`; MAPS `2` Advanced, safe sample submission `1`, PUA Block `1`, Block at First Sight not disabled (`False`) | Evaluate current audit policy using read-only commands; consider privacy of cloud reporting | **Verified selected health and advanced-protection indicators; no deficiency identified in fields checked** |
| Security log retention | Ensure enough history for investigations | **CHG-001:** 20 MiB → **128 MiB**, enabled and Circular; post-query **33,989** records; immediate observed span **1.96 days** (pre-change **1.93 days**) | Recheck endpoints after meaningful elapsed time; compare against proposed 7-day target | **CHG-001 configuration and immediate access verified; long-term target pending** |
| Process creation auditing | Improve process activity visibility if justified | **Fresh local `auditpol` check: `Sin auditoría` / No Auditing**; same setting in prior lab | Review a controlled, success-only auditing pilot with retention, privacy, policy management and rollback considerations | **Current gap verified; change not implemented** |
| PowerShell logging policy and event channel | Determine Script Block Logging policy and event-channel state | **HKLM ScriptBlockLogging policy key absent**; Operational **Enabled=True**, **808 records**, **15 MiB Circular**; **438 retained 4104 records**, and **11 IDs totaling 808 events** in distribution | Check HKCU ScriptBlockLogging policy, then assess scope without reading event contents | **4104 observed; comprehensive Script Block Logging policy not established** |
| Identity & access | Review local role exposure in a privacy-safe way | None | Define narrow, redacted query only when needed | **Not assessed** |
| Update posture | Identify missing or stale protection within defined scope | None | Choose supported read-only check | **Not assessed** |
| Platform protections | Review relevant built-in security features | **BitLocker C: FullyDecrypted/Off/0%; TPM flags all True; firmware UEFI; system disk 0 GPT (`IsBoot`/`IsSystem=True` on disk); Secure Boot False** | Check independent backup and recovery prerequisites before separate approval | **UEFI/GPT/TPM verified; Secure Boot and disk encryption remain off; no changes** |

## Verified firewall evidence — Initial query

```powershell
Get-NetFirewallProfile |
    Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction |
    Format-Table -AutoSize
```

The actual output returned three profiles (`Domain`, `Private`, `Public`), all `Enabled=True`, and `NotConfigured` for both default-action properties across the three. This initial result did **not** establish effective default actions.

## Verified firewall evidence — ActiveStore follow-up

```powershell
Get-NetFirewallProfile -PolicyStore ActiveStore |
    Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction |
    Format-Table -AutoSize
```

| Profile | Enabled | ActiveStore default inbound | ActiveStore default outbound |
|---|---|---|---|
| Domain | True | Block | Allow |
| Private | True | Block | Allow |
| Public | True | Block | Allow |

**Assessment:** The `ActiveStore` query resolves the ambiguous local property values: all three profiles have default **Block inbound / Allow outbound**. The inspected defaults do not identify a security weakness. This is not a rule-level inspection or a test of actual network traffic; exceptions and other controls may apply. **No firewall rules or settings have been changed.**

## Verified Microsoft Defender Antivirus status — Step 2

**Exact executed query:**

```powershell
Get-MpComputerStatus |
    Select-Object AMServiceEnabled,
                  AntivirusEnabled,
                  RealTimeProtectionEnabled,
                  AntivirusSignatureAge,
                  AntivirusSignatureLastUpdated |
    Format-List
```

| Property | Verified value |
|---|---|
| `AMServiceEnabled` | `True` |
| `AntivirusEnabled` | `True` |
| `RealTimeProtectionEnabled` | `True` |
| `AntivirusSignatureAge` | `0` days |
| `AntivirusSignatureLastUpdated` | `2026-11-17 09:41:14` (console local time) |

**Assessment:** The selected Microsoft Defender Antivirus health indicators were positive at the time of observation. This is not an antivirus efficacy test, threat scan, policy compliance assessment, or assessment of features not queried. **No antivirus configuration or signatures were modified by this command.**

## Step 3 — Verified Microsoft Defender advanced-protection configuration

**Read-only commands executed:**

```powershell
Get-MpComputerStatus |
    Select-Object IsTamperProtected,
                  RealTimeProtectionEnabled,
                  NISEnabled |
    Format-List

Get-MpPreference |
    Select-Object MAPSReporting,
                  SubmitSamplesConsent |
    Format-List
```

**Real sanitized output:**

```text
IsTamperProtected         : True
RealTimeProtectionEnabled : True
NISEnabled                : True

MAPSReporting        : 2
SubmitSamplesConsent : 1
```

**Interpretation:** Tamper Protection, real-time protection, and Network Inspection System reported `True`. `MAPSReporting=2` corresponds to **Advanced** cloud protection membership; `SubmitSamplesConsent=1` corresponds to **automatic submission of safe samples** under Microsoft Defender preference definitions. These settings were read, not modified.

**Limitations:** Preferences and status flags do not independently prove cloud connectivity, malware detection efficacy, or complete endpoint compliance. Advanced MAPS reporting may share more detailed threat-related information; any policy changes should account for privacy and business requirements. No changes or malware tests were performed.

**Microsoft reference:** [Configure cloud protection](https://learn.microsoft.com/en-us/defender-endpoint/cloud-protection-configure) · [Set-MpPreference parameter values](https://learn.microsoft.com/en-us/powershell/module/defender/set-mppreference).

## Step 4 — PUA Protection and Block at First Sight preferences (verified)

**Exact read-only command executed:**

```powershell
Get-MpPreference |
    Select-Object PUAProtection,
                  DisableBlockAtFirstSeen |
    Format-List
```

**Observed console output:**

```text
PUAProtection           : 1
DisableBlockAtFirstSeen : False
```

**Interpretation:** `PUAProtection=1` is the **Block** configuration for potentially unwanted applications. `DisableBlockAtFirstSeen=False` means the Block at First Sight capability is **not disabled by this preference**. This is a favorable configuration in the reviewed fields.

**Limits:** Preference values do not independently establish end-to-end Block at First Sight operation, cloud connectivity, or malware detection effectiveness. No file was submitted, executed, scanned or quarantined in this exercise. **No settings were changed.**

**Microsoft references:** [Configure PUA protection](https://learn.microsoft.com/en-us/defender-endpoint/detect-block-potentially-unwanted-apps-microsoft-defender-antivirus) · [Configure Block at First Sight](https://learn.microsoft.com/en-us/defender-endpoint/configure-block-at-first-sight-microsoft-defender-antivirus).

## Step 5 — Verify Process Creation audit policy (verified)

**Read-only command actually executed:**

```powershell
auditpol /get /subcategory:"{0cce922b-69ae-11d9-bed3-505054503030}"
```

**Actual console output (Spanish Windows locale):**

```text
Directiva de auditoría del sistema
Categoría o subcategoría                  Configuración
Seguimiento detallado
  Creación del proceso                    Sin auditoría
```

**Interpretation:** The advanced audit subcategory **Process Creation** is currently configured as **No Auditing**. This independently confirms the same setting observed in the [prior Windows Security Event Analysis](../../Windows-Security-Event-Analysis/README.md). With this subcategory disabled, routine process creation auditing is not configured to provide continuous Security event 4688 coverage. The earlier retained 4688 records remain unexplained; they are not proof that audit policy changed or that malicious processes ran.

**Security impact and limits:** This is a **monitoring visibility gap**, not proof of an attack or a compliance finding. The local Security log was previously observed at **20 MiB / Circular** with ~2.03 days between oldest/newest retained events; additional process events could accelerate log turnover if retention capacity is unchanged. Command-line inclusion in 4688 is a separate, potentially sensitive setting and was **not assessed or enabled**.

**Change status:** **Proposed only.** No audit configuration changes have been made during this hardening lab.

## Step 6 — Verify Security log capacity (verified; read-only)

**Exact command executed:**

```powershell
Get-WinEvent -ListLog Security |
    Select-Object LogName,
                  RecordCount,
                  MaximumSizeInBytes,
                  LogMode |
    Format-List
```

**Observed console result:**

```text
LogName            : Security
RecordCount        : 33959
MaximumSizeInBytes : 20971520
LogMode            : Circular
```

**Assessment:** The Security log was configured with a **20 MiB maximum** and **Circular** retention, holding **33,959 records** at inspection time. A separate previous case recorded **33,898 records**, an apparent difference of **61**. These were taken at different moments and, with circular retention, **the difference is not a count of newly generated events**; older records may have been replaced.

**Limitations:** `MaximumSizeInBytes` is a configured maximum, not free capacity. This query does not show the earliest retained event, actual occupied bytes, the frequency of rollover, or whether retention is adequate for a defined goal. No changes were made.

### Step 6b — Refresh oldest/newest retained timestamps (verified)

```powershell
$oldest = Get-WinEvent -LogName Security -Oldest -MaxEvents 1
$newest = Get-WinEvent -LogName Security -MaxEvents 1

[PSCustomObject]@{
    OldestEvent  = $oldest.TimeCreated
    NewestEvent  = $newest.TimeCreated
    RetainedDays = [math]::Round(
        ($newest.TimeCreated - $oldest.TimeCreated).TotalDays, 2
    )
}
```

**Purpose:** Determine the currently observed timestamp span before considering a change that may increase event volume. This does **not** by itself measure how much disk capacity remains.

**Verified console output:**

```text
OldestEvent        NewestEvent         RetainedDays
-----------        -----------         ------------
16-11-2026 0:52:07 17-11-2026 23:06:29         1,93
```

**Assessment:** The observed earliest-to-latest retained event timestamp span is **1.93 days** (November 16, 2026, 00:52:07 to November 17, 2026, 23:06:29, local console time). A separate earlier observation reported **2.03 days** (November 15, 2026, 20:02:39 to November 17, 2026, 20:40:43). The difference is **0.10 days (~2 hours 24 minutes)** between distinct measurements, not an observed data-loss rate, proof of a particular overwrite event, or a consistent daily trend. Current log capacity remains the separately observed **20 MiB maximum, Circular**.

**Limitations:** This measures a span between retained endpoints, **not** disk space free, completeness of events over that interval, or guaranteed future retention. A size change remains proposed only.



## Step 7 — Check free space on system volume (verified)

**Read-only command executed:**

```powershell
[PSCustomObject]@{
    FreeGiB = [math]::Round((Get-PSDrive -Name C).Free / 1GB, 2)
}
```

**Observed result (locale decimal comma):**

```text
FreeGiB
-------
  56,81
```

**Finding:** The system volume **C:** had **56.81 GiB free** at query time. This is a disk-capacity snapshot, not Security log free capacity or a guarantee of available storage at a future date.

**Proposed sizing pilot (NOT applied):** Review a **128 MiB Security log maximum** (134,217,728 bytes), replacing the current **20 MiB** limit, for an initial measured trial. The increase to the maximum is **108 MiB**. A **7-day retention window** is a *proposed target*, **not a predicted or guaranteed result**, especially if Process Creation auditing is enabled later and event volume rises.

**Change constraints:** Confirm device management/authority and collect a fresh pre-change snapshot. Enlarge only the Security log maximum; do not clear events, change the log mode, or enable 4688 auditing simultaneously. Independently verify the applied maximum and observe the retained time span on subsequent days. Recovery would restore the previous 20 MiB configuration only after checking whether a reduction risks record loss or conflicts with device management policies.

## CHG-001 — Security event log maximum size increase (implemented and configuration verified)

**Authorization:** Workstation owner expressly confirmed personal ownership and authorized changing **only** the Security log maximum size from 20 MiB to 128 MiB.

**Pre-change evidence:** `LogName=Security`; `RecordCount=33959`; `MaximumSizeInBytes=20971520` (**20 MiB**); `LogMode=Circular`. The prior `IsEnabled` state was not captured in this particular pre-change query.

**Executed command** (user PowerShell; no error printed):

```powershell
wevtutil sl Security /ms:134217728
```

**Executed post-change verification:**

```powershell
Get-WinEvent -ListLog Security |
    Select-Object LogName,
                  IsEnabled,
                  RecordCount,
                  MaximumSizeInBytes,
                  LogMode |
    Format-List
```

**Actual output:**

```text
LogName            : Security
IsEnabled          : True
RecordCount        : 33989
MaximumSizeInBytes : 134217728
LogMode            : Circular
```

**Validated outcome:** The configured maximum grew from **20 MiB to 128 MiB** (6.4x, **+108 MiB**), Security is enabled in the post-change query, and the log remains **Circular**. The later record count is 33,989 versus 33,959 in the earlier separate check, **not evidence of precisely 30 new events** because circular overwrite may occur. No clear-log command was part of the recorded steps; individual event continuity was **not** independently verified.

**Not yet validated:** The proposed **7-day retained-history target**. A capacity increase alone does not prove days retained, and the log needs a follow-up observation over time. The independently verified pre-change retained timestamp span was **1.93 days**, measured before this change. **Process Creation auditing remains a separate, unimplemented proposal**, last observed as `No Auditing` before CHG-001.

**Change status:** **Implemented and post-change maximum-size configuration verified**. Long-term retention outcome **pending**; no rollback performed.

### CHG-001 — Immediate post-change retention observation (verified)

**Read-only query executed:**

```powershell
$oldest = Get-WinEvent -LogName Security -Oldest -MaxEvents 1
$newest = Get-WinEvent -LogName Security -MaxEvents 1

[PSCustomObject]@{
    OldestEvent = $oldest.TimeCreated
    NewestEvent = $newest.TimeCreated
    OldestRecordId = $oldest.RecordId
    NewestRecordId = $newest.RecordId
    RetainedDays = [math]::Round(($newest.TimeCreated - $oldest.TimeCreated).TotalDays, 2)
} | Format-List
```

**Actual console result:**

```text
OldestEvent    : 16-11-2026 0:52:21
NewestEvent    : 18-11-2026 0:00:06
OldestRecordId : 2256643
NewestRecordId : 2291047
RetainedDays   : 1,96
```

**Comparison:** Before CHG-001, the oldest observed timestamp was **2026-11-16 00:52:07**, newest **2026-11-17 23:06:29**, and rounded observed span **1.93 days**. At immediate follow-up, the oldest observed timestamp was **2026-11-16 00:52:21** (14 seconds later) and newest **2026-11-18 00:00:06**, with a rounded observed span **1.96 days**. This shows that events with timestamps predating CHG-001 remained accessible after the setting change. It is **not** proof of uninterrupted event-by-event continuity or that the added capacity already extends retention to 7 days. Record IDs are metadata and were not checked for gaps or correlated to the pre-change IDs.

**Validation state:** Change applied; new 128 MiB maximum and enabled/Circular state confirmed; immediate timestamp endpoints recorded. **Seven-day retention target remains pending a later, time-separated observation**, with Process Creation auditing still not enabled in this project. No rollback performed.

## Step 8 — BitLocker system-volume encryption baseline (verified; read-only)

**Exact executed command:**

```powershell
Get-BitLockerVolume -MountPoint C: |
    Select-Object MountPoint,
                  VolumeStatus,
                  ProtectionStatus,
                  EncryptionPercentage |
    Format-List
```

**Actual sanitized console output:**

```text
MountPoint           : C:
VolumeStatus         : FullyDecrypted
ProtectionStatus     : Off
EncryptionPercentage : 0
```

**Finding:** The C: system volume was **fully decrypted (0% encrypted)** and its **BitLocker protection was Off** when queried. This is a verified **data-at-rest confidentiality exposure** if an attacker gains physical possession of the device/storage. It does not prove compromise or that every file or removable volume is unprotected. The command does not assess other volumes or other encryption solutions.

**Change status:** **No encryption change performed or authorized.** Encryption should not be enabled without confirming recovery readiness, TPM and Windows compatibility, storage and backup condition, and a safe offline recovery-key handling process. Recovery passwords and key-protector details must never be committed to this public portfolio.

**Next suggested read-only prerequisite:**

```powershell
Get-Tpm | Select-Object TpmPresent, TpmReady, TpmEnabled, TpmActivated | Format-List
```

**Limitations:** TPM readiness alone does not validate BitLocker usability or backup/recovery safety. Consider management policy, Windows edition, firmware boot mode and a tested backup before any authorized implementation.

## Step 9 — TPM readiness (verified; read-only)

**Actual executed command:**

```powershell
Get-Tpm |
    Select-Object TpmPresent,
                  TpmReady,
                  TpmEnabled,
                  TpmActivated |
    Format-List
```

**Observed console output:**

```text
TpmPresent   : True
TpmReady     : True
TpmEnabled   : True
TpmActivated : True
```

**Assessment:** A TPM is present, ready, enabled and activated according to the Windows `Get-Tpm` status fields. This resolves one platform prerequisite for a potential BitLocker deployment but does **not** validate the complete startup/recovery path, backup reliability, BitLocker policy configuration or availability of a recovery key. The C: system volume remains **fully decrypted and unprotected by BitLocker** in the most recent volume query. No TPM or BitLocker settings were modified.

**Next read-only prerequisite:** Inspect boot mode and Secure Boot support using `Confirm-SecureBootUEFI` (a result of `False` needs context; errors can indicate an unsupported/non-UEFI environment). Do not publish recovery passwords or TPM key material. Verify recovery and backups before any separate authorization for encryption.

## Step 10 — Secure Boot status (verified; read-only)

**Command executed:** `Confirm-SecureBootUEFI`

**Actual result (user confirmed):**

```text
False
```

**Interpretation:** Windows reported **Secure Boot disabled**. This is a verified boot-integrity configuration gap, not evidence of compromise or a malfunctioning TPM. The `Get-Tpm` query independently returned `True` for presence, readiness, enabled and activated; TPM state does not establish Secure Boot state.

**Limitations and safe next steps:** This command does not document firmware settings, signed boot components, storage partition layout, boot-loader compatibility, or why Secure Boot is disabled. **No firmware changes, BitLocker activation, or TPM changes were authorized or performed.** Before considering a separate change, verify actual BIOS mode via a read-only Windows query, record recovery/backup readiness, and evaluate firmware and Hyper-V compatibility. A manual firmware toggle without reviewing boot compatibility may cause boot or recovery problems.

## Step 11 — Firmware boot mode (verified; read-only)

**Exact executed command:**

```powershell
Get-ComputerInfo |
    Select-Object BiosFirmwareType |
    Format-List
```

**Actual console output:**

```text
BiosFirmwareType : Uefi
```

**Interpretation:** Windows reports that the current boot firmware mode is **UEFI**. Combined with the separate verified `Confirm-SecureBootUEFI=False`, this supports the distinction between UEFI boot mode and **disabled Secure Boot**. The `Get-Tpm` flags are also all `True`, while C: BitLocker is `FullyDecrypted` / `ProtectionStatus=Off` / 0% encrypted at their respective observations.

**Limits:** This single property does not establish whether Secure Boot can be safely enabled on this installation; boot-loader signatures, firmware keys, disk partition style, backup/recovery readiness, and hardware compatibility were not inspected. No firmware or encryption settings were changed.

**Proposed next read-only check:** Inspect the disk containing C: without exposing partitions or file content:

```powershell
Get-Partition -DriveLetter C |
    Get-Disk |
    Select-Object Number, PartitionStyle, IsBoot, IsSystem |
    Format-List
```

**Caution:** Disk-layout inspection is not permission to alter partitions, firmware, or BitLocker. Complete backups and recovery planning before proposing any change.

## Step 12 — Windows system disk partition style (verified; read-only)

**Exact executed command:**

```powershell
Get-Partition -DriveLetter C |
    Get-Disk |
    Select-Object Number, PartitionStyle, IsBoot, IsSystem |
    Format-List
```

**Actual sanitized output:**

```text
Number         : 0
PartitionStyle : GPT
IsBoot         : True
IsSystem       : True
```

**Interpretation:** Windows reported the disk containing C: as **Disk 0, GPT**, with the returned disk object reporting `IsBoot=True` and `IsSystem=True`. This aligns with the separately verified UEFI boot mode. GPT and UEFI are favorable baseline conditions, but **do not establish that Secure Boot can be safely enabled**: `Confirm-SecureBootUEFI` previously returned `False`. The TPM status flags were all `True`; the system volume remained unencrypted at its last BitLocker query.

**Limits:** `IsBoot` and `IsSystem` here are **disk-level properties returned by Get-Disk**, not a claim about the C: partition's special attributes. This query does not establish that EFI boot signatures, firmware key enrollment, recovery media, backup verification, or Windows recovery paths are ready. **No firmware, disk partition, TPM or BitLocker change was performed.**

**Next step:** Confirm the user has a current independent backup and a usable recovery method before planning any Secure Boot or BitLocker change. Never collect or publish BitLocker recovery secrets.

## Step 13 — Windows PowerShell Script Block Logging policy key (verified; read-only)

**Exact command executed:**

```powershell
$path = 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging'

if (Test-Path $path) {
    Get-ItemProperty -Path $path |
        Select-Object EnableScriptBlockLogging |
        Format-List
} else {
    'Policy registry key: Not configured'
}
```

**Actual observed output:**

```text
Policy registry key: Not configured
```

**Interpretation:** The queried **machine-level Windows PowerShell ScriptBlockLogging policy registry key was absent** at the specified HKLM path. No explicit machine policy value was observed at that location. This is **not proof of complete Script Block Logging inactivity**: other policy scopes (including HKCU), PowerShell editions and specific automatic script-block logging behaviors were not tested. Absence of a policy key is not, on its own, a confirmed vulnerability.

**Limits/privacy:** No event payloads or script text were inspected, no registry settings were modified, and no logging configuration was enabled. If investigating event 4104 later, report only event counts and configuration metadata until any potentially sensitive script contents can be safely excluded.

**Next read-only check proposed:**

```powershell
Get-WinEvent -ListLog 'Microsoft-Windows-PowerShell/Operational' |
    Select-Object LogName, IsEnabled, RecordCount, MaximumSizeInBytes, LogMode |
    Format-List
```

This tests the channel's availability and status only, not whether full Script Block Logging policy is enabled.

## Step 14 — Windows PowerShell Operational channel metadata (verified; read-only)

**Executed command:**

```powershell
Get-WinEvent -ListLog 'Microsoft-Windows-PowerShell/Operational' |
    Select-Object LogName,
                  IsEnabled,
                  RecordCount,
                  MaximumSizeInBytes,
                  LogMode |
    Format-List
```

**Actual observed output:**

```text
LogName            : Microsoft-Windows-PowerShell/Operational
IsEnabled          : True
RecordCount        : 808
MaximumSizeInBytes : 15728640
LogMode            : Circular
```

**Assessment:** The Windows PowerShell Operational event channel is **enabled**, holds **808 records**, and is configured with a **15 MiB maximum (15,728,640 bytes)** and **Circular** retention. This confirms channel availability and the presence of some operational events, **not** that all 808 records represent script executions, or that detailed Script Block Logging is enabled. Step 13 independently found no HKLM Windows PowerShell ScriptBlockLogging policy key at the queried location. Other scopes and PowerShell editions were not assessed.

**Privacy/limits:** No event content, command lines, usernames or script-block text was read. A current event-channel snapshot cannot establish whether event ID 4104 exists or comprehensive logging is active.

**Next proposed read-only check — 4104 count only:**

```powershell
$log = 'Microsoft-Windows-PowerShell/Operational'
$events = Get-WinEvent -FilterHashtable @{ LogName = $log; Id = 4104 } -ErrorAction SilentlyContinue
[PSCustomObject]@{ EventId = 4104; Count = @($events).Count } | Format-List
```

For reduced output and effort, use an API-level count, not the script text or full event objects. This query is proposed, not yet executed; absence of retained 4104 would not prove that no event ever occurred.

## Step 15 — Retained PowerShell event 4104 count (verified; read-only)

**Exact executed command:**

```powershell
$count = (Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-PowerShell/Operational'
    Id = 4104
} -ErrorAction SilentlyContinue | Measure-Object).Count

[PSCustomObject]@{
    EventId = 4104
    Count = $count
} | Format-List
```

**Actual console output:**

```text
EventId : 4104
Count   : 438
```

**Interpretation:** The queried Windows PowerShell Operational channel currently retains **438 events with ID 4104**, consistent with some script-block-related event generation. The preceding channel-metadata query reported **808 total records** at a separate point in time. These are separate snapshots; counts can change in a circular log. `4104` records are **not necessarily unique scripts**, as multipart and other recording behaviors are possible. The **HKLM ScriptBlockLogging policy key was absent** at the queried path, so this result does **not** prove comprehensive Script Block Logging policy is enabled; other scopes and automatic logging behavior remain unassessed.

**Security/privacy limits:** This check only counted event objects. It did **not** inspect any command text, scripts, user names, messages or file paths. The presence of 4104 events is **not evidence of malicious activity**. No policy or logging settings were modified.

**Next planned read-only step:** Summarize event IDs in the Operational channel without reading or publishing event messages. A bounded local query can group **only metadata** from up to 1,000 retained events (the previous channel count was 808):

```powershell
Get-WinEvent -LogName 'Microsoft-Windows-PowerShell/Operational' -MaxEvents 1000 |
    Group-Object Id |
    Sort-Object Count -Descending |
    Select-Object Name, Count |
    Format-Table -AutoSize
```

Do not publish full event `Message` fields or script-block text.

## Step 16 — PowerShell Operational event ID distribution (verified; read-only)

**Exact executed command:**

```powershell
Get-WinEvent -LogName 'Microsoft-Windows-PowerShell/Operational' -MaxEvents 1000 |
    Group-Object Id |
    Sort-Object Count -Descending |
    Select-Object Name, Count |
    Format-Table -AutoSize
```

**Actual observed output (ID and aggregate count only):**

```text
Name  Count
----  -----
4104    438
53504    63
12039    63
8196     63
40962    62
40961    62
8197     28
8194      9
8193      9
8195      9
4100      2
```

**Validation:** The eleven event ID groups total **808** records (438 + 63 + 63 + 63 + 62 + 62 + 28 + 9 + 9 + 9 + 2 = 808), matching the previously observed channel `RecordCount` at its earlier snapshot. **4104 accounts for 438/808 ≈ 54.2%** of the returned event records. This consistency supports the inventory, but the circular log may change between queries.

**Security interpretation:** 4104 is associated with PowerShell script-block records; 4100 can be associated with PowerShell command execution errors. An ID count alone cannot establish an incident, full script-block coverage, root cause, or unique script executions. The meanings of **53504, 12039, 8196, 40962, 40961, 8197, 8194, 8193, 8195** have **not** been independently validated in this case and are intentionally recorded without speculative labels. The HKLM `ScriptBlockLogging` policy key was absent in Step 13, but operational 4104 events exist.

**Privacy/limits:** No `Message`, event payload, script text, command line, usernames or private filenames were printed or recorded; no changes made.

**Proposed next read-only check:** Inspect the current-user Windows PowerShell ScriptBlockLogging policy key without dumping unrelated registry values:

```powershell
$path = 'HKCU:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging'
if (Test-Path $path) {
    Get-ItemProperty -Path $path |
        Select-Object EnableScriptBlockLogging |
        Format-List
} else {
    'HKCU policy registry key: Not configured'
}
```

This checks **only** the specified HKCU location, not every possible logging policy or PowerShell edition.

## Classification rules

- **Verified:** A command and its result were actually executed and supplied for this assessment.
- **Prior evidence:** A verified result from a different, linked investigation; **not** a fresh measurement.
- **Proposed:** A recommendation or validation method; **not implemented**.
- **Unknown:** Information has not yet been collected. Unknown does **not** mean failed.
- **Not applicable:** Requires explicit platform/context justification before use.

## Assessment criteria

Assess each control against a practical desired outcome, vendor references, the device's role, local policy precedence and compatibility. Do not automatically label every difference from an enterprise benchmark as a vulnerability. Record residual risks and evidence limits.

## Related investigation

[Windows Security Event Analysis — Real case](../../Windows-Security-Event-Analysis/README.md) documented the monitoring visibility concerns that motivated this new baseline project. That evidence is attributed to its original date and context.
