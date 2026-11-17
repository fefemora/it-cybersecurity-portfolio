# Windows Security Event Analysis — Initial Findings

[Project overview](../README.md) · [Reproduction guide](lab-guide.md)

**Case status:** Initial triage only; no confirmed security incident.  
**Data source:** User-provided PowerShell console output from the local Windows Security log.  
**Time scope:** Rolling seven-day window relative to command execution; exact collection time not supplied.

## Verified Security log configuration

| Property | Observed value |
|---|---|
| LogName | Security |
| IsEnabled | True |
| RecordCount | 33,898 |
| MaximumSizeInBytes | 20,971,520 bytes (20 MiB) |
| LogMode | Circular |

The log is enabled and stores records. Circular retention can overwrite older records, so a seven-day query does not guarantee complete seven-day historical coverage. The oldest and newest retained events were subsequently identified: **2026-11-15 20:02:39** and **2026-11-17 20:40:43** (local console timestamps), a span of **2.03 days**. This does not establish uninterrupted logging or prove events were overwritten.

## Verified retained time window

| Metric | Observed result |
|---|---|
| Oldest retained Security event | 2026-11-15 20:02:39 |
| Newest retained Security event | 2026-11-17 20:40:43 |
| Elapsed time between endpoints | 2.03 days |

The earlier seven-day query should **not** be described as seven days of observed activity. Only approximately two days of retained history were present when the later retention query was run.

## Verified observations

| Event ID | Records returned | Meaning |
|---|---:|---|
| 4624 | 638 | Successful logon |
| 4634 | 37 | Logoff |
| 4672 | 614 | Special privileges assigned to a new logon |
| 4688 | 13 | Process creation |
| 4625 | Not returned | Failed logon |
| 1102 | Not returned | Security audit log cleared |

**Total returned records: 1,302.** Counts represent records matching the query, not distinct sessions, accounts, processes, or security incidents.

## Interpretation and limitations

- The high 4672 count cannot be classified as suspicious without examining logon context, account types, and related events. Legitimate system and administrative logons can generate these records.
- The absence of 4625 or 1102 from the result set does **not** prove those events never occurred.
- The query suppresses non-terminating errors with `-ErrorAction SilentlyContinue`; a later validation should consider error visibility and audit policy.
- The Security log configuration was verified after correcting the first command's paste-order error. Circular retention limits what can be inferred about historical coverage.
- No event details, screenshots, or exported logs have been reviewed yet. No attribution, root cause, or incident severity is established.

## Next investigation steps

1. Inspect the audit policy relevant to logon and process creation if needed.
2. Summarize the distribution of logon types among event 4624 records without exposing account names or network identifiers.
3. Select a small number of representative 4624 and 4672 records for privacy-safe field-level review.
4. Document hypotheses and supporting or contradictory evidence.

## References & Official Documentation

- [Microsoft Learn — Event 4624](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624)
- [Microsoft Learn — Event 4672](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4672)
- [Microsoft Learn — Event 4688](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688)
- [Microsoft Learn — Get-WinEvent](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent)

References explain event semantics; actual observations come only from the supplied console output.
