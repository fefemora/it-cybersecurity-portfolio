# Lab Guide — Collecting a Read-Only Windows Security Baseline

[Project overview](../README.md) · [Assessment matrix](security-assessment.md) · [Validation plan](validation-plan.md) · [Related event-analysis case](../../Windows-Security-Event-Analysis/README.md)

**Status:** Steps 1, 1b, 2 and 3 executed and verified; other baseline checks pending.  
**Operating principle:** Minimal impact, read-only, anonymized output. Avoid changing live defenses while investigating.

## Step 1 — Review Windows Firewall profiles (verified)

```powershell
Get-NetFirewallProfile |
    Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction |
    Format-Table -AutoSize
```

**Purpose:** Inspect how the built-in firewall is configured for Domain, Private and Public profiles, including whether each profile is enabled and the default inbound/outbound actions.

**Observed output (actual user-provided PowerShell console):**

```text
Name    Enabled DefaultInboundAction DefaultOutboundAction
----    ------- -------------------- ---------------------
Domain     True        NotConfigured         NotConfigured
Private    True        NotConfigured         NotConfigured
Public     True        NotConfigured         NotConfigured
```

**Interpretation:** All three firewall profiles are enabled (`Enabled=True`). Both default-action fields returned `NotConfigured` for each profile. This **does not establish** that all inbound or outbound connections are allowed, that the firewall is inactive, or what default behavior is effective after applied policy. Do not call these fields `Allow` or `Block` without a separate effective-policy check.

**Limits:** A profile may be enabled without necessarily being the currently active network profile. Individual firewall rules, source-policy precedence, packet behavior, and profile selection were not assessed. No changes were performed.

### Step 1b — Inspect active policy store (verified; read-only)

```powershell
Get-NetFirewallProfile -PolicyStore ActiveStore |
    Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction |
    Format-Table -AutoSize
```

**Purpose:** Examine the resultant applied profile configuration for the same four properties.

**Observed PowerShell output:**

```text
Name    Enabled DefaultInboundAction DefaultOutboundAction
----    ------- -------------------- ---------------------
Domain     True                Block                 Allow
Private    True                Block                 Allow
Public     True                Block                 Allow
```

**Interpretation:** The three profiles remain `Enabled=True`, with **default inbound action `Block` and default outbound action `Allow`** in the active policy store. This resolves the initial `NotConfigured` ambiguity for the effective default actions. It is consistent with a typical workstation firewall posture; no deficiency is established in the profile defaults reviewed.

**Limits:** Specific allow/block rules, the active interface's profile, filtering precedence, and live packet behavior have not been tested. `Block` is a default action subject to policy/rule processing, not proof that every inbound connection is denied. No changes were made.

## Step 2 — Review Microsoft Defender Antivirus status (verified, read-only)

```powershell
Get-MpComputerStatus |
    Select-Object AMServiceEnabled,
                  AntivirusEnabled,
                  RealTimeProtectionEnabled,
                  AntivirusSignatureAge,
                  AntivirusSignatureLastUpdated |
    Format-List
```

**Purpose:** Identify selected antivirus operating and signature status fields without collecting detailed threat detections or personal information.

**Caveat:** Windows may be managed by other endpoint security software or policy. A field does not by itself prove overall protection quality; command availability and value semantics depend on installed components.

**Observed output (actual user-provided console):**

```text
AMServiceEnabled              : True
AntivirusEnabled              : True
RealTimeProtectionEnabled     : True
AntivirusSignatureAge         : 0
AntivirusSignatureLastUpdated : 17-11-2026 9:41:14
```

**Interpretation:** At the time of the command, the Windows antimalware service, antivirus and real-time protection all reported `True`. The signature age was `0` days and the last-update timestamp was **November 17, 2026, 09:41:14** (local console format). No deficiency was identified in these five properties.

**Limitations:** This is a point-in-time status check, not evidence that every protection feature is enabled, exclusions are safe, malware detections are effective, or all endpoint security requirements are satisfied. Cloud-delivered protection and tamper protection were not queried. The command did not perform a scan, update signatures, or alter settings.

**Evidence status:** **Verified; no changes performed.**

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

## Step 4 — Review relevant audit configuration (planned, read-only)

```powershell
auditpol /get /category:*
```

**Purpose:** Inspect current advanced audit policy. **Prior observations** in the separate event-analysis lab included Process Creation set to `No Auditing`; this project's current policy state remains **unverified** until a new query.

**Privacy note:** Record only the relevant subcategory names and setting values.

**Evidence status:** **Not executed for this new assessment.**

## Step 5 — Evaluate additional controls (scoped, not executed)

Only after reviewing the first results, choose targeted read-only checks for local privilege exposure, update posture and device safeguards. Avoid dumping users, software inventory, registry secrets, local administrator memberships, firewall rule details, or recovery keys into a public repository.

## How findings are documented

For every check, record:
1. The **exact** command actually run.
2. The security question it answers.
3. The **real, sanitized** console output and collection context where available.
4. The supported interpretation, uncertainty and operational relevance.
5. Whether a control change is merely **proposed**, **approved**, **implemented** or **validated**.

## References & Official Documentation

- [Microsoft Learn — Get-NetFirewallProfile](https://learn.microsoft.com/en-us/powershell/module/netsecurity/get-netfirewallprofile)
- [Microsoft Learn — Get-MpComputerStatus](https://learn.microsoft.com/en-us/powershell/module/defender/get-mpcomputerstatus)
- [Microsoft Learn — auditpol /get](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/auditpol-get)
- [Microsoft Learn — Security baselines guide](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/windows-security-baselines)
