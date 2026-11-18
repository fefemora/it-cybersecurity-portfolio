# Change and Validation Plan — CHG-001 Implemented, Monitoring Pending

[Project overview](../README.md) · [Assessment matrix](security-assessment.md) · [Baseline guide](baseline-guide.md)

**State:** **CHG-001 authorized, executed; maximum-size configuration and immediate retained-event access verified**. Immediate time span **1.96 days**; seven-day target not established. Process Creation auditing remains a separate proposal; no rollback.

## Change-control checklist

- [ ] Obtain a complete, read-only baseline for the chosen control and record timestamps.
- [ ] Specify the security objective and why the proposed change is needed.
- [ ] Identify whether the device is standalone, domain-managed or MDM-managed; confirm policy precedence.
- [ ] Assess user impact, remote-access dependencies, privacy and event-volume implications.
- [ ] Establish a safe recovery/rollback procedure suitable for that *specific* change.
- [ ] Approve one narrow change; avoid enabling or disabling unrelated features.
- [ ] Execute only when ready and authorized, then independently capture post-change evidence.
- [ ] Confirm functionality and record any deviations or rollback.

## Possible controlled improvements (not applied)

| Candidate | Why consider it | What would prove success | Risks to manage |
|---|---|---|---|
| Firewall posture review | Ensure intended profiles are protected | Actual enabled state plus connectivity testing appropriate to device role | Disruption of remote services or applications |
| Defender health improvements | Detect unavailable protection or aging signatures | Fresh protection/status output | Third-party AV management and policy precedence |
| Security log retention review — **CHG-001** | Before 20 MiB Circular / **1.93 days**; after 128 MiB enabled/Circular, **33,989 records**, immediate retained span **1.96 days** | Maximum and immediate retained endpoints verified; **7-day target** requires later measurement | Disk consumption, event churn, shorter-than-target retention, policy conflict |
| Process creation auditing pilot | **Fresh query** confirms `No Auditing` | After approved enabling, verify Success audit policy and new benign 4688 records | Higher event volume, faster overwrite if generated volume outpaces the configured Security log capacity, sensitive fields, managed policy conflicts |
| BitLocker system-volume encryption (future separate change) | **Verified C: FullyDecrypted, Protection Off, 0%** | Only after approval and recovery preparedness: validate encryption and protection; never publish recovery secrets | Potential lockout, recovery-key loss, backup/firmware conflicts, encryption impact |

## Proposed controlled change: Process Creation auditing (not approved or executed)

**Verified before-state:** A read-only call to `auditpol /get /subcategory:"{0cce922b-69ae-11d9-bed3-505054503030}"` returned **Creación del proceso — Sin auditoría**. A prior event-analysis project independently reported the same state. This is not evidence of how the policy was configured at the time of older 4688 records.

**Proposed objective:** Enable **Success** auditing for process creation so future, appropriately configured process launches can be recorded as event **4688**. Do not enable Failure auditing or command-line capture as part of the initial proposal.

**Preconditions:**

1. Confirm the workstation is personally controlled or that local changes are authorized, and determine whether AD Group Policy/MDM manages audit settings.
2. Confirm adequate disk space and choose an evidence retention goal. A fresh query found **33,959 records, 20 MiB maximum, Circular**; a separate earlier lab measured ~**2.03 days** between retained endpoints. A fresh time-span check found **1.93 days** between oldest/newest events, down **0.10 days** compared with the earlier independent snapshot; this does not establish overwrite or a trend. Adding process events could reduce retained history.
3. Record the current audit subcategory state and Security log properties, plus a timestamped, privacy-safe change note.
4. Set a planned observation window and rollback criterion (for example, unexpected event volume or disruption to the expected logging baseline).
5. Do **not** publish command lines, executable paths, or user identities from event 4688 without sanitization.

**Candidate command — DOCUMENTED ONLY, NOT RUN:**

```powershell
auditpol /set /subcategory:"{0cce922b-69ae-11d9-bed3-505054503030}" /success:enable
```

This requires an elevated shell and a deliberate approval. It changes only the requested success-auditing option, not general audit-policy categories. Whether it remains effective can depend on managed policy enforcement.

**Proposed verification, only after authorized implementation:**

```powershell
auditpol /get /subcategory:"{0cce922b-69ae-11d9-bed3-505054503030}"
```

Then launch a benign, pre-agreed process and check for relevant **new** 4688 records after the change, using bounded queries and no sensitive process data in public artifacts. Confirm system and application functionality and watch the Security log's growth and oldest-record timestamp. A settings query alone is not proof that the events were generated.

**Rollback plan — DOCUMENTED ONLY, NOT RUN:** If the pre-change state was indeed **No Auditing**, and policy management allows a local reversal, restore Success auditing to disabled:

```powershell
auditpol /set /subcategory:"{0cce922b-69ae-11d9-bed3-505054503030}" /success:disable
```

Check the resulting policy again. This rollback is appropriate **only** if the saved before-state matches this assessment and no other control authority or intervening change requires a different restoration procedure. Do not delete existing events or modify unrelated audit settings.

**Status:** **Proposal only; no change ID or authorized implementation outcome exists.**

## Current capacity check — verified before-state

```powershell
Get-WinEvent -ListLog Security |
    Select-Object LogName, RecordCount, MaximumSizeInBytes, LogMode |
    Format-List
```

**Observed:** Security; **33,959 records**; **20,971,520 bytes (20 MiB) maximum**; **Circular**. Compared with **33,898** records measured in the earlier event-analysis lab, the count differs by **61**, but this is *not* evidence that exactly 61 new events occurred because circular retention and the different collection times prevent that inference.

**Follow-up verified:** Oldest retained event **2026-11-16 00:52:07**, newest **2026-11-17 23:06:29**, elapsed **1.93 days**. Compared with the prior separate **2.03-day** observation, the difference is **0.10 days (~2h 24m)**. These snapshots are not evidence of a specific rollover event. **Next prerequisite completed:** C: free space was verified at **56.81 GiB**. **Subsequently approved and implemented as CHG-001:** 128 MiB maximum was verified after the change; the seven-day target remains pending.

## Security log capacity pilot — CHG-001 implemented and configuration verified

**Evidence-based inputs:**

- **Current maximum:** 20,971,520 bytes = 20 MiB, Circular.
- **Retained endpoint interval:** 1.93 days (different earlier snapshot: 2.03 days).
- **System drive free space:** 56.81 GiB on C: (read-only `Get-PSDrive` observation).
- **Design target:** 7 days retained (proposed, not verified).
- **Initial pilot maximum:** **128 MiB** (134,217,728 bytes; a 108 MiB increase over the current maximum), not a promise of seven-day coverage.

**Execution:** The user confirmed ownership and explicitly authorized changing only the Security log maximum. The command `wevtutil sl Security /ms:134217728` was executed without printed error. A subsequent `Get-WinEvent -ListLog Security` check verified the new configured maximum and Circular mode. Future checks must measure retention span; do not claim that seven days are already retained. The process creation audit setting was not changed in this action.

**Rollback consideration:** The previous maximum was 20 MiB; reducing capacity back to that value could truncate retained history depending on actual log usage. Decide whether restoration is necessary and safe *before* reducing it. Do not automatically run a rollback command.

**Status:** Owner-authorized, implemented and size configuration verified; seven-day retention not yet assessed after elapsed time.

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

## BitLocker encryption — separate proposed change (not authorized)

**Verified initial state:** C: `FullyDecrypted`; BitLocker protection `Off`; `EncryptionPercentage=0` in the actual read-only `Get-BitLockerVolume` result. This is a system-volume data-at-rest confidentiality gap, not evidence of an incident.

**Preconditions:** Confirm Windows/TPM and boot compatibility, reliable backup, recovery-key creation/storage outside the public repo, safe access if recovery is triggered, and any management policy. Preserve the exact baseline and plan appropriate testing. Do not publish a key protector or recovery password.

**TPM read-only check verified:** `TpmPresent=True`, `TpmReady=True`, `TpmEnabled=True`, `TpmActivated=True`. **Secure Boot separately verified:** `Confirm-SecureBootUEFI` returned `False`. TPM readiness is not full encryption readiness. **Firmware check verified:** `Get-ComputerInfo | Select-Object BiosFirmwareType | Format-List` returned `Uefi`. **Disk check verified:** `Get-Partition -DriveLetter C | Get-Disk | Select-Object Number, PartitionStyle, IsBoot, IsSystem | Format-List` returned Disk 0, `GPT`, `IsBoot=True`, `IsSystem=True` (the latter flags are disk-level attributes). **Next prerequisite:** verify independent backups and recovery preparedness.

**Status:** Finding verified; **no BitLocker change authorized or implemented**.

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

## Evidence fields for every *future* approved change

| Field | To be recorded |
|---|---|
| Change ID | **CHG-001** (Security log maximum size) |
| Approval and scope | Owner confirmed personal device and explicitly approved 20 → 128 MiB only |
| Before snapshot | Security, 33,959 records, 20 MiB, Circular |
| Exact command or administrative action | `wevtutil sl Security /ms:134217728` |
| After snapshot | Security, `IsEnabled=True`, 33,989 records, 128 MiB, Circular |
| Operational validation and rollback result | 128 MiB configuration verified; immediate retained span 1.96 days (oldest 2026-11-16 00:52:21); 7-day target pending; rollback not performed |
| Outcome | **CHG-001 implemented; config and immediate access verified; 7-day retention target pending** |

## References & Official Documentation

- [Microsoft Learn — Security Compliance Toolkit](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/security-compliance-toolkit-10)
- [Microsoft Learn — Windows security baselines](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/windows-security-baselines)
- [Microsoft Learn — Audit Process Creation](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-process-creation)
