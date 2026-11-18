# Change and Validation Plan — CHG-001 Implemented, Monitoring Pending

[Project overview](../README.md) · [Assessment matrix](security-assessment.md) · [Baseline guide](baseline-guide.md)

**State:** **CHG-001 authorized, executed and maximum-size configuration verified**. Seven-day retention remains unverified. Separate Process Creation auditing change is still a proposal; no rollback reported.

## Change-control checklist

- [ ] Obtain a complete, read-only baseline for the chosen control and record timestamps.
- [ ] Specify the security objective and why the proposed change is needed.
- [ ] Identify whether the device is standalone, domain-managed or MDM-managed; confirm policy precedence.
- [ ] Assess user impact, remote-access dependencies, privacy and event-volume implications.
- [ ] Establish a safe recovery/rollback procedure suitable for that *specific* change.
- [ ] Approve one narrow change; avoid enabling or disabling unrelated features.
- [ ] Execute only when ready and authorized, then independently capture post-change evidence.
- [ ] Confirm functionality and record any deviations or rollback.

## Possible controlled improvements (not applied)

| Candidate | Why consider it | What would prove success | Risks to manage |
|---|---|---|---|
| Firewall posture review | Ensure intended profiles are protected | Actual enabled state plus connectivity testing appropriate to device role | Disruption of remote services or applications |
| Defender health improvements | Detect unavailable protection or aging signatures | Fresh protection/status output | Third-party AV management and policy precedence |
| Security log retention review — **CHG-001** | Before: 33,959 records / 20 MiB Circular, 1.93-day span; after: **128 MiB, 33,989 records, Circular, enabled** | **Maximum-size configuration verified**; follow-up 7-day retention goal pending | Disk consumption, event churn, shorter-than-target retention, policy conflict |
| Process creation auditing pilot | **Fresh query** confirms `No Auditing` | After approved enabling, verify Success audit policy and new benign 4688 records | Higher event volume, faster overwrite if generated volume outpaces the configured Security log capacity, sensitive fields, managed policy conflicts |

## Proposed controlled change: Process Creation auditing (not approved or executed)

**Verified before-state:** A read-only call to `auditpol /get /subcategory:"{0cce922b-69ae-11d9-bed3-505054503030}"` returned **Creación del proceso — Sin auditoría**. A prior event-analysis project independently reported the same state. This is not evidence of how the policy was configured at the time of older 4688 records.

**Proposed objective:** Enable **Success** auditing for process creation so future, appropriately configured process launches can be recorded as event **4688**. Do not enable Failure auditing or command-line capture as part of the initial proposal.

**Preconditions:**

1. Confirm the workstation is personally controlled or that local changes are authorized, and determine whether AD Group Policy/MDM manages audit settings.
2. Confirm adequate disk space and choose an evidence retention goal. A fresh query found **33,959 records, 20 MiB maximum, Circular**; a separate earlier lab measured ~**2.03 days** between retained endpoints. A fresh time-span check found **1.93 days** between oldest/newest events, down **0.10 days** compared with the earlier independent snapshot; this does not establish overwrite or a trend. Adding process events could reduce retained history.
3. Record the current audit subcategory state and Security log properties, plus a timestamped, privacy-safe change note.
4. Set a planned observation window and rollback criterion (for example, unexpected event volume or disruption to the expected logging baseline).
5. Do **not** publish command lines, executable paths, or user identities from event 4688 without sanitization.

**Candidate command — DOCUMENTED ONLY, NOT RUN:**

```powershell
auditpol /set /subcategory:"{0cce922b-69ae-11d9-bed3-505054503030}" /success:enable
```

This requires an elevated shell and a deliberate approval. It changes only the requested success-auditing option, not general audit-policy categories. Whether it remains effective can depend on managed policy enforcement.

**Proposed verification, only after authorized implementation:**

```powershell
auditpol /get /subcategory:"{0cce922b-69ae-11d9-bed3-505054503030}"
```

Then launch a benign, pre-agreed process and check for relevant **new** 4688 records after the change, using bounded queries and no sensitive process data in public artifacts. Confirm system and application functionality and watch the Security log's growth and oldest-record timestamp. A settings query alone is not proof that the events were generated.

**Rollback plan — DOCUMENTED ONLY, NOT RUN:** If the pre-change state was indeed **No Auditing**, and policy management allows a local reversal, restore Success auditing to disabled:

```powershell
auditpol /set /subcategory:"{0cce922b-69ae-11d9-bed3-505054503030}" /success:disable
```

Check the resulting policy again. This rollback is appropriate **only** if the saved before-state matches this assessment and no other control authority or intervening change requires a different restoration procedure. Do not delete existing events or modify unrelated audit settings.

**Status:** **Proposal only; no change ID or authorized implementation outcome exists.**

## Current capacity check — verified before-state

```powershell
Get-WinEvent -ListLog Security |
    Select-Object LogName, RecordCount, MaximumSizeInBytes, LogMode |
    Format-List
```

**Observed:** Security; **33,959 records**; **20,971,520 bytes (20 MiB) maximum**; **Circular**. Compared with **33,898** records measured in the earlier event-analysis lab, the count differs by **61**, but this is *not* evidence that exactly 61 new events occurred because circular retention and the different collection times prevent that inference.

**Follow-up verified:** Oldest retained event **2026-11-16 00:52:07**, newest **2026-11-17 23:06:29**, elapsed **1.93 days**. Compared with the prior separate **2.03-day** observation, the difference is **0.10 days (~2h 24m)**. These snapshots are not evidence of a specific rollover event. **Next prerequisite completed:** C: free space was verified at **56.81 GiB**. **Subsequently approved and implemented as CHG-001:** 128 MiB maximum was verified after the change; the seven-day target remains pending.

## Security log capacity pilot — CHG-001 implemented and configuration verified

**Evidence-based inputs:**

- **Current maximum:** 20,971,520 bytes = 20 MiB, Circular.
- **Retained endpoint interval:** 1.93 days (different earlier snapshot: 2.03 days).
- **System drive free space:** 56.81 GiB on C: (read-only `Get-PSDrive` observation).
- **Design target:** 7 days retained (proposed, not verified).
- **Initial pilot maximum:** **128 MiB** (134,217,728 bytes; a 108 MiB increase over the current maximum), not a promise of seven-day coverage.

**Execution:** The user confirmed ownership and explicitly authorized changing only the Security log maximum. The command `wevtutil sl Security /ms:134217728` was executed without printed error. A subsequent `Get-WinEvent -ListLog Security` check verified the new configured maximum and Circular mode. Future checks must measure retention span; do not claim that seven days are already retained. The process creation audit setting was not changed in this action.

**Rollback consideration:** The previous maximum was 20 MiB; reducing capacity back to that value could truncate retained history depending on actual log usage. Decide whether restoration is necessary and safe *before* reducing it. Do not automatically run a rollback command.

**Status:** Owner-authorized, implemented and size configuration verified; seven-day retention not yet assessed after elapsed time.

## CHG-001 — Security event log maximum size increase (implemented and configuration verified)

**Authorization:** Workstation owner expressly confirmed personal ownership and authorized changing **only** the Security log maximum size from 20 MiB to 128 MiB.

**Pre-change evidence:** `LogName=Security`; `RecordCount=33959`; `MaximumSizeInBytes=20971520` (**20 MiB**); `LogMode=Circular`. The prior `IsEnabled` state was not captured in this particular pre-change query.

**Executed command** (user PowerShell; no error printed):

```powershell
wevtutil sl Security /ms:134217728
```

**Executed post-change verification:**

```powershell
Get-WinEvent -ListLog Security |
    Select-Object LogName,
                  IsEnabled,
                  RecordCount,
                  MaximumSizeInBytes,
                  LogMode |
    Format-List
```

**Actual output:**

```text
LogName            : Security
IsEnabled          : True
RecordCount        : 33989
MaximumSizeInBytes : 134217728
LogMode            : Circular
```

**Validated outcome:** The configured maximum grew from **20 MiB to 128 MiB** (6.4x, **+108 MiB**), Security is enabled in the post-change query, and the log remains **Circular**. The later record count is 33,989 versus 33,959 in the earlier separate check, **not evidence of precisely 30 new events** because circular overwrite may occur. No clear-log command was part of the recorded steps; individual event continuity was **not** independently verified.

**Not yet validated:** The proposed **7-day retained-history target**. A capacity increase alone does not prove days retained, and the log needs a follow-up observation over time. The independently verified pre-change retained timestamp span was **1.93 days**, measured before this change. **Process Creation auditing remains a separate, unimplemented proposal**, last observed as `No Auditing` before CHG-001.

**Change status:** **Implemented and post-change maximum-size configuration verified**. Long-term retention outcome **pending**; no rollback performed.

## Evidence fields for every *future* approved change

| Field | To be recorded |
|---|---|
| Change ID | **CHG-001** (Security log maximum size) |
| Approval and scope | Owner confirmed personal device and explicitly approved 20 → 128 MiB only |
| Before snapshot | Security, 33,959 records, 20 MiB, Circular |
| Exact command or administrative action | `wevtutil sl Security /ms:134217728` |
| After snapshot | Security, `IsEnabled=True`, 33,989 records, 128 MiB, Circular |
| Operational validation and rollback result | Maximum-size configuration verified; retained-history objective pending; rollback not performed |
| Outcome | **CHG-001 implemented and configuration verified; retention target pending** |

## References & Official Documentation

- [Microsoft Learn — Security Compliance Toolkit](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/security-compliance-toolkit-10)
- [Microsoft Learn — Windows security baselines](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/windows-security-baselines)
- [Microsoft Learn — Audit Process Creation](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-process-creation)
