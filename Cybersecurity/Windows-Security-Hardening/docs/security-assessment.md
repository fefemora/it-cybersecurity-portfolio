# Security Baseline — Assessment Register

[Project overview](../README.md) · [Baseline guide](baseline-guide.md) · [Validation plan](validation-plan.md)

**Assessment phase:** Open. **No hardening configuration changes have been applied.**

| Control domain | Objective | Directly observed in this project | Provisional next step | Status |
|---|---|---|---|---|
| Firewall | Confirm native protection across network profiles | Initial local query: all three `Enabled=True`, action fields `NotConfigured`; `ActiveStore`: all three `Enabled=True`, default inbound `Block`, outbound `Allow` | If required, examine scoped rule exceptions and active network profile without exposing addresses | **Profile defaults verified; no issue identified in checked fields** |
| Antivirus | Review on-device malware protection status | Service, antivirus, real-time, Tamper Protection, NIS: `True`; signatures age `0`, updated `2026-11-17 09:41:14`; MAPS `2` Advanced, safe sample submission `1`, PUA Block `1`, Block at First Sight not disabled (`False`) | Evaluate current audit policy using read-only commands; consider privacy of cloud reporting | **Verified selected health and advanced-protection indicators; no deficiency identified in fields checked** |
| Security log retention | Ensure enough history for investigations | None in this project; **prior lab:** 20 MiB Circular, ~2.03-day observed span | Define retention target and review feasibility | **Prior evidence / proposed review** |
| Process creation auditing | Improve process activity visibility if justified | None in this project; **prior lab:** `No Auditing` | Evaluate volume, privacy, and test approval | **Prior evidence / proposed review** |
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
