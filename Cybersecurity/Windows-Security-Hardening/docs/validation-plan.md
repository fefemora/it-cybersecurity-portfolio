# Change and Validation Plan — Not Yet Executed

[Project overview](../README.md) · [Assessment matrix](security-assessment.md) · [Baseline guide](baseline-guide.md)

**State: Proposal only.** No configuration changes, rollback actions, or post-change validation have happened.

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
| Security log retention review | Prior log history observed at ~2.03 days | Measured observation window meets a documented target | Disk consumption, data access, privacy |
| Process creation auditing pilot | Prior audit policy disabled this subcategory | Audit policy verified and benign test process produces appropriate 4688 | Higher event volume, data sensitivity, policy conflicts |

## Evidence fields for every *future* approved change

| Field | To be recorded |
|---|---|
| Change ID | TBD |
| Approval and scope | TBD |
| Before snapshot | TBD |
| Exact command or administrative action | TBD |
| After snapshot | TBD |
| Operational validation and rollback result | TBD |
| Outcome | **Not performed** |

## References & Official Documentation

- [Microsoft Learn — Security Compliance Toolkit](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/security-compliance-toolkit-10)
- [Microsoft Learn — Windows security baselines](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/windows-security-baselines)
- [Microsoft Learn — Audit Process Creation](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-process-creation)
