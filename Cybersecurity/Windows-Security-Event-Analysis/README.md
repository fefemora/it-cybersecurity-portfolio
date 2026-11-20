# <img src="../../assets/icons/cybersecurity.svg" width="32" alt="Cybersecurity"> Windows Security Event Analysis

![Windows](https://img.shields.io/badge/Windows-Security_Events-14283F?style=flat-square) ![PowerShell](https://img.shields.io/badge/PowerShell-Read--only-14283F?style=flat-square) ![Evidence](https://img.shields.io/badge/REAL_CASE-Initial_Assessment_Complete-059669?style=flat-square)

[← Cybersecurity](../README.md) · [Portfolio](../../README.md) · **[Executive summary](docs/executive-summary.md)** · [Technical findings](docs/event-analysis.md) · [Lab guide](docs/lab-guide.md) · [Recommendations](docs/monitoring-recommendations.md)

[![Windows security event investigation](diagrams/security-review-overview.svg)](docs/executive-summary.md)

I reviewed real Windows Security logs to understand the activity visible on a workstation, then checked how its audit settings and limited retention affected the investigation. A bounded correlation between events **4624** and **4672** was verified. The retained Security log covered approximately **2.03 days** at the time of measurement.

**Finding:** a useful initial monitoring baseline, with clear visibility gaps. **No incident or unauthorized activity was confirmed.**

## Investigation approach

1. **Check the available history:** confirm log settings, retention limits and which event IDs are present.
2. **Look at small, relevant samples:** classify successful logons, built-in accounts and privilege assignments.
3. **Correlate carefully:** match 4624 and 4672 using session IDs without publishing account identifiers.
4. **Report what the evidence supports:** document audit gaps, the unexplained historical 4688 records and what remains unknown.

The [12-step PowerShell lab guide](docs/lab-guide.md) records the individual commands and outputs. The investigation did not establish a security incident.

## Where to go deeper

The [executive summary](docs/executive-summary.md) gives the short version. The [technical analysis](docs/event-analysis.md) contains the findings and limits, while the [lab guide](docs/lab-guide.md) preserves the commands. Monitoring changes are [recommendations](docs/monitoring-recommendations.md), not implemented controls.

## Initial findings

The first successful query returned **1,302 matching records** for four IDs: 4624 (638), 4634 (37), 4672 (614), and 4688 (13). The queried IDs 4625 and 1102 were not returned. These are **event records, not unique users or confirmed incidents**. A separate, later sample of the 100 most recent 4624 events returned **100 service logons (Logon Type 5)**. This is a recent sample, **not** the distribution of all 638 earlier records. An additional, bounded five-event check classified all five as the built-in **LocalSystem** security principal (`S-1-5-18`); this does not establish the account category of all 100 earlier records. The current audit policy enables **success and failure logon auditing**, **success special-logon auditing**, and **success logoff auditing**, but **process creation auditing is disabled** at inspection time. Five recent 4688 records were dated **2026-11-16 14:30:43**, with consecutive record IDs **2266582–2266586**. The cause of their presence while process auditing is currently disabled is **not established**; a prior policy change is only a hypothesis. A subsequent query for event 4719 returned `NoMatchingEventsFound`; this does not rule out historical or unretained changes. Five recent event 4672 records were observed on **2026-11-17 between 20:44:56 and 21:00:46**; a separate later sample of three 4672 records was classified as **LocalSystem** and each carried the same **12 named privileges**. The sample includes a newer event at **21:05:00**. Privilege assignment does not demonstrate privilege use. A later bounded correlation returned **one matching 4624** for a selected event 4672 timestamped **2026-11-17 21:08:51**, comparing `SubjectLogonId` with `TargetLogonId` locally and exposing only a boolean and count. This establishes one observed session linkage, not misuse of privileges. A separate, later correlation-and-classification run returned `MatchFound=True`, `LogonType=5`, and `AccountCategory=LocalSystem`; the specific event timestamp was not output, so it must **not** be asserted to be the exact earlier 21:08:51 session. No malicious activity has been established.

Read the [step-by-step guide](docs/lab-guide.md) and [initial event analysis](docs/event-analysis.md) for commands, observed output, caveats, and next steps.

## Evidence status

Initial counts were provided as copied PowerShell console output; independent log export and exact collection timestamp have not yet been recorded. Do not interpret absent IDs as proof that the underlying activity never occurred. The log-configuration query was subsequently completed: enabled, 33,898 records, 20 MiB maximum size, circular retention. The oldest retained event is dated **2026-11-15 20:02:39** and newest **2026-11-17 20:40:43** (local timestamps in the user's console); the 2.03-day span is not a guarantee of continuous collection. This constrains interpretation of the earlier seven-day lookback.

## References & Official Documentation

- [Microsoft Learn — auditpol /get](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/auditpol-get)
- [Microsoft Learn — Audit Process Creation](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-process-creation)
- [Microsoft Learn — Get-WinEvent](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent)
- [Microsoft Learn — Windows security auditing](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/basic-security-audit-policies)
- [MITRE ATT&CK — Enterprise tactics](https://attack.mitre.org/tactics/enterprise/)

External references support interpretation and methods, not proof of execution.
