# How-To: Investigating Windows Security Events with PowerShell

[Project overview](../README.md) · [Executive summary](executive-summary.md) · [Event analysis](event-analysis.md) · [Recommendations](monitoring-recommendations.md) · [Portfolio home](../../../README.md)

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

**Observed result (verified after retry):**

```text
LogName            : Security
IsEnabled          : True
RecordCount        : 33898
MaximumSizeInBytes : 20971520
LogMode            : Circular
```

**Interpretation:** The Security log is enabled, with 33,898 stored records and a configured maximum of **20 MiB** (20,971,520 bytes). `Circular` means older records may be overwritten as the log fills. The oldest and newest retained event timestamps are verified in Step 3.

**Troubleshooting note:** The first attempted paste placed `Select-Object` before `Get-WinEvent`, leaving a trailing pipeline operator and causing `ParserError: EmptyPipeElement`. This was a command-entry error, **not** a Security log failure. The corrected command ran successfully.

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

## Step 3 — Determine the retained Security log time span

```powershell
$oldest = Get-WinEvent -LogName Security -Oldest -MaxEvents 1
$newest = Get-WinEvent -LogName Security -MaxEvents 1

[PSCustomObject]@{
    OldestEvent  = $oldest.TimeCreated
    NewestEvent  = $newest.TimeCreated
    RetainedDays = [math]::Round(
        ($newest.TimeCreated - $oldest.TimeCreated).TotalDays, 2
    )
}
```

**Purpose:** Find the timestamp of the first and last currently retained Security records, then calculate the elapsed time between them.

**Observed output (real PowerShell console result; local date formatting):**

```text
OldestEvent         NewestEvent         RetainedDays
-----------         -----------         ------------
15-11-2026 20:02:39 17-11-2026 20:40:43         2,03
```

**Interpretation:** Retained records span **approximately 2.03 days**, from November 15 to November 17, 2026. The previous seven-day filtered query could not provide a full seven days of history based on this subsequently observed retained range. The log is configured for circular retention, which **may** overwrite older entries, but this result alone does not prove overwrite occurred or establish uninterrupted logging across the interval.

**Limitations:** The timestamp range is one point-in-time observation. Event counts and retention metadata came from separate command runs; do not assume they share identical collection times.

## Step 4 — Classify a bounded sample of successful logons

**Original attempt:** An unbounded query retrieved every available 4624 event and processed each XML record. It did not return promptly in the user's session. It was replaced with a capped 100-event query; no configuration change was made.

**Successful bounded command:**

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4624
} -MaxEvents 100 |
ForEach-Object {
    $xml = [xml]$_.ToXml()
    ($xml.Event.EventData.Data |
        Where-Object Name -eq 'LogonType').'#text'
} |
Group-Object |
Sort-Object Count -Descending |
Select-Object Name, Count
```

**Purpose:** Extract and count only the logon type from the 100 most recent event 4624 records. This avoids collecting or displaying usernames, IP addresses, or full event messages.

**Observed result (real console output):**

```text
Name Count
---- -----
5      100
```

**Interpretation:** All 100 sampled successful logon records were **Logon Type 5 (Service)**. Service logons may be routine Windows activity, and this result does **not** establish malicious or unauthorized access.

**Sampling limitation:** This is a cap of 100 **most recent** records, not a random sample and not a full analysis of the 638 records from the earlier seven-day query. The circular log may also continue changing between commands; the 100-record result must not be extrapolated to all logons.

## Step 5 — Classify the account category in a five-event sample

An initial account-grouping query over 100 records did not return promptly. To minimize workstation processing, the analysis was reduced to **five** recent event 4624 records.

**Successful command (one-line version actually executed):**

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4624} -MaxEvents 5 | ForEach-Object { $e = [xml]$_.ToXml(); $d = @{}; foreach ($x in $e.Event.EventData.Data) { $d[$x.Name] = $x.'#text' }; [PSCustomObject]@{ LogonType=$d['LogonType']; AccountCategory=$(switch ($d['TargetUserSid']) { 'S-1-5-18' {'SYSTEM'; break} 'S-1-5-19' {'LOCAL SERVICE'; break} 'S-1-5-20' {'NETWORK SERVICE'; break} default {'Other'} }) } }
```

**Purpose:** Extract the logon type and classify the target security identifier into well-known built-in Windows account categories without displaying usernames, network addresses or complete event details.

**Observed output (real PowerShell console result):**

```text
LogonType AccountCategory
--------- ---------------
5         SYSTEM
5         SYSTEM
5         SYSTEM
5         SYSTEM
5         SYSTEM
```

**Interpretation:** All five sampled records were service logons (type 5) associated with the well-known **LocalSystem SID (S-1-5-18)**. This is consistent with legitimate Windows service activity and is **not** by itself evidence of compromise.

**Sampling and performance limitations:** This is a small, most-recent-events sample; it does not prove that the other 95 records in the earlier sample—or the previously counted 638 records—belonged to LocalSystem. The successful five-record command does not establish precisely why the prior 100-record query was slow.

## Step 6 — Inspect advanced audit policy (verified)

```powershell
auditpol /get /category:*
```

**Purpose:** Display the workstation's effective system audit policy without changing configuration.

**Observed excerpts (translated from Spanish console labels, preserving reported settings):**

| Audit subcategory | Observed setting |
|---|---|
| Logon | Success and Failure |
| Logoff | Success |
| Special Logon | Success |
| Account Lockout | Success |
| Process Creation | No Auditing |
| Process Termination | No Auditing |
| File System | No Auditing |
| Registry | No Auditing |
| Audit Policy Change | Success |
| User Account Management | Success |
| Security Group Management | Success |
| System Integrity | Success and Failure |

**Interpretation:** The current policy supports collecting successful and failed logon audit events and successful special-logon events. No returned 4625 records in an earlier query is not proof that failed-logon auditing is disabled or that failures never occurred.

**Finding requiring follow-up:** The first event count included **13 records of event ID 4688**, but **Process Creation is currently set to No Auditing**. This could reflect earlier policy settings; no cause is confirmed. Review the timestamps of a small bounded sample before drawing conclusions.

**Security visibility note:** Lack of Process Creation, Registry, and File System auditing constrains certain types of investigations. Enabling additional categories may increase log volume and expose sensitive data; configuration changes should be considered separately and are not part of this read-only lab phase.

## Step 7 — Inspect timestamps of a bounded 4688 sample (verified)

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4688} -MaxEvents 5 |
    Select-Object TimeCreated, Id, RecordId
```

**Purpose:** Identify when recent process creation audit events were logged without displaying process names, executable paths, commands, or accounts.

**Observed console output:**

```text
TimeCreated           Id RecordId
-----------           -- --------
16-11-2026 14:30:43 4688  2266586
16-11-2026 14:30:43 4688  2266585
16-11-2026 14:30:43 4688  2266584
16-11-2026 14:30:43 4688  2266583
16-11-2026 14:30:43 4688  2266582
```

**Interpretation:** The five most recent retained 4688 records share the same second (**November 16, 2026 at 14:30:43**) and have consecutive event record identifiers. This is consistent with multiple process-creation records in rapid succession, but the processes and the reason for auditing at that time have not been examined.

**Unresolved:** The subsequent audit policy check showed `Process Creation: No Auditing`. These observations were made at different times. A previous policy change is one possible explanation, **not a verified cause**.

## Step 8 — Search for system audit policy changes (verified: no matches)

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4719
} -MaxEvents 5 |
Select-Object TimeCreated, Id, RecordId
```

**Purpose:** Look for at most five retained events indicating a change to system audit policy.

**Observed result:** No rows were returned. PowerShell reported `NoMatchingEventsFound` (`ObjectNotFound`), meaning **no currently retained Security events matched ID 4719** in this query.

**Interpretation:** This is a valid negative finding, not a malfunction of the event log. It **does not establish that audit policy has never changed**, particularly because the log uses circular retention with a previously observed span of approximately 2.03 days. The reason for the earlier 4688 records alongside the currently disabled Process Creation policy remains **unresolved**.

## Step 9 — Inspect recent special-logon event metadata (verified)

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4672
} -MaxEvents 5 |
Select-Object TimeCreated, Id, RecordId
```

**Purpose:** Obtain a bounded set of event 4672 metadata without displaying accounts, IP addresses, or privilege assignments.

**Observed output:**

```text
TimeCreated           Id RecordId
-----------           -- --------
17-11-2026 21:00:46 4672  2286080
17-11-2026 20:57:13 4672  2286068
17-11-2026 20:54:58 4672  2285921
17-11-2026 20:53:51 4672  2285919
17-11-2026 20:44:56 4672  2285681
```

**Interpretation:** Five special-logon audit records were found on November 17, 2026, between 20:44:56 and 21:00:46. An event 4672 records assignment of special privileges to a new logon; it is not, by itself, evidence of unauthorized activity. No distinct-user count, associated account category, or exact privileges can be determined from these metadata alone.

## Step 10 — Inspect privilege assignment in three recent 4672 records (verified)

```powershell
$events = Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4672
} -MaxEvents 3

$results = foreach ($event in $events) {
    $xml = [xml]$event.ToXml()
    $data = @{}

    foreach ($item in $xml.Event.EventData.Data) {
        $data[$item.Name] = $item.'#text'
    }

    $category = switch ($data['SubjectUserSid']) {
        'S-1-5-18' { 'LocalSystem'; break }
        'S-1-5-19' { 'LocalService'; break }
        'S-1-5-20' { 'NetworkService'; break }
        default { 'Other' }
    }

    [PSCustomObject]@{
        TimeCreated = $event.TimeCreated
        AccountCategory = $category
        Privileges = $data['PrivilegeList']
    }
}

$results | Format-List
```

**Purpose:** Check a very small recent sample of privileged logons, classifying only built-in account security identifiers and displaying the names of assigned privileges. Raw usernames and session identifiers are not included.

**Observed results (user-provided console output):**

| Local timestamp | Account category | Number of privilege names |
|---|---|---:|
| 2026-11-17 21:05:00 | LocalSystem | 12 |
| 2026-11-17 21:00:46 | LocalSystem | 12 |
| 2026-11-17 20:57:13 | LocalSystem | 12 |

Each of these three events contained the same privilege list:

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

**Interpretation:** These are expected-capability categories for highly privileged Windows system contexts. The events show privileges assigned **at logon**; they do not prove that the rights were exercised, that activity was malicious, or that three distinct accounts or services were involved.

**Sampling limitations:** Only three recent records were inspected, and this is a separate snapshot from the prior five-event metadata query. The new event at 21:05:00 illustrates that the Security log continues to change. No process attribution or causal correlation has yet been established.

## Step 11 — Correlate special-logon and successful-logon events (verified)

```powershell
$special = Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4672
} -MaxEvents 1

$xml = [xml]$special.ToXml()
$logonId = ($xml.Event.EventData.Data |
    Where-Object Name -eq 'SubjectLogonId').'#text'

$start = $special.TimeCreated.AddMinutes(-2)
$end = $special.TimeCreated.AddMinutes(1)

$matches = @(Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4624
    StartTime = $start
    EndTime = $end
} -MaxEvents 100 | Where-Object {
    $e = [xml]$_.ToXml()
    ($e.Event.EventData.Data |
        Where-Object Name -eq 'TargetLogonId').'#text' -eq $logonId
})

[PSCustomObject]@{
    Event4672Time = $special.TimeCreated
    Matching4624Events = $matches.Count
    CorrelationFound = ($matches.Count -gt 0)
}
```

**Purpose:** Compare the selected 4672 event's `SubjectLogonId` with the `TargetLogonId` fields of 4624 records in a narrow window (two minutes before and one minute after the selected 4672). The query returns only the time, match count, and correlation flag—not the raw logon identifier or usernames.

**Observed output (actual console result):**

```text
Event4672Time       Matching4624Events CorrelationFound
-------------       ------------------ ----------------
17-11-2026 21:08:51                  1             True
```

**Interpretation:** In the bounded query, one successful-logon event (4624) matched the logon identifier of the selected special-privileges-assigned event (4672). This supports a session-level relationship. It does **not** establish the account category for that exact pair, prove any privileges were exercised, or demonstrate malicious behavior.

**Limitations:** One 4672 event and up to 100 candidate 4624 events were inspected. The event log can change over time, and this result must not be generalized to every 4672 event. The time and logon ID correlation were performed locally; no raw identity or identifier was shared.

## Step 12 — Classify the account and logon type of a correlated session (verified)

**Purpose:** Independently select the most recent 4672, find a matching 4624 using its logon ID in a narrow time window, and report only a boolean result, logon type, and built-in account category.

**Reproducible read-only command:**

```powershell
$special = Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4672
} -MaxEvents 1

$s = [xml]$special.ToXml()
$logonId = ($s.Event.EventData.Data |
    Where-Object Name -eq 'SubjectLogonId').'#text'

$start = $special.TimeCreated.AddMinutes(-2)
$end = $special.TimeCreated.AddMinutes(1)

$match = Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4624
    StartTime = $start
    EndTime = $end
} -MaxEvents 100 | Where-Object {
    $x = [xml]$_.ToXml()
    ($x.Event.EventData.Data |
        Where-Object Name -eq 'TargetLogonId').'#text' -eq $logonId
} | Select-Object -First 1

if ($match) {
    $x = [xml]$match.ToXml()
    $d = @{}
    foreach ($item in $x.Event.EventData.Data) {
        $d[$item.Name] = $item.'#text'
    }

    $category = switch ($d['TargetUserSid']) {
        'S-1-5-18' { 'LocalSystem'; break }
        'S-1-5-19' { 'LocalService'; break }
        'S-1-5-20' { 'NetworkService'; break }
        default { 'Other' }
    }

    [PSCustomObject]@{
        MatchFound = $true
        LogonType = $d['LogonType']
        AccountCategory = $category
    }
} else {
    [PSCustomObject]@{
        MatchFound = $false
        LogonType = 'N/A'
        AccountCategory = 'N/A'
    }
}
```

**Troubleshooting note:** An intermediate attempt to inspect `$match` returned `No matching event stored. Rerun Step 12.` because the variable did not contain a matching event in that session. Running the complete standalone query subsequently succeeded.

**Observed output (user-provided console):**

```text
MatchFound LogonType AccountCategory
---------- --------- ---------------
      True 5         LocalSystem
```

**Finding:** A successful logon event 4624 was matched to a selected special-logon event 4672. The 4624 record was **Logon Type 5 (Service)** and its target account was the built-in **LocalSystem** principal.

**Scope:** This run selected the **latest 4672 at execution time**, not an event pinned to a specific record ID. The output did not include the selected timestamp, so the matched session cannot be asserted to be the same 4672 previously observed at 21:08:51. The result does not establish privilege use or malicious activity.

## Case reporting (completed)

The [executive summary](executive-summary.md) consolidates verified observations, the still-unresolved 4688/audit-policy discrepancy, and the known retention and sampling limitations. The [monitoring recommendations](monitoring-recommendations.md) describe proposed actions only. No system configuration was changed.

## Documentation rules

- Label real observations separately from expected output and hypotheses.
- Keep the exact command, purpose, observed output, interpretation, and limitations for each step.
- Do not fabricate screenshots, collection timestamps, alert results, or conclusions.

## References & Official Documentation

- [Microsoft Learn — auditpol /get](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/auditpol-get)
- [Microsoft Learn — Audit Process Creation](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-process-creation)
- [Microsoft Learn — Get-WinEvent](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent)
- [Microsoft Learn — Event 4624](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624)
- [Microsoft Learn — Event 4634](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4634)
- [Microsoft Learn — Event 4672](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4672)
- [Microsoft Learn — Well-known security identifiers](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers)
- [Microsoft Learn — Event 4719: System audit policy was changed](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4719)
- [Microsoft Learn — Event 4688](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688)
