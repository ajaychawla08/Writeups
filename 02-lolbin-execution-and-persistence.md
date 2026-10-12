# Write-up 2: Living-off-the-land execution and persistence

> Simulated scenario on synthetic data (same incident as write-up 1).

**Alerts (all on SRV-FILE01, user `CORP\svc_backup`):**
- 10:19:40 PowerShell Encoded Command (T1059.001, T1027)
- 10:21:05 Certutil Download (T1105)
- 10:22:30 Scheduled Task To User-Writable Path (T1053.005)

**Investigation**
1. Decoded the `-enc` payload (base64 of UTF-16LE text): it runs `whoami /all`, which is host and privilege discovery.
2. `certutil -urlcache` fetched `tool.exe` from the same IP that performed the brute force, into `C:\Users\Public`.
3. A task named `Updater` was created to run that file at logon. Compared with the legitimate `NightlyBackup` task, which points to `C:\Program Files` and was created by `itadmin`, the new one points to a user-writable folder and was created by the compromised account.

**Findings:** after the valid logon, the attacker enumerated privileges, downloaded a tool using a built-in binary, and set up persistence.

**Recommendations**
- Isolate the host, collect `tool.exe`, hash it, and submit the hash for enrichment (see `ioc-enricher`).
- Remove the `Updater` task and check other hosts for the same task name and file path.
- Block outbound certutil downloads and enable PowerShell script block logging.
