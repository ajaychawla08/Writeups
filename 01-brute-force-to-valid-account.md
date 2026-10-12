# Write-up 1: Brute force that ended in a valid logon

> Simulated scenario on synthetic data. Reproduce it with `detection-lab-demo/generate_logs.py` and `detect.py`.

**Alert:** Brute Force Followed By Success (MITRE T1110, T1078), 10:17:16, host SRV-FILE01.

**Investigation**
1. Filtered Security events for the account `svc_backup`: 40 failed logons (4625, type 3) from `203.0.113.50` between 10:14:00 and 10:16:30, then one successful logon (4624) at 10:17:16 from the same IP.
2. Compared with normal behavior: other users show isolated failures from internal addresses (`10.0.0.x`), and none of them come close to this rate.
3. Checked what the account did after logon (see write-up 2).

**Findings:** a service account was guessed over the network from an external address. A burst of about 10 failures per minute, followed by a success from the same source, is the signature.

**Recommendations**
- Reset the credentials of `svc_backup` and block the source IP.
- Restrict service accounts from interactive and network logon from outside the server segment.
- Alert on failures-then-success per source and account, not on failures alone.

**Tuning notes:** a single typo (as with `bob` in the data) must not alert. The threshold is 10 failures in 5 minutes, then a success within 10 minutes.
