# Security Baseline — Assessment Register

[Project overview](../README.md) · [Baseline guide](baseline-guide.md) · [Validation plan](validation-plan.md)

**Assessment phase:** Active. **CHG-001 Security log size change implemented and immediate after-state observed; seven-day retention target remains unverified.**

| Control domain | Objective | Directly observed in this project | Provisional next step | Status |
|---|---|---|---|---|
| Firewall | Confirm native protection across network profiles | Initial local query: all three `Enabled=True`, action fields `NotConfigured`; `ActiveStore`: all three `Enabled=True`, default inbound `Block`, outbound `Allow` | If required, examine scoped rule exceptions and active network profile without exposing addresses | **Profile defaults verified; no issue identified in checked fields** |
| Antivirus | Review on-device malware protection status | Service, antivirus, real-time, Tamper Protection, NIS: `True`; signatures age `0`, updated `2026-11-17 09:41:14`; MAPS `2` Advanced, safe sample submission `1`, PUA Block `1`, Block at First Sight not disabled (`False`) | Evaluate current audit policy using read-only commands; consider privacy of cloud reporting | **Verified selected health and advanced-protection indicators; no deficiency identified in fields checked** |
| Security log retention | Ensure enough history for investigations | **CHG-001:** 20 MiB → **128 MiB**, enabled and Circular; post-query **33,989** records; immediate observed span **1.96 days** (pre-change **1.93 days**) | Recheck endpoints after meaningful elapsed time; compare against proposed 7-day target | **CHG-001 configuration and immediate access verified; long-term target pending** |
| Process creation auditing | Improve process activity visibility if justified | **Fresh local `auditpol` check: `Sin auditoría` / No Auditing**; same setting in prior lab | Review a controlled, success-only auditing pilot with retention, privacy, policy management and rollback considerations | **Current gap verified; change not implemented** |
| Identity & access | Review local role exposure in a privacy-safe way | None | Define narrow, redacted query only when needed | **Not assessed** |
| Update posture | Identify missing or stale protection within defined scope | None | Choose supported read-only check | **Not assessed** |
| Platform protections | Review relevant built-in security features | None | Determine OS/build and device support before selecting checks | **Not assessed** |

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
