# Windows Security Event Analysis Lab

**Area:** Cybersecurity / Blue Team  
**Status:** In progress — real event counts and Security log configuration verified; investigation not yet completed  
**Environment:** Windows workstation, local Security event log, Windows PowerShell. No SIEM or additional virtual machines required.

[Portfolio home](../../README.md) · [Lab guide](docs/lab-guide.md) · [Event analysis](docs/event-analysis.md)

## Objective

Investigate native Windows Security events with read-only PowerShell commands, interpret the results without assuming malicious activity, and produce reproducible, privacy-safe evidence for an entry-level security monitoring workflow.

## Investigation workflow

1. Verify Security log configuration and retention (**completed after correcting an initial paste-order error**).
2. Count selected event IDs over a seven-day lookback (**completed**).
3. Inspect selected events' fields with sensitive values redacted (**pending**).
4. Evaluate explanations, limitations, and security significance (**pending**).
5. Publish a concise, evidence-backed case report (**pending**).

## Initial findings

The first successful query returned **1,302 matching records** for four IDs: 4624 (638), 4634 (37), 4672 (614), and 4688 (13). The queried IDs 4625 and 1102 were not returned. These are **event records, not unique users or confirmed incidents**. No malicious activity has been established.

Read the [step-by-step guide](docs/lab-guide.md) and [initial event analysis](docs/event-analysis.md) for commands, observed output, caveats, and next steps.

## Evidence status

Initial counts were provided as copied PowerShell console output; independent log export and exact collection timestamp have not yet been recorded. Do not interpret absent IDs as proof that the underlying activity never occurred. The log-configuration query was subsequently completed: enabled, 33,898 records, 20 MiB maximum size, circular retention. The exact collection timestamp and oldest retained event remain unknown.

## References & Official Documentation

- [Microsoft Learn — Get-WinEvent](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent)
- [Microsoft Learn — Windows security auditing](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/basic-security-audit-policies)
- [MITRE ATT&CK — Enterprise tactics](https://attack.mitre.org/tactics/enterprise/)

External references support interpretation and methods, not proof of execution.
