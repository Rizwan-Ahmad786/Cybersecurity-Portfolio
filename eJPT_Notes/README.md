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

### Windows: SMB Brute Force & PSexec
* **Objective:** Exploit misconfigured SMB services through credential abuse.
* **Methodology:** Enumerated SMB service details and used `smb_login` to recover valid high-privilege credentials.
* **Exploitation:** Leveraged the `PSexec` module to achieve Remote Code Execution (RCE) and upgrade to a Meterpreter session.
* **Post-Exploitation:** Demonstrated administrative system access and sensitive data retrieval.

### Windows: IIS WebDAV Exploitation
* **Objective:** Exploit misconfigured WebDAV write permissions on a Windows IIS server.
* **Exploitation:** Achieved RCE via a file upload and execution chain.
* **Access:** Gained a stable Meterpreter session using the Metasploit Framework.

### Windows: WinRM Exploitation (Credential Attack + Remote Execution)
* **Objective:** Identify an exposed WinRM service, obtain valid credentials through brute-force, and gain remote command execution.
* **Methodology:** Discovered WinRM on port 5985 using Nmap, performed credential brute-force against the administrator account using CrackMapExec, and validated access via remote command execution.
* **Initial Access:** Leveraged valid credentials with Metasploit (`winrm_script_exec`) to establish a Meterpreter session.
* **Technical Insight:** Kept `FORCE_VBS` disabled for stable PowerShell-based payload delivery. Alternative execution methods available when restrictions are present.
* **Post-Exploitation:** Verified access, navigated the system, and retrieved the flag.
* **Impact:** Exposed WinRM with weak credentials provides a direct path to full system compromise without requiring traditional exploits.

### Windows: UAC Bypass & Privilege Escalation (UACMe)
* **Objective:** Escalate from a low-privilege Meterpreter session to `NT AUTHORITY\SYSTEM` by bypassing UAC.
* **Methodology:** Identified Rejetto HFS 2.3x on port 80 via Nmap; exploited unauthenticated RCE using Metasploit's `rejetto_hfs_exec` module.
* **Exploitation:** Migrated to a stable 64-bit process, confirmed UAC restriction, generated a reverse payload with `msfvenom`, and bypassed UAC silently using UACMe (Akagi64.exe, method 23).
* **Post-Exploitation:** Migrated into `lsass.exe` to inherit the SYSTEM token; executed `hashdump` to retrieve all local NTLM password hashes.
* **Impact:** Demonstrates that admin group membership with UAC active does not equal full privileges — and UAC can be bypassed silently using public tooling.

## Windows: Privilege Escalation via Token Impersonation (Incognito)

* **Objective: Abuse SeImpersonatePrivilege on a low-privilege service account to steal an Administrator delegation token and escalate without any exploit or additional payload.
* **Methodology: Gained initial access via Rejetto HFS 2.3x RCE (rejetto_hfs_exec); enumerated session privileges and identified * **SeImpersonatePrivilege; confirmed Explorer migration was blocked at this privilege level; loaded the Incognito Meterpreter extension to list tokens available in memory.
* **Exploitation: Identified an ATTACKDEFENSE\Administrator delegation token in memory; impersonated it using impersonate_token — no exploit, no UAC bypass, no files uploaded for the escalation step. Migration to Explorer succeeded after impersonation, confirming the elevated token was active.
* **Post-Exploitation: Verified full Administrator privilege set via getprivs; navigated the filesystem to C:\Documents and Settings\Administrator\Desktop (older Windows profile path) and retrieved flag.txt, confirming complete system compromise.
* **Impact: Demonstrates that SeImpersonatePrivilege on a service account is a critical and often overlooked escalation path. Token impersonation abuses a legitimate Windows feature, leaving no exploit payload on disk — making it significantly harder to detect than UAC bypass or exploit-based escalation methods.


### Windows: Privilege Escalation via Unattended Installation Credentials

* **Objective:** Escalate from a low-privilege student account to Administrator by discovering and abusing credentials left behind in a Windows unattended installation answer file.
* **Methodology:** Confirmed low-privilege context with `whoami`; ran `Invoke-PrivescAudit` from PowerUp.ps1 (PowerSploit) to audit the system for misconfigurations; identified a leftover `Unattend.xml` file at `C:\Windows\Panther\` containing AutoLogon credentials for the Administrator account.
* **Exploitation:** Read `Unattend.xml` to extract the Base64-encoded Administrator password; decoded it using PowerShell's `System.Convert` class to recover the plaintext credential `Admin@123`; used `runas.exe` to spawn an elevated `cmd.exe` under the Administrator account.
* **Payload Delivery:** Used Metasploit's `hta_server` exploit on Kali to host a malicious HTA payload over HTTP; executed it on the Windows machine using the built-in `mshta.exe` binary (LOLBin) from the Administrator cmd — no custom binary written to disk.
* **Post-Exploitation:** Received Meterpreter session with full Administrator privileges; verified with `sysinfo`, `getuid`, and `getprivs`; retrieved `flag.txt` from `C:\Users\Administrator\Desktop`.
* **Impact:** Demonstrates that unattended installation files left on disk represent a direct, zero-exploit path to full Administrator access. Base64 encoding provides no real protection — credentials stored this way are immediately recoverable by any user with filesystem read access.
---

## Technical Focus Areas

* **Network Penetration Testing:** SMB, RDP, HTTP/IIS, WinRM, and service fingerprinting.
* **Protocol Analysis:** Understanding NLA, NetBIOS, SMB signing, WinRM authentication, and Windows token architecture.
* **Tool Mastery:** Metasploit Framework, Hydra, Nmap, CrackMapExec, msfvenom, UACMe, xfreerdp, Incognito.
* **Exploitation Chains:** Moving from reconnaissance to initial access, privilege escalation, and post-exploitation.
* **Privilege Escalation Techniques:** UAC bypass (UACMe), token impersonation (Incognito), process migration, LSASS access.

---

## Disclaimer

All work is performed in controlled lab environments for educational purposes only.
Unauthorized access to computer systems is illegal.
