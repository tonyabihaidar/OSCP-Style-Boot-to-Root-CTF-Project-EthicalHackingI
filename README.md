# OSCP-Style Boot-to-Root Penetration Test — CTF Project
**EECE 503G: Special Topic in Ethical Hacking I — American University of Beirut**

Full black-box penetration test conducted against a vulnerable Linux target machine as a two-person team project, following a structured five-phase methodology: reconnaissance, scanning, enumeration, exploitation, and post-exploitation. All flags were captured through four independent exploitation paths to root.

📄 **[Full technical report (PDF)](./beta-ctf-report.pdf)**

---

## Overview

- **Target:** Ubuntu 16.04 LTS (hostname `BETA`), 7 open services (SSH, HTTP, SMB, custom web app, AJP, Tomcat)
- **Team:** Rita Abou Fares & Tony Abi Haidar
- **Tools:** Nmap, Nikto, Gobuster, Dirsearch, enum4linux, Hydra, LinPEAS, ssh2john, John the Ripper, Metasploit, msfvenom, Nessus, Nexpose

## Methodology

1. **Reconnaissance** — ARP scan and connectivity checks to identify the target on the local subnet; initial web server inspection uncovered a hidden `/development/` directory leaking usernames and password hints.
2. **Scanning & Enumeration** — Full Nmap SYN and vulnerability scans, directory brute-forcing (Dirsearch, Gobuster), SMB enumeration via `smb-enum-users`/enum4linux, and Nessus/Nexpose scans to surface critical CVEs across all open ports.
3. **Exploitation** — Four independent paths to root (see below).
4. **Post-Exploitation** — Full `/etc/shadow` extraction, flag capture, and system enumeration from each foothold.

## Exploitation Paths

| # | Path | Technique |
|---|------|-----------|
| 1 | SSH Brute Force → SUID Escalation | Hydra dictionary attack against SSH, then privilege escalation via a SUID-bit-set `vim.basic` binary (GTFOBins) |
| 2 | SSH Key Extraction → LXD Escalation | LinPEAS enumeration discovered a world-readable SSH private key; cracked its passphrase offline with `ssh2john` + John the Ripper (rockyou.txt), then escalated via `lxd` group membership to mount and chroot into the host filesystem |
| 3 | OS Command Injection | Unsanitized input in a custom "ping-a-host" web app allowed arbitrary command execution as root, delivered via a staged reverse shell |
| 4 | Tomcat Manager WAR Deployment → PwnKit | Recovered plaintext Tomcat Manager credentials, deployed a malicious WAR file containing a JSP reverse shell, then escalated from the `tomcat9` service account to root via PwnKit (CVE-2021-4034) |

**Additional vectors investigated:** GhostCat (CVE-2020-1938, unauthenticated AJP file disclosure via Metasploit), Apache Struts 2.5.12 (dead end — no reachable endpoints), Slowloris DoS (CVE-2007-6750), and DirtyCOW (CVE-2016-5195, unsuccessful on this target).

## Remediation

The report includes a full remediation section addressing **18 identified vulnerabilities** with concrete, copy-pasteable system-hardening fixes, including:

- PAM-enforced password policy and SSH key-based authentication (with Fail2ban)
- Removal of SUID bits from unnecessary binaries and a monthly SUID-baseline audit process
- Input sanitization and running web services under dedicated non-root accounts
- Disabling the exposed AJP connector and rotating Tomcat Manager credentials
- Restricting `lxd`/`docker`/`sudo` group membership
- SSH Terrapin (CVE-2023-48795) mitigation, ICMP timestamp disclosure fix, Samba null-session hardening
- A staged OS upgrade path off end-of-life Ubuntu 16.04

## Contributions

Full task-level contribution breakdown is documented in the report. My individual work included: initial web server inspection, Nexpose/Nessus scan configuration, the full SUID `vim.basic` escalation path, LinPEAS-driven SSH key extraction/cracking and LXD escalation, and investigation of the Struts, Slowloris, and DirtyCOW attack surfaces.
