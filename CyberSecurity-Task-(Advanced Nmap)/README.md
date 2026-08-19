# Task 12 – Advanced Nmap (High-Port Recon & Banner Grabbing)

**Author:** Farhan Shaikh
**Track:** TuteDude Cybersecurity — TryHackMe
**Room:** https://tryhackme.com/jr/tutedude-cybersec
**Target:** `192.168.155.129`
**Date:** 16 August 2026
**Status:** ✅ Confirmed — Flag Captured

## 🎯 Objective
Perform a comprehensive Nmap scan and retrieve the flag using Nmap scripts / banner grabbing on a non-standard high port.

## 🛠️ Tools Used
- Nmap 7.99 (Windows PowerShell attack host)

## 📋 Methodology

**1. Full TCP port sweep** across all 65,535 ports to enumerate the complete attack surface:
```
nmap -Pn -p- -T4 192.168.155.129
```
→ Open ports: `21` (ftp), `22` (ssh), `80` (http), `139` (netbios-ssn), `445` (microsoft-ds), `8080` (http-proxy), **`65534` (unknown)**

**2. Service detection + banner grab** targeted at the unusual high port:
```
nmap -Pn -sV --script=banner -p65534 192.168.155.129
```
→ Nmap's `banner` NSE script returned the flag directly in the unauthenticated service response (GenericLines / HTTPOptions / NULL probes).

## 🚩 Flag
```
FLAG{HIGH_PORT_RECON_SUCCESS}
```

## 📂 Package Contents
| File | Description |
|---|---|
| `Task12_Advanced_Nmap_Report.docx` | Full VAPT-style report (Redline format) — objective, methodology, findings, evidence & recommendations |
| `Screenshots/Screenshot_2026-08-16_182245.png` | Full 65,535-port scan output |
| `Screenshots/Screenshot_2026-08-16_182616.png` | Banner grab output — flag visible in terminal |
| `Screenshots/1787136076005_image.png` | TryHackMe correct-answer confirmation |
| `README.md` | This file |

## 🔍 Key Takeaway
Non-standard high ports aren't "security by obscurity" — a single unauthenticated banner grab exposed the service data instantly. Always scan the full 65,535-port range, not just the default top-1000.
