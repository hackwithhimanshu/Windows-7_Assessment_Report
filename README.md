# Windows 7 Security Assessment Report (Lab Environment)

[![OS](https://img.shields.io/badge/Target-Windows%207%20SP1-blue)](https://en.wikipedia.org/wiki/Windows_7) [![Tools](https://img.shields.io/badge/Tools-Nmap%20%7C%20Metasploit-orange)]()  
*(Controlled lab environment — for learning and portfolio use only.)*

---

## Table of Contents
- [Overview](#overview)  
- [Environment](#environment)  
- [Methodology](#methodology)  
- [Service Enumeration (Nmap)](#service-enumeration-nmap)  
- [SMB Version Scan (Metasploit)](#smb-version-scan-metasploit)  
- [Vulnerability Checks](#vulnerability-checks)  
- [Print Spooler Assessment (MS10-061)](#print-spooler-assessment-ms10-061)  
- [Key Findings](#key-findings)  
- [Reproducible Commands (Safe)](#reproducible-commands-safe)  
- [Screenshots & Evidence](#screenshots--evidence)  
- [Conclusions](#conclusions)  
- [How to cite / Use in portfolio](#how-to-cite--use-in-portfolio)  
- [License](#license)

---

## Overview
This repository contains the documented results of a controlled security assessment performed on a Windows 7 Ultimate SP1 virtual machine. The goal was to practice service enumeration, safe vulnerability checking, and professional reporting for a cybersecurity portfolio.

> **Important:** All testing was performed in an isolated lab environment with full authorization. Do not run these tools or procedures against systems you do not own or explicitly have permission to test.

---

## Environment
- **Attacker host:** Kali Linux (VM)  
- **Target host:** Windows 7 Ultimate SP1 (Build 7601) — VirtualBox VM  
- **Network:** Host-only / internal lab network  
- **Tools used:** Nmap, Metasploit Framework (msfconsole), basic OS utilities

---

## Methodology
1. Passive/active enumeration of target services with **Nmap**.  
2. Safe vulnerability checks and service/version enumeration using Metasploit **auxiliary** modules (no destructive tests).  
3. Attempted assessment of historically relevant modules where appropriate, verifying compatibility and patch status.  
4. Document findings and provide remediation recommendations.

---

## Service Enumeration (Nmap)
**Summary of discovered services (Nmap output):**

- `TCP 135` — MSRPC  
- `TCP 139` — NetBIOS-SSN  
- `TCP 445` — SMB (Microsoft-DS)  
- `TCP 49152–49157` — MSRPC dynamic ports  
- `UDP 137` — NetBIOS-NS  
- `UDP 138` — NetBIOS-DGM (open|filtered)  
- `UDP 500` — ISAKMP (open|filtered)  
- `UDP 4500` — NAT-T IKE (open|filtered)  
- `UDP 5355` — LLMNR (open|filtered)

> These results identify SMB and RPC as the main remote attack surface for this default Win7 VM.

---

## SMB Version Scan (Metasploit)
Using Metasploit’s `auxiliary/scanner/smb/smb_version`:

- **SMB versions supported:** 1, 2  
- **Preferred dialect:** SMB 2.1  
- **OS detected:** Windows 7 Ultimate SP1 (Build 7601)  
- **Authentication domain:** WINDOWS-7

**Interpretation:** SMBv1 presence alone does not equal vulnerability. The preferred dialect (SMB 2.1) and OS build suggest the system is likely patched for modern SMB vulnerabilities unless intentionally made vulnerable.

---

## Vulnerability Checks
### MS17-010 (EternalBlue) Check
- **Module used:** `auxiliary/scanner/smb/smb_ms17_010`  
- **Result:** `Host does NOT appear vulnerable.`

**Interpretation:** The target is patched or configured such that the EternalBlue (MS17-010) condition is not present (SMBv1 may be present but is not exploitable on this build/patch level).

---

## Print Spooler Assessment (MS10-061)
- **Module inspected:** `exploit/windows/smb/ms10_061_spoolss` (inspection and safe checks only)  
- **PNAME used (example):** `XPS Document Writer` (printer share name detected on typical Win7 installs)  
- **Observed result:** Attempts returned `STATUS_ACCESS_DENIED` when probing the spooler pipe.  
- **Conclusion:** Under default/standard configuration of this Win7 SP1 VM, the Print Spooler attack surface for MS10-061 is not exploitable.

---

## Key Findings
- SMBv1 is present but **not vulnerable** to MS17-010 on the current build.  
- Windows 7 SP1 (Build 7601) in this lab is patched for the major publicly-known SMB remote code execution issues.  
- Print Spooler (MS10-061) probes returned access denied; target is not vulnerable under default configuration.  
- MSRPC, NetBIOS, and LLMNR are present for enumeration but did not expose a remote-memory-exploitation vector in this configuration.  
- The assessment demonstrated a correct, safe scanning and verification workflow; lack of exploitable results is a valid and important test outcome.

---

## Reproducible Commands (Safe)
> The following commands are for **enumeration and non-destructive checks only**.

### Nmap example
```bash
# Basic service/version scan and scripts for SMB
nmap -sV -p 139,445,135,49152-49157 --script=smb-protocols,smb-enum-shares 192.168.29.54
