# Lab Guide — Collecting a Read-Only Windows Security Baseline

[Project overview](../README.md) · [Assessment matrix](security-assessment.md) · [Validation plan](validation-plan.md) · [Related event-analysis case](../../Windows-Security-Event-Analysis/README.md)

**Status:** Procedure prepared; **no commands have been verified on this workstation for this project**.  
**Operating principle:** Minimal impact, read-only, anonymized output. Avoid changing live defenses while investigating.

## Step 1 — Review Windows Firewall profiles (ready to execute)

```powershell
Get-NetFirewallProfile |
    Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction |
    Format-Table -AutoSize
```

**Purpose:** Inspect how the built-in firewall is configured for Domain, Private and Public profiles, including whether each profile is enabled and the default inbound/outbound actions.

**Expected output shape (illustrative only; not a measured result):**

```text
Name     Enabled DefaultInboundAction DefaultOutboundAction
----     ------- -------------------- ---------------------
Domain   <value> <value>              <value>
Private  <value> <value>              <value>
Public   <value> <value>              <value>
```

**Interpretation guide:** A disabled profile could reduce protection when that profile is active. `NotConfigured`/unspecified actions should not be silently assumed to be explicitly set to Allow or Block; assess effective configuration in context. These settings **do not** enumerate every rule or establish that all traffic is blocked or permitted.

**Evidence status:** **Pending user console output.** Do not record fabricated setting values.

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
