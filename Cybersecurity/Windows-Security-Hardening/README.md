# Windows Security Hardening — Baseline, Improvement Plan & Validation

**Area:** Cybersecurity / Blue Team · Endpoint security  
**Evidence status:** **Read-only firewall profile baseline verified.** Further effective-policy checks are pending; no configuration changes were made.  
**Environment:** Windows workstation, Windows PowerShell, built-in security controls. No new VM or commercial tools required.

[Portfolio home](../../README.md) · [Baseline guide](docs/baseline-guide.md) · [Assessment matrix](docs/security-assessment.md) · [Validation plan](docs/validation-plan.md) · [Related real event-analysis case](../Windows-Security-Event-Analysis/README.md)

## Why this project?

A prior **real Windows Security Event Analysis** investigation documented a locally enabled Security log with a 20 MiB circular maximum and an observed retained time span of approximately **2.03 days**. It also found the **Process Creation** audit subcategory set to **No Auditing** at inspection time. These findings motivate a broader security baseline review; they do **not** establish that other settings are insecure or that a security incident occurred.

This project separates **observation → assessment → proposed change → authorized implementation → post-change validation**. No changes will be applied without a deliberate review of compatibility, recovery steps and evidence.

![Planned workflow: baseline, assessment, change, validation](diagrams/hardening-workflow.svg)

## Scope

| Workstream | Initial read-only observation | Current evidence state |
|---|---|---|
| Windows Defender Antivirus | Antivirus, real-time protection, signature state | **Not collected** |
| Windows Firewall | Domain, Private, Public all `Enabled=True`; both action fields `NotConfigured` | **Verified local profile query; effective actions pending** |
| Security auditing | Relevant advanced audit subcategories and retention | **Prior case evidence only; fresh baseline pending** |
| Local accounts and permissions | Limited, anonymized policy/role review | **Not collected** |
| Update and platform security | Update posture and selected native protections | **Not collected** |
| Recommendations and validation | Prioritized, evidence-based change candidates | **Not implemented** |

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

**Assessment:** All three profiles are enabled. `NotConfigured` is not proof that the effective inbound/outbound behavior is unrestricted; the effective policy still requires verification. **No configuration changes performed.** The next check will query the active policy store, displaying only the same four fields.

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
