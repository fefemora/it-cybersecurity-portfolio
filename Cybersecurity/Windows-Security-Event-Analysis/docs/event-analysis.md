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

## Verified logon-type sample

A later bounded command retrieved the **100 most recent event 4624 records** and extracted the `LogonType` XML field. The output contained one group:

| Logon type | Interpretation | Count in sampled records |
|---|---|---:|
| 5 | Service logon | 100 |

**Interpretation:** Service logons can be normal operating-system activity; no malicious behavior was established from this sample. The initial unbounded XML-processing query did not return promptly and was replaced by a limited query. This was an efficiency/troubleshooting change, not an evidence of a failed audit log.

**Limitation:** These are the latest 100 records at the time of sampling, not all 638 events from the earlier count and not a representative random sample.

## Verified account-category sample

A later **five-event** bounded query classified `TargetUserSid` for the five most recent event 4624 records into well-known account categories without revealing individual account names.

| Logon type | Classified principal | Sampled count |
|---|---|---:|
| 5 (Service) | LocalSystem (`S-1-5-18`) | 5 |

**Interpretation:** These five service logons are compatible with normal system behavior. This does not establish what service generated them or whether every earlier 4624 record belonged to this account. A larger account-classification command did not return promptly, so the validated query was limited to five records.

## Verified audit-policy assessment

The read-only `auditpol /get /category:*` output was collected from the Windows workstation. Selected subcategories are shown below; reported configurations are translated from Spanish.

| Subcategory | Current setting | Implication |
|---|---|---|
| Logon | Success and Failure | Both successful and failed logon auditing configured |
| Logoff | Success | Successful logoff auditing configured |
| Special Logon | Success | Special logon events configured |
| Process Creation | No Auditing | No process-creation audit enabled at inspection time |
| Process Termination | No Auditing | Process termination auditing disabled |
| File System / Registry | No Auditing | These object-access subcategories disabled |
| Audit Policy Change | Success | Successful audit-policy changes configured |
| System Integrity | Success and Failure | Both outcomes configured |

**Potential visibility gap:** Process creation auditing is disabled at the time of inspection, limiting future 4688 coverage unless policy changes. This is a configuration observation, **not** a confirmed security incident or proof of noncompliance.

**Unresolved discrepancy:** The initial seven-day-lookback query returned **13 events with ID 4688**, despite the current Process Creation policy being disabled. A historical policy change is one possible explanation, not a verified conclusion; inspect record timestamps and policy-change evidence before interpreting.

**Privacy/performance:** No audit-policy modifications were made. The next check should be a bounded, metadata-only query to avoid exposing process paths or command-line arguments.

## Interpretation and limitations

- The high 4672 count cannot be classified as suspicious without examining logon context, account types, and related events. Legitimate system and administrative logons can generate these records.
- The absence of 4625 or 1102 from the result set does **not** prove those events never occurred.
- The query suppresses non-terminating errors with `-ErrorAction SilentlyContinue`; a later validation should consider error visibility and audit policy.
- The Security log configuration was verified after correcting the first command's paste-order error. Circular retention limits what can be inferred about historical coverage.
- The `LogonType` field was inspected in a 100-record sample, and `TargetUserSid` was classified in a separate five-record sample; detailed account/context fields, screenshots, and exported logs have not yet been reviewed. No attribution, root cause, or incident severity is established.

## Next investigation steps

1. Check timestamps for at most five retained events 4688 without retrieving their process details.
2. If useful, expand beyond the latest service-logon-heavy sample using bounded queries.
3. Review selected 4624 and 4672 records for privacy-safe context and correlation.
4. Document hypotheses and supporting or contradictory evidence.

## References & Official Documentation

- [Microsoft Learn — auditpol /get](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/auditpol-get)
- [Microsoft Learn — Audit Process Creation](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-process-creation)
- [Microsoft Learn — Event 4624](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624)
- [Microsoft Learn — Well-known security identifiers](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers)
- [Microsoft Learn — Event 4672](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4672)
- [Microsoft Learn — Event 4688](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688)
- [Microsoft Learn — Get-WinEvent](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent)

References explain event semantics; actual observations come only from the supplied console output.
