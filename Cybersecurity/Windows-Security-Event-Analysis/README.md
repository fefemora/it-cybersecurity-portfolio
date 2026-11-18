# Windows Security Event Analysis Lab

**Area:** Cybersecurity / Blue Team  
**Status:** Initial read-only assessment documented and reviewed; proposed monitoring improvements remain unimplemented  
**Environment:** Windows workstation, local Security event log, Windows PowerShell. No SIEM or additional virtual machines required.

[Portfolio home](../../README.md) · **[Executive summary](docs/executive-summary.md)** · [Technical findings](docs/event-analysis.md) · [Step-by-step lab guide](docs/lab-guide.md) · [Recommendations](docs/monitoring-recommendations.md)

## Objective

Investigate native Windows Security events with read-only PowerShell commands, interpret the results without assuming malicious activity, and produce reproducible, privacy-safe evidence for an entry-level security monitoring workflow.

## Investigation workflow

1. Verify Security log configuration (**completed after correcting an initial paste-order error**).
2. Count selected event IDs over a seven-day lookback (**completed; retained data covers only about 2.03 days**).
3. Verify oldest/newest retained timestamps (**completed**).
4. Sample and classify 100 recent successful logons by logon type (**completed: all 100 were type 5 / Service**).
5. Classify built-in account SIDs on a five-record 4624 sample (**completed: 5 of 5 LocalSystem / S-1-5-18**).
6. Review current advanced audit policy with `auditpol /get /category:*` (**completed; read-only**).
7. Check five recent 4688 event timestamps (**completed: 2026-11-16 14:30:43**).
8. Search for retained 4719 audit-policy-change events (**completed: no matching events returned**).
9. Collect five recent 4672 timestamps and record IDs (**completed**).
10. Classify accounts and privilege names in three bounded 4672 records (**completed: 3 LocalSystem records with the same 12 privileges**).
11. Correlate one recent 4672 with a 4624 using a local logon ID (**completed: one match found within a bounded time window**).
12. Classify a matched 4624 logon by type and account category (**completed: Type 5 / LocalSystem in a later query**).
13. Produce an evidence-backed case summary (**completed: executive report, proposed actions, and visualization**).

## Executive overview

[![Windows event investigation dashboard](diagrams/security-review-overview.svg)](docs/executive-summary.md)

**Read the [executive summary](docs/executive-summary.md)** for the recruiter-friendly outcome, **[technical analysis](docs/event-analysis.md)** for verifiable results, **[lab guide](docs/lab-guide.md)** for reproducible commands, and **[monitoring recommendations](docs/monitoring-recommendations.md)** for proposed next steps. The assessment is complete as an **initial triage case study**; it does not claim a security incident was discovered.

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
