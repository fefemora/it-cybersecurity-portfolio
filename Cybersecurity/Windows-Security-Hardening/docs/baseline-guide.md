# Lab Guide — Collecting a Read-Only Windows Security Baseline

[Project overview](../README.md) · [Assessment matrix](security-assessment.md) · [Validation plan](validation-plan.md) · [Related event-analysis case](../../Windows-Security-Event-Analysis/README.md)

**Status:** Step 1 executed and verified; further baseline checks pending.  
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

### Step 1b — Inspect active policy store (planned; read-only)

```powershell
Get-NetFirewallProfile -PolicyStore ActiveStore |
    Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction |
    Format-Table -AutoSize
```

**Purpose:** Check the resultant policy store for the same four fields. If `NotConfigured` persists, it remains important not to infer effective traffic behavior from those fields alone; additional supported observations may be needed.

**Evidence status:** Not executed.

## Step 2 — Review Microsoft Defender Antivirus status (planned, read-only)

```powershell
Get-MpComputerStatus |
    Select-Object AMServiceEnabled, AntivirusEnabled, RealTimeProtectionEnabled, AntivirusSignatureAge, AntivirusSignatureLastUpdated
```

**Purpose:** Identify selected antivirus operating and signature status fields without collecting detailed threat detections or personal information.

**Caveat:** Windows may be managed by other endpoint security software or policy. A field does not by itself prove overall protection quality; command availability and value semantics depend on installed components.

**Evidence status:** **Not executed.**

## Step 3 — Review relevant audit configuration (planned, read-only)

```powershell
auditpol /get /category:*
```

**Purpose:** Inspect current advanced audit policy. **Prior observations** in the separate event-analysis lab included Process Creation set to `No Auditing`; this project's current policy state remains **unverified** until a new query.

**Privacy note:** Record only the relevant subcategory names and setting values.

**Evidence status:** **Not executed for this new assessment.**

## Step 4 — Evaluate additional controls (scoped, not executed)

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
