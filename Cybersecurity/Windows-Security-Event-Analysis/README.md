# Windows Security Event Analysis Lab

**Area:** Cybersecurity / Blue Team  
**Status:** In progress — real event counts, log configuration and retained date range verified; investigation not yet completed  
**Environment:** Windows workstation, local Security event log, Windows PowerShell. No SIEM or additional virtual machines required.

[Portfolio home](../../README.md) · [Lab guide](docs/lab-guide.md) · [Event analysis](docs/event-analysis.md)

## Objective

Investigate native Windows Security events with read-only PowerShell commands, interpret the results without assuming malicious activity, and produce reproducible, privacy-safe evidence for an entry-level security monitoring workflow.

## Investigation workflow

1. Verify Security log configuration (**completed after correcting an initial paste-order error**).
2. Count selected event IDs over a seven-day lookback (**completed; retained data covers only about 2.03 days**).
3. Verify oldest/newest retained timestamps (**completed**).
4. Inspect selected events' fields with sensitive values redacted (**pending**).
5. Evaluate explanations, limitations, and security significance (**pending**).
6. Publish a concise, evidence-backed case report (**pending**).

## Initial findings

The first successful query returned **1,302 matching records** for four IDs: 4624 (638), 4634 (37), 4672 (614), and 4688 (13). The queried IDs 4625 and 1102 were not returned. These are **event records, not unique users or confirmed incidents**. No malicious activity has been established.

Read the [step-by-step guide](docs/lab-guide.md) and [initial event analysis](docs/event-analysis.md) for commands, observed output, caveats, and next steps.

## Evidence status

Initial counts were provided as copied PowerShell console output; independent log export and exact collection timestamp have not yet been recorded. Do not interpret absent IDs as proof that the underlying activity never occurred. The log-configuration query was subsequently completed: enabled, 33,898 records, 20 MiB maximum size, circular retention. The oldest retained event is dated **2026-11-15 20:02:39** and newest **2026-11-17 20:40:43** (local timestamps in the user's console); the 2.03-day span is not a guarantee of continuous collection. This constrains interpretation of the earlier seven-day lookback.

## References & Official Documentation

- [Microsoft Learn — Get-WinEvent](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent)
- [Microsoft Learn — Windows security auditing](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/basic-security-audit-policies)
- [MITRE ATT&CK — Enterprise tactics](https://attack.mitre.org/tactics/enterprise/)

External references support interpretation and methods, not proof of execution.
