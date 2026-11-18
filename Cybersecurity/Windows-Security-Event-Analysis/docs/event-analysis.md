# Windows Security Event Analysis — Initial Findings

[Project overview](../README.md) · [Executive summary](executive-summary.md) · [Reproduction guide](lab-guide.md) · [Recommendations](monitoring-recommendations.md)

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

## Verified event 4688 timestamp sample

A read-only query extracted metadata from the **five most recent retained** event 4688 records.

| Local timestamp | Event ID | Record ID |
|---|---|---|
| 2026-11-16 14:30:43 | 4688 | 2266586 |
| 2026-11-16 14:30:43 | 4688 | 2266585 |
| 2026-11-16 14:30:43 | 4688 | 2266584 |
| 2026-11-16 14:30:43 | 4688 | 2266583 |
| 2026-11-16 14:30:43 | 4688 | 2266582 |

**Finding:** All five share a one-second timestamp and consecutive record IDs. They are compatible with a burst of logged process creations. The current Process Creation subcategory was subsequently observed as `No Auditing`, but the audit policy at the exact earlier event time is not established. No change event, specific process, security incident, or root cause is confirmed.

**Follow-up result:** A bounded event 4719 query found no matches. The audit-policy history remains unconfirmed.

## Verified event 4719 search result

The read-only query `Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4719} -MaxEvents 5 | Select-Object TimeCreated, Id, RecordId` returned **no matching events** and PowerShell reported `NoMatchingEventsFound`.

This supports only the statement that **no event 4719 record was returned from the currently retained log**. It does not prove that audit policy never changed, that a change took place, or why event 4688 records exist while Process Creation auditing is currently disabled. The log's circular retention and limited retained history constrain this finding.

## Verified event 4672 metadata sample

A bounded read-only query returned the five latest event 4672 records:

| Local timestamp | Event ID | Record ID |
|---|---|---:|
| 2026-11-17 21:00:46 | 4672 | 2286080 |
| 2026-11-17 20:57:13 | 4672 | 2286068 |
| 2026-11-17 20:54:58 | 4672 | 2285921 |
| 2026-11-17 20:53:51 | 4672 | 2285919 |
| 2026-11-17 20:44:56 | 4672 | 2285681 |

**Assessment:** This is evidence that five records for assignment of special privileges to new logons exist in the retained Security log. It does not show whether they belong to five different sessions or accounts, which privileges were assigned, or whether any activity was unauthorized. Further correlation or a privacy-safe field-level sample is needed to assess context.

## Verified 4672 privilege sample

A later bounded three-event query parsed the `SubjectUserSid` and `PrivilegeList` XML fields and returned the following anonymized observations:

| Local timestamp | Account category | Privileges assigned |
|---|---|---:|
| 2026-11-17 21:05:00 | LocalSystem (`S-1-5-18`) | 12 |
| 2026-11-17 21:00:46 | LocalSystem (`S-1-5-18`) | 12 |
| 2026-11-17 20:57:13 | LocalSystem (`S-1-5-18`) | 12 |

**Identical privilege list in all three records:**

- `SeAssignPrimaryTokenPrivilege`
- `SeTcbPrivilege`
- `SeSecurityPrivilege`
- `SeTakeOwnershipPrivilege`
- `SeLoadDriverPrivilege`
- `SeBackupPrivilege`
- `SeRestorePrivilege`
- `SeDebugPrivilege`
- `SeAuditPrivilege`
- `SeSystemEnvironmentPrivilege`
- `SeImpersonatePrivilege`
- `SeDelegateSessionUserImpersonatePrivilege`

**Assessment:** LocalSystem is a highly privileged built-in Windows security principal. This sample is compatible with legitimate operating-system activity, but lacks information about the services and logon sessions that generated the records. Event 4672 means special privileges were assigned to new logons; **assignment does not demonstrate actual use**, nor does it prove abuse. Three event records are not necessarily three separate accounts or services.

**Limitations:** These are only the three latest records at that particular collection time, not all 614 records from the initial query. No identity, executable, session correlation, or misuse has been established.

## Verified 4672-to-4624 session correlation

A bounded read-only query compared the `SubjectLogonId` of the latest selected event 4672 with the `TargetLogonId` of event 4624 records in a window from two minutes before to one minute after the 4672 record.

| Field | Observed result |
|---|---|
| Selected 4672 timestamp (local) | 2026-11-17 21:08:51 |
| Matching event 4624 records | 1 |
| `CorrelationFound` | `True` |

**Finding:** One matching successful logon was returned for the selected privileged-logon event, using a matching logon ID. This demonstrates an observed link between the two event types, not attribution to an individual or evidence of unauthorized access. The specific account category for this correlated pair and actual privilege use were **not** established.

**Limitations:** The query examined one recent 4672 and capped the candidate 4624 set at 100 within a narrow time interval. No raw logon ID was disclosed. The observation cannot be generalized to all records.

## Verified correlated logon classification (separate follow-up run)

A standalone, bounded read-only query selected the latest available 4672, compared its `SubjectLogonId` with `TargetLogonId` on nearby 4624 events, and classified the matching account using its well-known target SID.

| Output field | Observed result |
|---|---|
| `MatchFound` | `True` |
| `LogonType` | `5` (Service) |
| `AccountCategory` | `LocalSystem` |

**Assessment:** A successful **service logon under LocalSystem** was correlated with a special-privileges-assigned event. This is consistent with legitimate system activity and does not demonstrate abuse or actual exercise of privileges.

**Critical distinction:** This was a **new query of the latest 4672**, and its result did not include a timestamp or record ID. We **cannot confirm** that it classified the same 21:08:51 event from the earlier correlation check. The earlier unsuccessful variable-only check likewise does not invalidate either successful result.

## Interpretation and limitations

- The high 4672 count cannot be classified as suspicious without examining logon context, account types, and related events. Legitimate system and administrative logons can generate these records.
- The absence of 4625 or 1102 from the result set does **not** prove those events never occurred.
- The query suppresses non-terminating errors with `-ErrorAction SilentlyContinue`; a later validation should consider error visibility and audit policy.
- The Security log configuration was verified after correcting the first command's paste-order error. Circular retention limits what can be inferred about historical coverage.
- The `LogonType` field was inspected in a 100-record sample, and `TargetUserSid` was classified in a separate five-record sample; detailed account/context fields, screenshots, and exported logs have not yet been reviewed. No attribution, root cause, or incident severity is established.

## Next investigation steps

1. Retain the discrepancy between earlier 4688 records and currently disabled Process Creation auditing as unresolved; consider only privacy-safe, bounded checks if further investigation is justified.
2. If useful, expand beyond the latest service-logon-heavy sample using bounded queries.
3. Consolidate the two separate, successful correlation observations without claiming they represent the same session.
4. **Completed:** executive summary and non-implemented monitoring recommendations are available as separate, linked documents.

## References & Official Documentation

- [Microsoft Learn — auditpol /get](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/auditpol-get)
- [Microsoft Learn — Audit Process Creation](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-process-creation)
- [Microsoft Learn — Event 4624](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624)
- [Microsoft Learn — Well-known security identifiers](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers)
- [Microsoft Learn — Event 4672](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4672)
- [Microsoft Learn — Event 4719: System audit policy was changed](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4719)
- [Microsoft Learn — Event 4688](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688)
- [Microsoft Learn — Get-WinEvent](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent)

References explain event semantics; actual observations come only from the supplied console output.
