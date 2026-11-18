# Windows Security Hardening — Baseline, Improvement Plan & Validation

**Area:** Cybersecurity / Blue Team · Endpoint security  
**Evidence status:** **CHG-001 implemented and verified:** Security log maximum 20 → 128 MiB; immediate retained span 1.96 days. **BitLocker C: fully decrypted, protection Off; TPM ready; Secure Boot disabled (`False`).** Encryption and process-audit changes are proposals only; seven-day retention remains unverified.  
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
| Update and platform security | BitLocker C: 0% / Off; TPM present/ready/enabled/activated = `True`; **Secure Boot = `False`** | **Two distinct gaps verified (disk encryption and Secure Boot); changes not implemented** |
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

**Next:** Gather non-sensitive boot-mode evidence; do not change firmware or enable encryption without separate readiness and approval.

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
