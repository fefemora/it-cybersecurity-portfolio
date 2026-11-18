# Windows Security Hardening — Baseline, Improvement Plan & Validation

**Area:** Cybersecurity / Blue Team · Endpoint security  
**Project status:** **Read-only assessment completed through Step 28; Steps 29–30 documented closure and recommendations.**  
**Evidence status:** **233 event 4672 records in a 24-hour query (Step 28).** **Logon audit policy Success and Failure verified (Step 25); 235 event 4624 records, no 4625 matches (Step 26), and a 4624 distribution of 229 service / 4 cached-interactive / 2 unlock logons (Step 27).** **Windows PowerShell Transcription HKLM/HKCU policy keys absent at inspected paths (Steps 23–24); three PowerShell logging policy mechanisms summarized.** **Windows PowerShell ModuleLogging HKLM/HKCU keys absent at inspected paths; 4103 event count returned zero (Step 22).** **CHG-001 implemented and verified:** Security log maximum 20 → 128 MiB; immediate retained span 1.96 days. **BitLocker C: fully decrypted, protection Off; TPM ready; UEFI boot mode and GPT system disk verified; Secure Boot disabled (`False`).** Encryption and process-audit changes are proposals only; seven-day retention remains unverified.  
**Environment:** Windows workstation, Windows PowerShell, built-in security controls. No new VM or commercial tools required.

[Portfolio home](../../README.md) · [Baseline guide](docs/baseline-guide.md) · [Assessment matrix](docs/security-assessment.md) · [Validation plan](docs/validation-plan.md) · [Related real event-analysis case](../Windows-Security-Event-Analysis/README.md)

## Why this project?

A prior **real Windows Security Event Analysis** investigation documented a locally enabled Security log with a 20 MiB circular maximum and an observed retained time span of approximately **2.03 days**. It also found the **Process Creation** audit subcategory set to **No Auditing** at inspection time. These findings motivate a broader security baseline review; they do **not** establish that other settings are insecure or that a security incident occurred.

This project separates **observation → assessment → proposed change → authorized implementation → post-change validation**. No changes will be applied without a deliberate review of compatibility, recovery steps and evidence.

![Planned workflow: baseline, assessment, change, validation](diagrams/hardening-workflow.svg)

## Scope

| Workstream | Initial read-only observation | Current evidence state |
|---|---|---|
| Windows Defender Antivirus | Service, antivirus, real-time, tamper and NIS protection reported `True`; signatures age `0`; MAPS `2`, sample consent `1`; PUA Block `1`, Block at First Sight not disabled (`False`) | **Selected protection statuses/preferences verified; operational effectiveness not tested** |
| Windows Firewall | Local query: three profiles enabled, action fields `NotConfigured`; **ActiveStore: enabled, inbound `Block`, outbound `Allow` for all three** | **Verified profile default actions; rule-level traffic behavior not tested** |
| Security auditing | Process Creation **No Auditing** in the last read-only check; Security log **20 MiB → 128 MiB** under **CHG-001**, Circular; after-change **33,989** records | **Size change implemented/verified; 7-day retention not yet established** |
| Local accounts and permissions | Limited, anonymized policy/role review | **Not collected** |
| Update and platform security | BitLocker C: 0% / Off; TPM flags `True`; firmware **UEFI**; system disk **GPT**, disk-level `IsBoot`/`IsSystem=True`; Secure Boot `False` | **UEFI/GPT verified; Secure Boot and encryption remain off; no changes** |
| Recommendations and validation | CHG-001 Security log maximum-size adjustment; separate future process-auditing pilot | **CHG-001 configuration verified; follow-up retention pending** |

## Working sequence

1. **Read-only baseline:** Collect minimal, sanitized evidence for one control family at a time.
2. **Assess:** Compare current settings to the intended security outcome, while considering standalone versus managed endpoints.
3. **Plan:** Describe the candidate improvement, prerequisites, operational impact, and rollback.
4. **Implement only if authorized:** Save a pre-change baseline and apply narrowly scoped modifications. These are **future actions, not executed work**.
5. **Validate:** Check configuration, expected behavior and unexpected side effects; mark verified only with real results.
6. **Report:** Produce a concise hardening case study with before/after evidence if implementation occurs.

## Step 1 — Verified Windows Firewall profile baseline

Run this **read-only** command on the Windows workstation in PowerShell:

```powershell
Get-NetFirewallProfile |
    Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction |
    Format-Table -AutoSize
```

**Actual observed output:**

```text
Name    Enabled DefaultInboundAction DefaultOutboundAction
----    ------- -------------------- ---------------------
Domain     True        NotConfigured         NotConfigured
Private    True        NotConfigured         NotConfigured
Public     True        NotConfigured         NotConfigured
```

**Assessment:** All three profiles are enabled. The initial `NotConfigured` output did not establish traffic behavior, so the effective ActiveStore was queried separately. **No configuration changes performed.**

**Step 1b — Verified ActiveStore query:**

```powershell
Get-NetFirewallProfile -PolicyStore ActiveStore |
    Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction |
    Format-Table -AutoSize
```

**Observed output:**

```text
Name    Enabled DefaultInboundAction DefaultOutboundAction
----    ------- -------------------- ---------------------
Domain     True                Block                 Allow
Private    True                Block                 Allow
Public     True                Block                 Allow
```

**Finding:** All three profile defaults are **inbound Block / outbound Allow** in ActiveStore. Specific allow rules, active network profile, and actual packet behavior were **not** examined. **Next:** read-only Microsoft Defender Antivirus health check (completed, shown below).

## Step 2 — Verified Microsoft Defender Antivirus health

```powershell
Get-MpComputerStatus |
    Select-Object AMServiceEnabled,
                  AntivirusEnabled,
                  RealTimeProtectionEnabled,
                  AntivirusSignatureAge,
                  AntivirusSignatureLastUpdated |
    Format-List
```

**Observed console output (real):**

```text
AMServiceEnabled              : True
AntivirusEnabled              : True
RealTimeProtectionEnabled     : True
AntivirusSignatureAge         : 0
AntivirusSignatureLastUpdated : 17-11-2026 9:41:14
```

**Finding:** Microsoft Defender antimalware service, antivirus and real-time protection were enabled at inspection time. Reported antivirus signature age was **0 days**, with a last-update timestamp of **2026-11-17 09:41:14** in the console's local time. These five fields show no identified issue in this bounded check; they do **not** establish cloud-delivered protection, tamper protection, exclusions, detection efficacy or enterprise compliance.

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

**Initial sizing pilot subsequently implemented and verified as CHG-001:** The **128 MiB Security log maximum** (134,217,728 bytes) replaced the previous **20 MiB** limit. The increase to the maximum is **108 MiB**. A **7-day retention window** is a *proposed target*, **not a predicted or guaranteed result**, especially if Process Creation auditing is enabled later and event volume rises.

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

## Step 17 — Current-user PowerShell Script Block Logging policy key (verified; read-only)

**Exact executed command:**

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

**Actual output:**

```text
HKCU policy registry key: Not configured
```

**Interpretation:** No `ScriptBlockLogging` policy key was present under the **current user's** inspected HKCU path. Along with the earlier absent inspected **HKLM** policy key, this supports **no explicit setting in either of these two Windows PowerShell policy paths at inspection time**. This does **not** prove the absence of all effective policies, logging behaviors, or alternative PowerShell edition settings. In particular, **438 retained event ID 4104 records** exist in the Operational log despite the two absent keys; no assumption about their origin or detection coverage is warranted.

**Security and privacy:** The check read one registry-key existence and did not edit policies, extract scripts, or query secrets. No configuration changes were performed. This observation is not a confirmed exploit or security incident.

**Proposed next step (read-only, metadata only):** Group the retained event 4104 records by their event severity level without outputting script text:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-PowerShell/Operational'
    Id = 4104
} |
    Group-Object LevelDisplayName |
    Select-Object Name, Count |
    Format-Table -AutoSize
```

An event's severity alone is not evidence of malicious execution. The script contents and full messages must remain private.

## Step 18 — PowerShell event 4104 severity distribution (verified; read-only)

**Exact executed command:**

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-PowerShell/Operational'
    Id = 4104
} |
    Group-Object LevelDisplayName |
    Select-Object Name, Count |
    Format-Table -AutoSize
```

**Actual console output (localized):**

```text
Name        Count
----        -----
Advertencia   439
```

**Assessment:** At this later snapshot, the query returned **439 retained event ID 4104 records** and classified all of them as **Warning** (`Advertencia`) by their displayed severity. The preceding count-only check returned **438**, so **the snapshots differ by one record**; ongoing logging between queries is a plausible explanation, **not independently confirmed**. A warning-level 4104 is not by itself evidence of malicious PowerShell activity, a unique executed script, or a confirmed incident. Missing HKLM and HKCU policy keys at their inspected locations likewise do not establish the effective source of these events or comprehensive script-block logging coverage.

**Privacy/change control:** No script text, event messages, usernames, paths, or commands were extracted. No policy or firmware settings changed. Secure Boot and BitLocker work remains deferred pending backup and recovery planning.

**Next proposed check (read-only; timestamps only):** Compare the oldest and newest retained PowerShell Operational events to assess the channel's currently observed time span, without displaying event messages:

```powershell
$log = 'Microsoft-Windows-PowerShell/Operational'
$oldest = Get-WinEvent -LogName $log -Oldest -MaxEvents 1
$newest = Get-WinEvent -LogName $log -MaxEvents 1
[PSCustomObject]@{
    OldestEvent = $oldest.TimeCreated
    NewestEvent = $newest.TimeCreated
    RetainedDays = [math]::Round(($newest.TimeCreated - $oldest.TimeCreated).TotalDays, 2)
} | Format-List
```

Channel-wide retention does not establish how long individual event IDs persist.

## Step 19 — PowerShell Operational retained time span (verified; read-only)

**Executed read-only command:**

```powershell
$log = 'Microsoft-Windows-PowerShell/Operational'
$oldest = Get-WinEvent -LogName $log -Oldest -MaxEvents 1
$newest = Get-WinEvent -LogName $log -MaxEvents 1

[PSCustomObject]@{
    OldestEvent = $oldest.TimeCreated
    NewestEvent = $newest.TimeCreated
    RetainedDays = [math]::Round(($newest.TimeCreated - $oldest.TimeCreated).TotalDays, 2)
} | Format-List
```

**Actual console output (local time; decimal comma as displayed):**

```text
OldestEvent  : 13-11-2026 1:11:52
NewestEvent  : 18-11-2026 2:20:33
RetainedDays : 5,05
```

**Finding:** At the time of this snapshot, the earliest and latest retained `Microsoft-Windows-PowerShell/Operational` events span **5.05 days**, from **2026-11-13 01:11:52** to **2026-11-18 02:20:33**, local time. The earlier configuration check found a **15 MiB Circular** log. This is an **observed event-history span**, **not a configured minimum retention time or SLA** and **not a proven minimum history for ID 4104**. A log's oldest event may reflect when recording began, inactivity, or overwritten records; the cause is not established by these timestamps.

**Limitations:** No event messages, scripts, private identifiers or contents were accessed; no settings were changed. A separate Security log was increased from 20 to 128 MiB under CHG-001, but its seven-day objective remains independently unverified. Secure Boot and BitLocker modifications remain postponed until backup and recovery readiness.

**Next proposed read-only check:** Verify a narrow overview of PowerShell module logging policy at the **HKLM** path only, without enumerating module names or event payloads:

```powershell
$path = 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging'
if (Test-Path $path) {
    Get-ItemProperty -Path $path |
        Select-Object EnableModuleLogging |
        Format-List
} else {
    'HKLM ModuleLogging policy key: Not configured'
}
```

This checks one Windows PowerShell policy path; it cannot alone establish effective logging across all scopes or PowerShell editions.

## Step 20 — Windows PowerShell Module Logging machine policy (verified; read-only)

**Exact executed command:**

```powershell
$path = 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging'

if (Test-Path $path) {
    Get-ItemProperty -Path $path |
        Select-Object EnableModuleLogging |
        Format-List
} else {
    'HKLM ModuleLogging policy key: Not configured'
}
```

**Actual observed output:**

```text
HKLM ModuleLogging policy key: Not configured
```

**Interpretation:** The **Windows PowerShell ModuleLogging machine policy key** was absent in the specific inspected HKLM location. This is evidence of no explicit machine-policy setting at this path, **not proof that all PowerShell module logging is inactive**. Other scopes, edition-specific settings, and actual event 4103 availability have not been evaluated in this step. The previously observed event 4104 counts relate to Script Block Logging and **do not validate Module Logging**.

**Privacy and change-control:** No policy changes, log payloads, module names, command arguments or secrets were accessed. Secure Boot and BitLocker work remains postponed pending backups and recovery planning.

**Next proposed read-only check:** Inspect the current-user counterpart of Module Logging policy, reporting only the flag or whether the exact key is absent:

```powershell
$path = 'HKCU:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging'
if (Test-Path $path) {
    Get-ItemProperty -Path $path |
        Select-Object EnableModuleLogging |
        Format-List
} else {
    'HKCU ModuleLogging policy key: Not configured'
}
```

This check does not determine the full effective configuration by itself.

## Step 21 — Windows PowerShell Module Logging current-user policy (verified; read-only)

**Exact executed command:**

```powershell
$path = 'HKCU:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging'

if (Test-Path $path) {
    Get-ItemProperty -Path $path |
        Select-Object EnableModuleLogging |
        Format-List
} else {
    'HKCU ModuleLogging policy key: Not configured'
}
```

**Actual observed output:**

```text
HKCU ModuleLogging policy key: Not configured
```

**Interpretation:** The ModuleLogging registry key was absent at the inspected **current-user HKCU** path. Step 20 independently verified absence of the corresponding inspected **HKLM** path. Neither result confirms that all effective module logging is disabled; alternative PowerShell editions, policy stores and runtime configuration were not fully assessed. Earlier **4104 events** relate to script-block reporting and are not proof of **4103** Module Logging records.

**Privacy and change status:** Only policy-key presence was queried. No module names, event messages, command parameters, user data, registry values outside the narrow flag, or scripts were published. Nothing was modified. Secure Boot and BitLocker remain deferred until backup and recovery readiness.

**Next read-only check proposed:** Count retained PowerShell **4103** events without reading event content. Use an event log filter and only show the count:

```powershell
$count = (Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-PowerShell/Operational'
    Id = 4103
} -ErrorAction SilentlyContinue | Measure-Object).Count

[PSCustomObject]@{ EventId = 4103; Count = $count } | Format-List
```

A zero result would only show that no matching records were retrieved from the currently retained channel; with `-ErrorAction SilentlyContinue`, query errors must be considered separately.

## Step 22 — Retained PowerShell Module Logging event 4103 count (verified; read-only)

**Exact executed command:**

```powershell
$count = (Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-PowerShell/Operational'
    Id = 4103
} -ErrorAction SilentlyContinue | Measure-Object).Count

[PSCustomObject]@{
    EventId = 4103
    Count = $count
} | Format-List
```

**Actual output:**

```text
EventId : 4103
Count   : 0
```

**Interpretation:** This query retrieved **zero retained event ID 4103 records** from `Microsoft-Windows-PowerShell/Operational`. The previous read-only ID distribution (11 different IDs, 808 total records at its snapshot) also did not list 4103. The HKLM and HKCU `ModuleLogging` policy registry keys were independently absent at their inspected paths in Steps 20–21. Together these results indicate **no 4103 visibility evidenced in the examined retained channel**, but do **not** establish that module logging was never enabled, that all policy scopes were assessed, or that a security incident occurred.

**Methodological limitation:** `-ErrorAction SilentlyContinue` suppresses non-terminating query errors, so `Count=0` alone cannot distinguish no matches from certain query problems. A follow-up may perform an error-visible metadata-only query if verification is required. Log rotation may also discard historical records.

**Privacy/change control:** No event messages, command lines, modules, scripts, paths or credentials were displayed; no settings modified. Secure Boot and BitLocker remain paused pending backup/recovery readiness.

**Next read-only assessment:** Inspect the existence and enabled flag of the machine-level Windows PowerShell **Transcription** policy only, without accessing transcript output or paths:

```powershell
$path = 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription'
if (Test-Path $path) {
    Get-ItemProperty -Path $path |
        Select-Object EnableTranscripting |
        Format-List
} else {
    'HKLM Transcription policy key: Not configured'
}
```

## Step 23 — Windows PowerShell Transcription machine policy (verified; read-only)

**Exact executed command:**

```powershell
$path = 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription'

if (Test-Path $path) {
    Get-ItemProperty -Path $path |
        Select-Object EnableTranscripting |
        Format-List
} else {
    'HKLM Transcription policy key: Not configured'
}
```

**Actual observed output:**

```text
HKLM Transcription policy key: Not configured
```

**Interpretation:** The Windows PowerShell `Transcription` machine-policy key was absent at the inspected HKLM path. This establishes **no explicit machine Transcription policy at this location at the time of inspection**, not that transcription has been disabled in every scope or that no transcript files exist. Other policy stores, current-user configuration, editions, and session-level transcription were not examined.

**Privacy and change control:** The query checked only a specific registry-key location and flag; no transcript content, output destination, session text or personal data was retrieved. No policies or endpoint settings were changed. Secure Boot and BitLocker remain deferred pending backups and recovery readiness.

**Next proposed read-only check:** Inspect the current-user `Transcription` policy path without displaying transcript output settings or transcript files:

```powershell
$path = 'HKCU:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription'
if (Test-Path $path) {
    Get-ItemProperty -Path $path |
        Select-Object EnableTranscripting |
        Format-List
} else {
    'HKCU Transcription policy key: Not configured'
}
```

## Step 24 — Windows PowerShell Transcription current-user policy (verified; read-only)

**Exact executed command:**

```powershell
$path = 'HKCU:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription'

if (Test-Path $path) {
    Get-ItemProperty -Path $path |
        Select-Object EnableTranscripting |
        Format-List
} else {
    'HKCU Transcription policy key: Not configured'
}
```

**Actual observed output:**

```text
HKCU Transcription policy key: Not configured
```

**Interpretation:** No `Transcription` policy key was found at the **current-user HKCU** path. Along with Step 23, **both examined Windows PowerShell Transcription policy paths (HKLM and HKCU) lacked an explicit key**. This does not rule out session-level transcripts, other policy stores, PowerShell editions or effective settings not tested. No transcript files or contents were inspected.

### PowerShell logging assessment summary (read-only)

| Feature | HKLM key at queried path | HKCU key at queried path | Observed telemetry / limits |
|---|---|---|---|
| Script Block Logging | Absent | Absent | 4104 events observed; 438 in one query, 439 Warning events in a later query; logging scope undetermined |
| Module Logging | Absent | Absent | No 4103 events retrieved by the count query (errors suppressed); effective logging unverified |
| Transcription | Absent | Absent | No transcript-file check performed; effective transcription unverified |
| Operational event channel | Not a policy-key check | Not a policy-key check | Enabled; 808 records and 15 MiB Circular at earlier snapshot; 5.05-day observed history span |

**Assessment:** A missing explicit registry policy at the inspected paths is **not equivalent to no telemetry or a proven vulnerability**. The existence of 4104 events demonstrates some retained script-block-related records, but not comprehensive recording or a security incident. This phase produced **verified configuration and event metadata only**. No logging, TPM, Secure Boot, BitLocker, firewall or audit policy settings were modified.

**Planned follow-up:** Draft a privacy-aware monitoring recommendations summary and prioritize any separate changes only after explicit owner approval. Secure Boot and BitLocker stay deferred until backup/recovery preparations.

## Step 25 — Windows Logon audit policy (verified; read-only)

**Exact command executed:**

```powershell
auditpol /get /subcategory:"{0cce9215-69ae-11d9-bed3-505054503030}"
```

**Actual observed localized output (policy rows):**

```text
Directiva de auditoría del sistema
Categoría o subcategoría                  Configuración
Inicio/cierre de sesión
  Inicio de sesión                        Aciertos y errores
```

**Interpretation:** The Windows Advanced Audit Policy **Logon** subcategory is configured to audit **Success and Failure** (`Aciertos y errores`) on this machine. This supports collection of relevant sign-in activity such as Security event IDs **4624 (successful logon)** and **4625 (failed logon)** when the relevant events are generated and retained. This does not verify actual counts, the completeness of all authentication flows, forwarding, or any suspicious activity.

**Retention and limitations:** The Security event log maximum was previously raised from 20 MiB to 128 MiB under the **only approved change, CHG-001**; long-term seven-day history has not yet been verified. Querying `auditpol` did not change any policies or expose user names, IP addresses or credentials.

**Next proposed read-only check (counts only):** Quantify recent 4624/4625 records in the past 24 hours with an error-visible and privacy-safe query, without printing event messages or user details:

```powershell
try {
    $events = Get-WinEvent -FilterHashtable @{
        LogName = 'Security'
        Id = 4624,4625
        StartTime = (Get-Date).AddHours(-24)
    } -ErrorAction Stop
    $events | Group-Object Id |
        Select-Object Name, Count |
        Format-Table -AutoSize
} catch {
    Write-Output ('Query status: ' + $_.Exception.Message)
}
```

A missing event ID from grouped results is not proof that the audit subcategory is disabled or that no such event has ever occurred. Only aggregate counts should be shared.

## Step 26 — Retained logon events over the preceding 24 hours (verified; read-only)

**Exact query executed:**

```powershell
try {
    $events = Get-WinEvent -FilterHashtable @{
        LogName = 'Security'
        Id = 4624,4625
        StartTime = (Get-Date).AddHours(-24)
    } -ErrorAction Stop

    $events | Group-Object Id |
        Select-Object Name, Count |
        Format-Table -AutoSize
} catch {
    Write-Output ('Query status: ' + $_.Exception.Message)
}
```

**Actual user-provided output:**

```text
Name Count
---- -----
4624   235
```

**Interpretation:** The query returned **235 event ID 4624 records** (successful logon) and **no 4625 group** (failed logon) in the queried **preceding 24-hour window**, from retained Security-log data. Since `Get-WinEvent` returned records and the catch branch did not appear, this is consistent with **zero 4625 matches in that combined query**, not proof that failures never occurred or that every possible failure is logged. **235 event records do not imply 235 distinct people or interactive sessions**: service, batch and system logons can also generate 4624. No individual accounts or logon sources were inspected.

**Audit and retention context:** Step 25 verified the Logon audit subcategory as `Aciertos y errores` (Success and Failure). Security log maximum was previously raised to **128 MiB under the sole approved CHG-001**, but the seven-day history objective is not yet validated. This query and all current monitoring steps were **read-only**.

**Next proposed check (read-only; aggregate only):** Group successful 4624 events from the last 24 hours by the `LogonType` field, so interactive, service, network and other login types are not confused with unique human users. Do not share usernames, IP addresses or raw event XML.

```powershell
$from = (Get-Date).AddHours(-24)
$types = Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4624
    StartTime = $from
} -ErrorAction Stop | ForEach-Object {
    $xml = [xml]$_.ToXml()
    ($xml.Event.EventData.Data | Where-Object Name -eq 'LogonType').'#text'
}
$types | Group-Object | Sort-Object Count -Descending |
    Select-Object Name, Count | Format-Table -AutoSize
```

The aggregate type distributions are sampled later and may not sum to 235 because the query time window advances.

## Step 27 — Successful logon type distribution (verified; read-only)

**Exact command executed:**

```powershell
$from = (Get-Date).AddHours(-24)
$types = Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4624
    StartTime = $from
} -ErrorAction Stop | ForEach-Object {
    $xml = [xml]$_.ToXml()
    ($xml.Event.EventData.Data | Where-Object Name -eq 'LogonType').'#text'
}
$types | Group-Object | Sort-Object Count -Descending |
    Select-Object Name, Count | Format-Table -AutoSize
```

**Actual observed aggregate output:**

```text
Name Count
---- -----
5      229
11       4
7        2
```

**Finding:** The query returned **235 retained Security event 4624 records**, grouped as **LogonType 5 (Service): 229**, **LogonType 11 (CachedInteractive): 4**, and **LogonType 7 (Unlock): 2**. Totals reconcile exactly to the prior separate 24-hour 4624 count (235), although the two 24-hour windows were queried at different times and may evolve.

**Interpretation:** **97.4%** of these records were **service-type logons**, **1.7%** cached interactive, and **0.9%** unlock events. These are **event records**, not unique users or proof of legitimacy. The distribution does not independently establish what accounts, services or circumstances generated the records; no anomalous activity was confirmed based on the aggregate metadata. Prior audit-policy query returned Logon: Success and Failure; the earlier 24-hour query returned no 4625 group.

**Privacy/change control:** Source event XML was parsed **locally** only to extract the LogonType field. No usernames, IP addresses, session IDs or full records were transmitted or committed. No policy, logging, BitLocker or firmware changes were made.

**Next proposed read-only assessment:** Inspect only the count of Security event **4672** (special privileges assigned to a new logon) over the preceding 24 hours, to compare privilege-related event volume without disclosing account names. This is a different event type and must not be equated with a confirmed administrator login or suspicious activity.

```powershell
try {
    $count = (Get-WinEvent -FilterHashtable @{
        LogName = 'Security'
        Id = 4672
        StartTime = (Get-Date).AddHours(-24)
    } -ErrorAction Stop | Measure-Object).Count
    [PSCustomObject]@{ EventId = 4672; Count = $count } | Format-List
} catch {
    'Query status: ' + $_.Exception.Message
}
```

## Step 28 — Special privileges assigned to new logons (verified; read-only)

**Exact command:**
```powershell
try {
    $count = (Get-WinEvent -FilterHashtable @{
        LogName = 'Security'
        Id = 4672
        StartTime = (Get-Date).AddHours(-24)
    } -ErrorAction Stop | Measure-Object).Count
    [PSCustomObject]@{ EventId = 4672; Count = $count } | Format-List
} catch {
    'Query status: ' + $_.Exception.Message
}
```

**Actual user-provided output:**
```text
EventId : 4672
Count   : 233
```

**Interpretation:** **233 event ID 4672 records** were retrieved from the preceding 24 hours. Event 4672 indicates special privileges were assigned to a new logon; it is common in routine system/service activity and **does not prove compromise, malicious elevation, 233 privileged people, or a direct correspondence to the previously observed 235 event 4624 records**. Those independent queries used moving 24-hour windows. Correlating particular sessions would require more sensitive event fields and was outside this privacy-preserving scope. No changes were made.

## Step 29 — Consolidated assessment and risk priorities (analysis; no new command)

| Priority | Finding supported by evidence | Assessment | Recommended action (not implemented) |
|---|---|---|---|
| High attention / recovery prerequisite | System volume C: decrypted with BitLocker Off; Secure Boot query returned False; TPM ready and UEFI/GPT observed | Potential device-loss and boot-chain exposure; actual threat and business risk depend on context | Prepare a verified backup and recovery media, protect recovery keys, check firmware compatibility and then separately authorize BitLocker/Secure Boot changes |
| Medium / monitoring assurance | Security log grew from 20 to 128 MiB under approved CHG-001; immediate history **1.96 days**, seven-day goal not validated | Storage setting validated, monitoring time objective **pending** | Recheck oldest/newest records after sufficient elapsed time; tune event volumes and capacity based on measurements |
| Medium / detection coverage | Process Creation audit subcategory observed as No Auditing; no change approved | Possible gap for process-execution investigations; business need and privacy impact must be assessed | Consider scoped process creation auditing and command-line capture only under separately approved change management |
| Review / telemetry tuning | Script Block Logging policy keys absent at inspected HKLM/HKCU paths despite 4104 records; Module Logging keys absent with no retrieved 4103; Transcription keys absent | Effective coverage unclear; **not proof of missing all logging, a vulnerability, or an incident** | Determine effective policy sources, privacy limits, record retention and target use cases before enabling additional logging |
| Monitor / no confirmed anomaly | Logon audit Success and Failure; 24h 4624 **235**, no matching 4625; 4624 types 5=229, 11=4, 7=2; 4672 **233** | Predominantly service logons and notable privilege event volume, but aggregate counts alone do not confirm benignness or malice | Baseline trends; investigate only with appropriate consent and controlled correlation when justified |
| Existing protection | Firewall ActiveStore inbound Block/outbound Allow, checked Defender protection indicators enabled | Positive baseline observations limited to fields and snapshot tested | Periodically reconfirm health; avoid unnecessary changes |

**Risk levels above are planning priorities, not measured CVSS scores or a certification/compliance attestation.** No live exploitation, attack simulation, malware claim or identity attribution was performed.

## Step 30 — Project closure and validation summary

**Project phase:** **Completed — read-only baseline and one narrowly authorized change**, with deferred hardening activities documented. 28 command-led checks were completed, followed by this consolidation (Step 29) and publication/closure (Step 30). The lab is **not** a claim of full system hardening, security certification or incident clearance.

**Verified change:** **CHG-001 only** — Security event log maximum increased from **20 MiB to 128 MiB**; resulting configuration and immediate event access confirmed. The proposed seven-day retained-history outcome is **not yet validated**. No Process Creation auditing, Module Logging, Transcription, Script Block Logging, Defender, firewall, BitLocker or Secure Boot changes are claimed.

**Validated capabilities demonstrated:** PowerShell-based, privacy-preserving endpoint baseline collection; audit-policy and event-channel interpretation; classification of Event IDs 4103/4104/4624/4625/4672; aggregation of logon types; measurement versus proof of retention; prioritization of pending controls; and evidence-based change management.

**Deferred activities / exit notes:**
1. Ensure backups, recovery media, BitLocker recovery-key custody, and firmware readiness **before** considering encryption or Secure Boot changes.
2. Recheck Security log oldest/newest event timestamps after adequate elapsed time; distinguish observed span from guaranteed retention.
3. Propose and separately approve any process creation or PowerShell logging changes only after privacy, performance, rollback and policy-precedence reviews.
4. If investigating privilege/logon behavior later, use access-controlled correlation and do not publish raw account identifiers or logs to this repository.

**Evidence handling:** All reported outputs came from user-run commands; only sanitized numeric aggregates, configuration flags, timestamps and command text are published. The final 4672 observation is a count of **233** in a moving 24-hour window, not a finding of malicious privilege elevation.

**Final disposition:** **Assessment documented; no incident confirmed; one change validated for configuration only; remaining recommendations deferred or proposed.**



## Evidence integrity

- **Real prior findings** are linked to their source project; they are not relabeled as newly collected hardening results.
- **Unknown** means not yet tested, not insecure.
- A recommended baseline is a reference for evaluation, not an assertion that every setting should be enabled on every workstation.
- No attack simulations, policy edits, service restarts, exclusions, or firewall-rule changes have yet been performed.

## References & Official Documentation

- [Microsoft Learn — Windows security baselines](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/windows-security-baselines)
- [Microsoft Learn — Security Compliance Toolkit](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/security-compliance-toolkit-10)
- [Microsoft Learn — Get-NetFirewallProfile](https://learn.microsoft.com/en-us/powershell/module/netsecurity/get-netfirewallprofile)
- [Microsoft Learn — Get-MpComputerStatus](https://learn.microsoft.com/en-us/powershell/module/defender/get-mpcomputerstatus)

References are technical guidance; no vendor certification or compliance attestation is implied.
