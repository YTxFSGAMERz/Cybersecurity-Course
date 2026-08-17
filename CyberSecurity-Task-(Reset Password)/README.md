# Task 09 — Reset Password

**TryHackMe · Tutedude Cybersecurity Track**
**Room:** tutedude-cybersec · **Target:** SecureCorp Portal (`192.168.155.129`)
**Prepared by:** Farhan · **Date:** August 16, 2026

## Summary

The SecureCorp Portal's password-reset flow verifies identity with a 4-digit OTP sent to the target username. The verification endpoint (`/verify_otp.php`) never expires that code and enforces no attempt limit, so the entire 10,000-value keyspace can be brute-forced in seconds. This was used to reset the password of the `admin` account without knowing its original credentials.

- **Vulnerability:** Improper Restriction of Excessive Authentication Attempts (CWE-307, related CWE-640)
- **OWASP Category:** A07:2021 — Identification and Authentication Failures
- **Severity:** Critical — unauthenticated takeover of any account, including admin

```
FLAG{BRUTE_FORCE_BYPASS_SUCCESS}
```

## How it was done (short version)

1. Requested a reset for `admin` at `/forgot_password.php`.
2. Opened `/verify_otp.php` and checked the session in DevTools — the page itself states codes never expire and carry no attempt limit, and the only session artifact needed was the `PHPSESSID` cookie.
3. Brute-forced `/verify_otp.php` with `ffuf` across all 4-digit codes (`0000`–`9999`), filtering out the known "wrong code" response size (`-fs 2842`). One value, `1337`, returned a distinct `302` / `0`-byte response.
4. Submitted `1337` — password reset succeeded and the flag was revealed.
5. Submitted the flag on the lab platform — accepted.

The full write-up (objective, methodology, vulnerability analysis, impact, remediation) is in `9__Reset_Password.docx` / `.pdf`.

## Folder contents

| File | Description |
|---|---|
| `9__Reset_Password.docx` | Full report — Word format |
| `9__Reset_Password.pdf` | Full report — PDF rendition |
| `evidence/01_forgot_password_admin_username.png` | `/forgot_password.php` — reset requested for `admin` |
| `evidence/02_verify_otp_page_devtools_cookies.png` | `/verify_otp.php` + DevTools: page discloses no expiry / no attempt limit |
| `evidence/03_phpsessid_cookie_value.png` | Captured `PHPSESSID` used in the brute-force request |
| `evidence/04_ffuf_brute_force_otp_1337.png` | `ffuf` run isolating OTP `1337` |
| `evidence/05_account_recovery_successful_flag.png` | Reset confirmed for `admin`, flag revealed |
| `evidence/06_flag_correct_answer_confirmation.png` | Flag accepted on the lab platform |

## Tools used

- [ffuf](https://github.com/ffuf/ffuf) v2.2.1
- Browser DevTools (Application → Storage → Cookies)
- SecLists — `Fuzzing/4-digits-0000-9999.txt`
