# 🚩 Improper Session Management — Cookie-Based Privilege Escalation

**Target:** SecureCorp Employee Portal — `http://192.168.155.129`
**Platform:** TryHackMe — TuteDude Cybersecurity Track (Task 6)
**Flag:** `FLAG{COOKIE_MANIPULATION_SUCCESS}` ✅
**Tester:** Farhan · August 15, 2026

## Objective

> Exploit an improper session management vulnerability on the site and capture the flag.

## TL;DR

The portal decided a logged-in user's privilege level from a plaintext `user_role` cookie instead of checking it server-side. Flip the cookie from `employee` → `admin` in Burp, forward the request, and the app hands you the admin view — flag included.

## Tools Used

- Burp Suite Community Edition v2026.7.2 (Proxy / Intercept / Repeater)
- Browser DevTools / `view-source`

## Attack Path

1. **Recon** — Sent a throwaway login attempt through Burp, pushed it to Repeater, and reviewed the raw HTML response (cross-checked via `view-source`). Found a leftover developer comment in `index.php`:
   > *"Hey John Welcome onboard use your first name as username and 'babayaga' as your password and remove this comment after you login"*
2. **Login** — Authenticated as `john` / `babayaga`.
3. **Session analysis** — Reloaded `/dashboard.php` with Burp's intercept on. Two cookies were present: `PHPSESSID` (opaque session ID, fine) and `user_role=employee` (plaintext role, not fine).
4. **Tamper** — Edited the intercepted `GET /dashboard.php` request in Burp's Inspector: `user_role=employee` → `user_role=admin`.
5. **Forward** — Server accepted the modified cookie with zero re-validation. Dashboard re-rendered with a "High Alert: privilege escalation detected via cookie tampering" banner containing the flag.
6. **Capture** — `FLAG{COOKIE_MANIPULATION_SUCCESS}` submitted on the lab platform and validated as correct.

## Root Cause

Authorization was decided from a value the client fully controls (`user_role` cookie), with no server-side session-store lookup, no signature, and no integrity check binding it to the authenticated session.

## Impact

Any authenticated user → full admin, via one edited cookie. No re-authentication, rate-limiting, or detection observed.

## Fix

- Resolve role/privilege server-side from a trusted store keyed by the session ID — never trust a client-supplied role.
- If any role indicator must live client-side, sign or encrypt it and verify server-side on every request.
- Re-check authorization on every privileged page/endpoint, not just at login.
- Strip hardcoded credentials / dev comments from shipped HTML before going live.

## What's in this submission

```
.
├── README.md
├── 6. Improper Session Management.docx    ← full VAPT finding report (exec summary, walkthrough, CWE/CVSS, remediation)
└── evidence/                              ← raw screenshots captured during testing
```

> Note: 2 of the 10 screenshots referenced in the report (Burp's "Intercept on" empty state, and the final "Correct Answer" confirmation) weren't retained as files on upload — they're described directly in the report's walkthrough and evidence index instead of embedded.

---
*Educational lab exercise on a designated training target (SecureCorp is not a real organization). Documented for personal portfolio / lab submission purposes.*
