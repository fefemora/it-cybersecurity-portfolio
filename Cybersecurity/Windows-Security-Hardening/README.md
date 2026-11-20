# <img src="../../assets/icons/cybersecurity.svg" width="32" alt="Cybersecurity"> Windows Security Hardening

![Windows](https://img.shields.io/badge/Windows-Endpoint_Security-14283F?style=flat-square) ![PowerShell](https://img.shields.io/badge/PowerShell-Baseline_Checks-14283F?style=flat-square) ![Verified](https://img.shields.io/badge/ASSESSMENT-30_Steps_Documented-059669?style=flat-square) ![Follow-up](https://img.shields.io/badge/FOLLOW--UP-Pending-E5A000?style=flat-square)

[← Cybersecurity](../README.md) · [Portfolio](../../README.md) · **[30-step lab guide](docs/baseline-guide.md)** · [Assessment register](docs/security-assessment.md) · [Change and validation plan](docs/validation-plan.md)

[![Windows security hardening workflow](diagrams/hardening-workflow.svg)](docs/baseline-guide.md)

## At a glance

I assessed the security baseline of a Windows workstation using **28 technical checks** followed by two steps covering findings and closure. The work included Windows Firewall, Microsoft Defender, auditing, PowerShell logging, BitLocker, TPM and boot security.

Most of the investigation was read-only. One limited change was approved and applied: **CHG-001**, increasing the maximum size of the Security event log from **20 MiB to 128 MiB**. The new configuration was verified, but the proposed **seven-day retention target has not been demonstrated**.

| Category | What the evidence shows |
|---|---|
| **Firewall** | Domain, Private and Public profiles enabled. Effective profile defaults: inbound **Block**, outbound **Allow** |
| **Microsoft Defender** | Antivirus, real-time protection, Tamper Protection and NIS active in the inspected status fields |
| **Security auditing** | Logon success/failure auditing configured; process-creation auditing reported **No Auditing** |
| **Security log** | CHG-001 increased maximum from **20 to 128 MiB**; immediately observed retained span **1.93 → 1.96 days** |
| **BitLocker** | System volume **fully decrypted**, protection **Off** |
| **Boot security** | TPM ready, UEFI and GPT observed; **Secure Boot disabled** |
| **PowerShell logging** | Operational channel active. Some policy keys absent at inspected paths; retained 4104 events observed |
| **Authentication** | 235 event 4624 records and 233 event 4672 records in separate 24-hour queries; counts alone do not establish misuse |

## CHG-001 · A controlled change, not a completed hardening rollout

**Why change it?** The original 20 MiB circular Security log offered a relatively short window for local investigation.

**Approved action:** increase the log's maximum capacity only.

```powershell
wevtutil sl Security /ms:134217728
```

The post-change query returned `MaximumSizeInBytes=134217728`, `IsEnabled=True` and `LogMode=Circular`. The immediate time span of retained records increased slightly from **1.93** to **1.96 days**. That confirms the configured limit and immediate access—not seven days of history or uninterrupted future retention.

[Read the before/after evidence and validation criteria](docs/validation-plan.md)

## Open findings and follow-up

| Area | Observation | Next step |
|---|---|---|
| **Disk encryption** | BitLocker protection is off | Plan backups, key recovery and an authorized encryption change |
| **Secure Boot** | Disabled despite UEFI/GPT and ready TPM | Check compatibility and recovery before any firmware change |
| **Process auditing** | Creation auditing disabled | Consider a scoped pilot after reviewing event volume and privacy |
| **Retention** | 128 MiB configured; seven days not verified | Measure oldest/newest retained events again after enough time |
| **PowerShell** | Partial telemetry and unknown effective policy coverage | Review coverage requirements before considering additional logging |

These are **recommendations**, not configuration changes that were performed. No security incident was confirmed during the assessment.

## Read the evidence

| Document | Best for |
|---|---|
| **[Lab guide · 30 steps](docs/baseline-guide.md)** | Exact commands, console outputs and step-by-step interpretation |
| **[Security assessment register](docs/security-assessment.md)** | Findings, control coverage, limitations and priorities |
| **[Change and validation plan](docs/validation-plan.md)** | CHG-001 scope, pre/post checks, recovery considerations and pending work |
| **[Related Windows event investigation](../Windows-Security-Event-Analysis/README.md)** | Earlier security-event analysis that motivated the follow-up |

## What this project demonstrates

**Windows security assessment · PowerShell · Endpoint monitoring · Audit policy · Evidence handling · Change control · Verification**

This is an **evidence-based workstation assessment**, not proof that an enterprise baseline has been fully applied or a security standard certified. Detailed observations remain in the linked technical reports; uncertain outcomes remain explicitly marked.

[← Back to Cybersecurity](../README.md)
