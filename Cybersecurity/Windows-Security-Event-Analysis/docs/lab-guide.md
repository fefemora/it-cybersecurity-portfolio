# How-To: Investigating Windows Security Events with PowerShell

[Project overview](../README.md) · [Event analysis](event-analysis.md) · [Portfolio home](../../../README.md)

**Status:** Partially executed and verified against the user's console output.  
**Approach:** Read-only queries. No policy changes, event deletion, or simulated attacks.

## Prerequisites

- Windows PowerShell, preferably opened **as administrator** to access the Security log.
- An existing Windows Security event log.
- Do not publish raw usernames, SIDs, IP addresses, hostnames, or event message content without review.

## Step 1 — Inspect the Security log configuration

Run as a **single line** to avoid paste-order issues:

```powershell
Get-WinEvent -ListLog Security | Select-Object LogName, IsEnabled, RecordCount, MaximumSizeInBytes, LogMode
```

**Purpose:** Determine whether the Security log is enabled, how many records it currently holds, its maximum size, and its retention mode.

**Observed result:** **Not yet obtained.** The first attempted paste placed `Select-Object` before `Get-WinEvent`, leaving a trailing pipeline operator and causing `ParserError: EmptyPipeElement`. This was a command-entry error, **not** a Security log failure.

**Next action:** Rerun the single-line version and record the sanitized output.

## Step 2 — Count selected Security events in the last seven days

```powershell
$ids = 4624,4625,4634,4672,4688,1102

Get-WinEvent -FilterHashtable @{
    LogName   = 'Security'
    Id        = $ids
    StartTime = (Get-Date).AddDays(-7)
} -ErrorAction SilentlyContinue |
    Group-Object Id |
    Sort-Object Name |
    Select-Object Name, Count
```

**Purpose:** Retrieve matching records from the Security log over a rolling seven-day period and group them by event ID.

**Observed output (copied from an actual console run):**

```text
Name Count
---- -----
4624   638
4634    37
4672   614
4688    13
```

**Validation:** Four event IDs returned records. IDs 4625 and 1102 did not appear in the grouped output. This does not establish that they never occurred; auditing, retention, permissions, query scope, and the use of `-ErrorAction SilentlyContinue` may affect what is returned.

**Interpretation:** Event 4624 reports a successful logon, 4634 a logoff, 4672 special privileges assigned to a new logon, and 4688 process creation. Counts are not unique people or evidence of malicious behavior.

## Step 3 — Investigate event details (planned)

Before proceeding, confirm Step 1 and select a bounded, privacy-safe inspection query. Record only the fields needed for the investigation and redact identifying information before publication.

## Documentation rules

- Label real observations separately from expected output and hypotheses.
- Keep the exact command, purpose, observed output, interpretation, and limitations for each step.
- Do not fabricate screenshots, collection timestamps, alert results, or conclusions.

## References & Official Documentation

- [Microsoft Learn — Get-WinEvent](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent)
- [Microsoft Learn — Event 4624](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624)
- [Microsoft Learn — Event 4634](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4634)
- [Microsoft Learn — Event 4672](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4672)
- [Microsoft Learn — Event 4688](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688)
