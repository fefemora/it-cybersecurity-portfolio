# Windows Security Event Analysis Lab

**Area:** Cybersecurity / Blue Team  
**Status:** In progress — real event counts, log configuration, retained date range and sampled logon types and a five-event built-in account classification and local audit policy verified; investigation not yet completed  
**Environment:** Windows workstation, local Security event log, Windows PowerShell. No SIEM or additional virtual machines required.

[Portfolio home](../../README.md) · [Lab guide](docs/lab-guide.md) · [Event analysis](docs/event-analysis.md)

## Objective

Investigate native Windows Security events with read-only PowerShell commands, interpret the results without assuming malicious activity, and produce reproducible, privacy-safe evidence for an entry-level security monitoring workflow.

## Investigation workflow

1. Verify Security log configuration (**completed after correcting an initial paste-order error**).
2. Count selected event IDs over a seven-day lookback (**completed; retained data covers only about 2.03 days**).
3. Verify oldest/newest retained timestamps (**completed**).
4. Sample and classify 100 recent successful logons by logon type (**completed: all 100 were type 5 / Service**).
5. Classify built-in account SIDs on a five-record 4624 sample (**completed: 5 of 5 LocalSystem / S-1-5-18**).
6. Review current advanced audit policy with `auditpol /get /category:*` (**completed; read-only**).
7. Inspect selected event fields with sensitive values redacted (**pending**).
8. Evaluate explanations, limitations, and security significance (**pending**).
9. Publish a concise, evidence-backed case report (**pending**).

## Initial findings

The first successful query returned **1,302 matching records** for four IDs: 4624 (638), 4634 (37), 4672 (614), and 4688 (13). The queried IDs 4625 and 1102 were not returned. These are **event records, not unique users or confirmed incidents**. A separate, later sample of the 100 most recent 4624 events returned **100 service logons (Logon Type 5)**. This is a recent sample, **not** the distribution of all 638 earlier records. An additional, bounded five-event check classified all five as the built-in **LocalSystem** security principal (`S-1-5-18`); this does not establish the account category of all 100 earlier records. The current audit policy enables **success and failure logon auditing**, **success special-logon auditing**, and **success logoff auditing**, but **process creation auditing is disabled** at inspection time. Previously counted 4688 records may predate this policy state; their timestamps have not yet been checked. No malicious activity has been established.

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
