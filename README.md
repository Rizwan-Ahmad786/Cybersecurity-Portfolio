# Cybersecurity Portfolio

This repository contains my hands-on cybersecurity labs and penetration testing practice work.  
Each lab documents my methodology, mindset, and findings in a structured, professional format.

The goal of this portfolio is to demonstrate practical offensive security skills developed during my eJPT preparation and hands-on laboratory engagements.

---

## eJPT Certification Labs

### Windows: Insecure RDP Service (Non-Standard Port)
- **Objective:** Identify and exploit RDP services moved to non-standard ports to bypass automated detection.
- **Methodology:** Performed service fingerprinting using Nmap and Metasploit's `rdp_scanner` to confirm RDP identity on port 3333.
- **Exploitation:** Executed a targeted credential brute-force attack via `Hydra` against the Administrator account.
- **Access:** Established a secure GUI session using `xfreerdp` (NLA supported) to retrieve final objectives.

### Windows: SMB Brute Force & PSexec
- **Objective:** Exploit misconfigured SMB services through credential abuse.
- **Methodology:** Enumerated SMB service details and used `smb_login` to recover valid high-privilege credentials.
- **Exploitation:** Leveraged the `PSexec` module to achieve Remote Code Execution (RCE) and upgrade to a Meterpreter session.
- **Post-Exploitation:** Demonstrated administrative system access and sensitive data retrieval.

### Windows: IIS WebDAV Exploitation
- **Objective:** Exploit misconfigured WebDAV write permissions on a Windows IIS server.
- **Exploitation:** Achieved RCE via a file upload and execution chain.
- **Access:** Gained a stable Meterpreter session using the Metasploit Framework.

## Technical Focus Areas
- **Network Penetration Testing:** SMB, RDP, HTTP/IIS, and service fingerprinting.
- **Protocol Analysis:** Understanding NLA, NetBIOS, and SMB signing configurations.
- **Tool Mastery:** Metasploit Framework, Hydra, Nmap, and xfreerdp.
- **Exploitation Chains:** Moving from reconnaissance to initial access and post-exploitation.

## Windows: WinRM Exploitation (Credential Attack + Remote Execution)
- **Objective:** Identify an exposed WinRM service, obtain valid credentials through brute-force, and gain remote command execution.
- **Methodology:** Discovered WinRM on port 5985 using Nmap, performed credential brute-force against the administrator account using CrackMapExec, and validated access via remote command execution.
- **Initial Access:** Leveraged valid credentials with Metasploit (`winrm_script_exec`) to establish a Meterpreter session and execute commands on the target system.
- **Technical Insight:** Used default execution method (PowerShell) by keeping `FORCE_VBS` disabled, allowing more stable payload delivery. Alternative execution methods can be used if restrictions are present.
- **Post-Exploitation:** Verified access, navigated the system, and retrieved the flag confirming successful compromise.
- **Impact:** Demonstrates that exposed WinRM services combined with weak credentials provide a direct path to full system compromise without requiring traditional exploits.

---

## Disclaimer
All work is performed in controlled lab environments for educational purposes only. Unauthorized access to computer systems is illegal.
