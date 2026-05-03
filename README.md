# Cybersecurity Portfolio

This repository contains my hands-on cybersecurity labs and penetration testing practice work. Each lab documents my methodology, mindset, and findings in a structured, professional format.

The goal of this portfolio is to demonstrate practical offensive security skills developed during my eJPT preparation and hands-on laboratory engagements.

---

## eJPT Certification Labs

### Windows: Insecure RDP Service (Non-Standard Port)
* **Objective:** Identify and exploit RDP services moved to non-standard ports to bypass automated detection.
* **Methodology:** Performed service fingerprinting using Nmap and Metasploit's `rdp_scanner` to confirm RDP identity on port 3333.
* **Exploitation:** Executed a targeted credential brute-force attack via `Hydra` against the Administrator account.
* **Access:** Established a secure GUI session using `xfreerdp` (NLA supported) to retrieve final objectives.

### Windows: SMB Brute Force and PSexec
* **Objective:** Exploit misconfigured SMB services through credential abuse.
* **Methodology:** Enumerated SMB service details and used `smb_login` to recover valid high-privilege credentials.
* **Exploitation:** Leveraged the `PSexec` module to achieve Remote Code Execution and upgrade to a Meterpreter session.
* **Post-Exploitation:** Demonstrated administrative system access and sensitive data retrieval.

### Windows: IIS WebDAV Exploitation
* **Objective:** Exploit misconfigured WebDAV write permissions on a Windows IIS server.
* **Exploitation:** Achieved RCE via a file upload and execution chain.
* **Access:** Gained a stable Meterpreter session using the Metasploit Framework.

### Windows: WinRM Exploitation (Credential Attack and Remote Execution)
* **Objective:** Identify an exposed WinRM service, obtain valid credentials through brute-force, and gain remote command execution.
* **Methodology:** Discovered WinRM on port 5985 using Nmap, performed credential brute-force against the administrator account using CrackMapExec, and validated access via remote command execution.
* **Initial Access:** Leveraged valid credentials with Metasploit (`winrm_script_exec`) to establish a Meterpreter session.
* **Technical Insight:** Kept `FORCE_VBS` disabled for stable PowerShell-based payload delivery.
* **Post-Exploitation:** Verified access, navigated the system, and retrieved the flag.
* **Impact:** Exposed WinRM with weak credentials provides a direct path to full system compromise without requiring traditional exploits.

### Windows: UAC Bypass and Privilege Escalation (UACMe)
* **Objective:** Escalate from a low-privilege Meterpreter session to `NT AUTHORITY\SYSTEM` by bypassing UAC.
* **Methodology:** Identified Rejetto HFS 2.3x on port 80 via Nmap; exploited unauthenticated RCE using Metasploit's `rejetto_hfs_exec` module.
* **Exploitation:** Migrated to a stable 64-bit process, confirmed UAC restriction, generated a reverse payload with `msfvenom`, and bypassed UAC silently using UACMe (Akagi64.exe, method 23).
* **Post-Exploitation:** Migrated into `lsass.exe` to inherit the SYSTEM token; executed `hashdump` to retrieve all local NTLM password hashes.
* **Impact:** Demonstrates that admin group membership with UAC active does not equal full privileges and UAC can be bypassed silently using public tooling.

### Windows: Privilege Escalation via Token Impersonation (Incognito)
* **Objective:** Abuse `SeImpersonatePrivilege` on a low-privilege service account to steal an Administrator delegation token and escalate without any exploit.
* **Methodology:** Gained initial access via Rejetto HFS RCE; enumerated session privileges and identified `SeImpersonatePrivilege`; confirmed Explorer migration was blocked at this privilege level; loaded the Incognito Meterpreter extension to list tokens available in memory.
* **Exploitation:** Identified an `ATTACKDEFENSE\Administrator` delegation token in memory; impersonated it using `impersonate_token` with no exploit, no UAC bypass, and no files uploaded for the escalation step. Migration to Explorer succeeded after impersonation, confirming the elevated token was active.
* **Post-Exploitation:** Verified full Administrator privilege set via `getprivs`; navigated the filesystem to `C:\Documents and Settings\Administrator\Desktop` and retrieved `flag.txt`, confirming complete system compromise.
* **Impact:** `SeImpersonatePrivilege` on a service account is a critical and often overlooked escalation path. Token impersonation abuses a legitimate Windows feature, leaving no exploit payload on disk. This makes it significantly harder to detect than UAC bypass or exploit-based escalation methods.

### Windows: Privilege Escalation via Unattended Installation Credentials
* **Objective:** Escalate from a low-privilege student account to Administrator by discovering and abusing credentials left behind in a Windows unattended installation answer file.
* **Methodology:** Confirmed low-privilege context with `whoami`; ran `Invoke-PrivescAudit` from PowerUp.ps1 (PowerSploit) to audit the system for misconfigurations; identified a leftover `Unattend.xml` file at `C:\Windows\Panther\` containing AutoLogon credentials for the Administrator account.
* **Exploitation:** Read `Unattend.xml` to extract the Base64-encoded Administrator password; decoded it using PowerShell's `System.Convert` class to recover the plaintext credential; used `runas.exe` to spawn an elevated `cmd.exe` under the Administrator account with no exploit required.
* **Payload Delivery:** Used Metasploit's `hta_server` module on Kali to host a malicious HTA payload over HTTP; executed it from the Administrator cmd using the built-in `mshta.exe` binary (LOLBin). No custom executable was written to disk on the target.
* **Post-Exploitation:** Received Meterpreter session with full Administrator privileges; verified with `sysinfo`, `getuid`, and `getprivs`; retrieved `flag.txt` from `C:\Users\Administrator\Desktop`.
* **Impact:** Unattended installation files left on disk after deployment are a direct, zero-exploit path to full Administrator access. Base64 encoding provides no real protection. Any user with filesystem read access can recover the credentials in seconds.

### CTF: Host and Network Penetration Testing - System/Host Based Attacks (4 Flags)
* **Objective:** Complete a two-target CTF by retrieving four flags across a WebDAV-exposed IIS server and an SMB-exposed Windows machine using credential attacks, file upload exploitation, and remote code execution.
* **Target 1 - WebDAV (Flags 1 and 2):** Identified HTTP Basic Auth on port 80 via Nmap; brute-forced credentials for user `bob` using Hydra with the `unix_passwords` wordlist. Logged into the WebDAV share via `cadaver` and retrieved Flag 1. Used `davtest` to confirm `.asp` file execution is permitted; uploaded a pre-built ASP webshell and accessed it through the browser to read `flag2.txt` from the C drive root.
* **Target 2 - SMB (Flags 3 and 4):** Confirmed SMB on port 445 via Nmap; used Metasploit `smb_login` to brute-force credentials and recover the Administrator account password. Used `exploit/windows/smb/psexec` to establish a Meterpreter session; dropped to shell and retrieved Flag 3 from `C:\` and Flag 4 from `C:\Users\Administrator\Desktop`.
* **Impact:** Demonstrates a full multi-target engagement across two different protocols and attack surfaces. Both flag chains required zero CVE exploitation. All access was gained through credential abuse and misconfiguration, which reflects the most common real-world attack path in enterprise environments.

### Linux: ProFTPD Credential Brute-Force and FTP Exploitation
* **Objective:** Identify an exposed ProFTPD service, confirm anonymous access is disabled, recover valid credentials through brute-force, and retrieve a sensitive file over FTP.
* **Methodology:** Confirmed target with ping and noted Linux TTL; ran Nmap version scan to identify ProFTPD 1.3.5a on port 21; tested anonymous login which was rejected; used Hydra with a unix username list and unix password list to brute-force FTP credentials.
* **Exploitation:** Recovered valid credentials for the `auditor` account; logged in via the FTP client, listed the directory, and located `secret.txt`.
* **Post-Exploitation:** Downloaded `secret.txt` using the `get` command and read the flag locally with `cat`.
* **Impact:** FTP services with weak credentials and no account lockout policy are trivially brute-forced. Standard FTP also transmits credentials in plaintext, meaning any successful login on an unencrypted network is visible to anyone monitoring traffic.


### Linux: SSH Credential Brute-Force with Metasploit (Two Post-Exploitation Methods)
* **Objective:** Identify an exposed SSH service, enumerate the version, recover valid credentials through Metasploit brute-force, and retrieve the flag using two documented post-exploitation approaches.
* **Methodology:** Confirmed target with ping and noted Linux TTL; ran Nmap to identify SSH on port 22; created a dedicated Metasploit workspace; used `ssh_version` to confirm password authentication is enabled; configured `ssh_login` with `common_users.txt` and `common_passwords.txt` using `STOP_ON_SUCCESS true` and `VERBOSE false`.
* **Exploitation:** Recovered valid credentials for the `sysadmin` account; Metasploit automatically opened an SSH session on success.
* **Post-Exploitation Method 1:** Used `sessions -i 1` to interact with the auto-opened Metasploit session; ran `find / -name "flag"` to locate the flag without manual navigation; read it with `cat`.
* **Post-Exploitation Method 2:** Used recovered credentials with the standard `ssh` command in a terminal; navigated the filesystem manually with `cd` and `dir`; read the flag directly.
* **Impact:** SSH with weak credentials and no lockout policy is as vulnerable to brute-force as any cleartext service. Knowing both the Metasploit session approach and manual SSH login provides flexibility across different engagement scenarios.
---

## Technical Focus Areas

* **Network Penetration Testing:** SMB, RDP, HTTP/IIS, WinRM, WebDAV, FTP, and service fingerprinting.
* **Protocol Analysis:** Understanding NLA, NetBIOS, SMB signing, WinRM authentication, WebDAV write permissions, FTP cleartext transmission, and Windows token architecture.
* **Tool Mastery:** Metasploit Framework, Hydra, Nmap, CrackMapExec, msfvenom, UACMe, xfreerdp, Incognito, PowerSploit, cadaver, davtest.
* **Exploitation Chains:** Moving from reconnaissance to initial access, privilege escalation, and post-exploitation.
* **Privilege Escalation Techniques:** UAC bypass (UACMe), token impersonation (Incognito), process migration, LSASS access, unattended installation credential abuse.
* **Living off the Land:** Payload delivery using signed Windows binaries (`mshta.exe`, `runas.exe`) to minimise forensic footprint.

---

## Disclaimer

All work is performed in controlled lab environments for educational purposes only.
Unauthorized access to computer systems is illegal.
