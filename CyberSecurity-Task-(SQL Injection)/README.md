# Task 11 — SQL Injection

**Platform:** TryHackMe — `tutedude-cybersec` room
**Target:** SecureCorp Portal — `http://192.168.155.129/`
**Vulnerable Endpoint:** `GET /search.php?q=`
**Tools Used:** Burp Suite (Proxy/Repeater), sqlmap v1.10.8
**Flag:** `Flag{SQLI_EXFILTRATION_SUCCESSFUL}`

## Objective

Find a SQL Injection vulnerability on the SecureCorp Portal, exploit it, and
retrieve the flag by dumping data from the back-end database.

## Summary

1. Logged into the portal as the low-privileged user `john` and opened the
   **Employee Directory**, which searches employee records via `search.php?q=`.
2. Intercepted a search request (`q=admin`) in Burp Suite and appended an
   asterisk (`*`) after the search term to mark the injection point for sqlmap.
3. Exported the marked request from Burp Suite (**Save selected text to
   file**) as `sqlinjection.txt`.
4. Ran `sqlmap -r sqlinjection.txt --dbs --random-agent` to confirm the
   injection and enumerate databases — the back end fingerprinted as
   **MySQL/MariaDB** on Linux/Apache, with three databases available,
   including `securecorp`.
5. Ran `sqlmap -r sqlinjection.txt -D securecorp --tables --random-agent`
   to list tables in `securecorp` — found `users` and `secret_flags`.
6. Ran `sqlmap -r sqlinjection.txt -D securecorp -T secret_flags --dump
   --random-agent` to dump the `secret_flags` table, which contained the
   flag: `Flag{SQLI_EXFILTRATION_SUCCESSFUL}`.
7. Submitted the flag on TryHackMe and confirmed it as correct.

Full methodology, command output, and vulnerability/remediation analysis are
documented in **`SQL-Injection-Report.docx`**.

## Folder Contents

```
CyberSecurity-Task-SQLInjection-YourName/
├── README.md                      This file
├── SQL-Injection-Report.docx      Full write-up with embedded screenshots
├── 11. SQL Injection.docx.md      Original task brief (as provided)
└── evidence/                      Raw screenshots used in the report
    ├── 01_directory_search.png
    ├── 02_burp_intercept_request.png
    ├── 03_burp_save_selected_text.png
    ├── 04_burp_save_dialog_sqlinjection_txt.png
    ├── 05_sqlmap_dbs_start.png
    ├── 06_sqlmap_dbs_result.png
    ├── 07_sqlmap_tables_start.png
    ├── 08_sqlmap_tables_result.png
    ├── 09_sqlmap_dump_start.png
    ├── 10_sqlmap_dump_flag_result.png
    └── 11_flag_accepted.png
```

## Vulnerability

- **Type:** SQL Injection (OWASP A03:2021 — Injection)
- **CWE:** CWE-89 — Improper Neutralization of Special Elements used in an
  SQL Command
- **Root cause:** `search.php` concatenates the `q` parameter directly into
  a back-end SQL query without parameterization or input validation,
  allowing arbitrary query manipulation.
- **Severity:** Critical — full read access to the application database was
  obtained with three automated sqlmap commands, independent of any
  application-level role/session controls.

## Remediation (high level)

- Use parameterized queries / prepared statements for all database access.
- Enforce least-privilege on the application's database account.
- Suppress verbose SQL error output returned to the client.
- Add WAF rules and automated SQLi testing to the CI/CD pipeline as
  compensating and regression controls.

> **Note:** This assessment was performed against a TryHackMe lab
> environment for educational purposes only, as part of the Tutedude
> Cybersecurity Track.
