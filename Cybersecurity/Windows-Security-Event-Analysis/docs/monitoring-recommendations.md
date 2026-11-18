# Monitoring Improvement Recommendations — Proposed Actions

[Project overview](../README.md) · [Executive summary](executive-summary.md) · [Technical findings](event-analysis.md) · [Lab guide](lab-guide.md)

**Status: Proposed only.** None of the actions below were executed or validated as part of this read-only exercise. Prioritization reflects this **single-workstation lab**, not an organizational risk assessment or a compliance determination.

| Priority | Proposal | Supporting observation | Validation if approved |
|---|---|---|---|
| 1 — High | Define a target Security log retention period; evaluate capacity and forwarding options | Maximum 20 MiB, Circular; measured earliest-to-latest span 2.03 days | Recheck log properties and oldest/newest record dates over time; verify collection/forwarding if deployed |
| 2 — Medium | Pilot **Audit Process Creation** in a controlled change window | Subcategory `Process Creation` reported `No Auditing` | Record before/after `auditpol` configuration; safely validate generation of new 4688 events; monitor event volume |
| 3 — Medium | Establish a baseline for 4624/4672 patterns, separating system service logons from unusual principals | Bounded samples were dominated by LocalSystem; one 4624/4672 logon-ID match verified | Compare bounded, privacy-safe samples across dates and account categories |
| 4 — Medium | Define a lightweight evidence collection and sanitization process | Console output only; collection runs are not timestamp-aligned and no signed/exported evidence retained | Record collection timestamp, query, bounded scope, sanitized result, and limitations |
| 5 — Investigate if justified | Seek additional evidence for 4688-versus-current-policy discrepancy | Five 4688 records at 2026-11-16 14:30:43; later `No Auditing`; no matching 4719 | Review authorized historical configuration/change records; retain unresolved status without evidence |

## Operational and privacy considerations

- A larger Security log or centralized event collection consumes disk/network resources. Size and target duration should be established **before** applying a change.
- Process-creation auditing can increase event volume; command-line auditing can expose sensitive arguments. Review privacy and access controls before a pilot.
- Object-access auditing may require additional audit settings and can be high-volume. Do not enable every subcategory indiscriminately.
- Use built-in tools and small bounded queries first, given the limited resources of the lab machine.
- Any configuration change requires separate approval and a before/after verification plan; **this assessment made none**.

## Suggested next assessment

After choosing a reasonable retention target, repeat time-span measurements on different days, record timestamps, and compare against the target. Separately, evaluate whether enabling Process Creation auditing is justified by the use case and logging overhead. Do not assume a change has been implemented until verified.

## References & Official Documentation

- [Microsoft Learn — Event log mode and retention behavior](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.eventing.reader.eventlogmode)
- [Microsoft Learn — Audit Process Creation](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-process-creation)
- [Microsoft Learn — Windows Event Forwarding and intrusion detection](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/use-windows-event-forwarding-to-assist-in-intrusion-detection)
- [Microsoft Learn — Audit Special Logon](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-special-logon)

References support recommended approaches, not confirmation of deployment.
