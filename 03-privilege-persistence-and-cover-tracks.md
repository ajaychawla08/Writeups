# Write-up 3: New administrator and log clearing

> Simulated scenario on synthetic data (same incident as write-ups 1 and 2).

**Alerts (SRV-FILE01):**
- 10:24:10 Member Added To Local Administrators (T1098): `helpdesk01` added by `svc_backup`.
- 10:26:00 Event Log Cleared (T1070.001): Security log cleared by `svc_backup`.

**Investigation**
1. `helpdesk01` has no earlier activity in the data, and a service account does not normally manage group membership, so I treated the change as unauthorized instead of a ticketed request.
2. The log was cleared 2 minutes after the admin change. Event 1102 itself survives the clearing, which is why it is such a reliable alert.
3. Built the full timeline: brute force (10:14), logon (10:17), discovery (10:19), download (10:21), persistence (10:22), privilege (10:24), cleanup (10:26).

**Findings:** the attacker created a second privileged account for access that survives a reset of `svc_backup`, then tried to hide the trail.

**Recommendations**
- Disable `helpdesk01`, reset all credentials touched from this host, and review who can change local groups.
- Forward Security logs to the SIEM in real time so clearing a local log cannot erase the evidence.
- Compare admin-group changes against approved change tickets; this is the main source of false positives for this rule.
