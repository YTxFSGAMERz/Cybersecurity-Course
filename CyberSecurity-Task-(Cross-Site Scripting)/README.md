# CyberSecurity Task - Cross-Site Scripting - Farhan

## Task

**Objective:** Exploit a Cross-Site Scripting (XSS) vulnerability on the SecureCorp Employee Portal (TryHackMe lab) and pop up an alert to capture the flag.

**Room:** [https://tryhackme.com/jr/tutedude-cybersec](https://tryhackme.com/jr/tutedude-cybersec)

## Summary

Two candidate injection points were identified after logging in with the low-privileged credentials recovered from the client-side source code in a prior task: the **Employee Directory** search field and the **Support Board** "Report an issue" message field.

1. Submitting `<script>alert('XSS attack, you are hacked')</script>` into the **Employee Directory** search box failed — the application threw a raw MySQL/MariaDB `SQLSTATE[42000]` syntax error caused by the unescaped single quote (`'`) in the payload. This indicates the field is vulnerable to SQL injection, but the broken query prevents the payload from ever reaching the browser as valid HTML, so it is not a working XSS vector.
2. Submitting the identical payload into the **Support Board** message field succeeded. The Community Thread stores and later renders the message without any output encoding, so the browser parsed and executed the injected `<script>` tag, firing the `alert()` popup. The application then displayed the objective flag directly beneath the injected post.

**Flag:** `FLAG{STORED_XSS_ACHIEVED}`

The flag was submitted and validated as correct on the TryHackMe platform.

## Vulnerability

- **Type:** Stored Cross-Site Scripting (XSS)
- **OWASP Category:** A03:2021 — Injection
- **CWE:** CWE-79 — Improper Neutralization of Input During Web Page Generation
- **Vulnerable Endpoint:** Support Board → "Report an issue" message field / Community Thread
- **Related Finding:** Employee Directory search field passes input unescaped into a SQL query (potential SQL injection, CWE-89)

## Folder Contents

```
CyberSecurity-Task-XSS-Farhan/
├── README.md                                 - this file
├── 10_Cross-Site_Scripting_Report.docx        - full write-up with methodology, screenshots, and remediation
├── 10. Cross-Site Scripting.docx.md           - original task brief (as provided)
└── evidence/
    ├── 01_employee_directory_sql_error.png    - failed XSS attempt in Employee Directory (SQL exception)
    ├── 02_support_board_before.png            - Support Board / Community Thread before payload submission
    ├── 03_support_board_flag.png              - stored XSS executes, flag displayed (FLAG{STORED_XSS_ACHIEVED})
    └── 04_flag_correct_answer.png             - flag accepted on TryHackMe
```

## How the Attack Works

The Support Board takes free-text user input and, when rendering the Community Thread, inserts it directly into the page's HTML with no sanitization or encoding. Because the browser cannot tell attacker-supplied markup apart from the site's own markup once both are concatenated into the same response, the injected `<script>` tag is parsed and executed with full trust — exactly like the application's own JavaScript. This is what allowed the `alert()` popup to fire and the flag to be revealed.

## Remediation

- HTML-encode all user-supplied content at the point of output.
- Use a templating framework that auto-escapes by default.
- Apply a strict Content-Security-Policy (CSP) to block inline script execution.
- Use parameterized queries for the Employee Directory search to fix the underlying SQL injection issue as well.
- Set `HttpOnly` on session cookies to limit the impact of any script that does execute.

See the full Word report for the detailed step-by-step walkthrough, screenshots, root cause analysis, and complete remediation recommendations.
