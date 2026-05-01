
CTF: Host & Network Penetration Testing - System/Host Based Attacks (4 Flags)

Objective: Complete a two-target CTF by retrieving four flags across a WebDAV-exposed IIS server and an SMB-exposed Windows machine using credential attacks, file upload exploitation, and remote code execution.
Target 1 - WebDAV (Flags 1 and 2):

Identified HTTP Basic Auth on port 80 via Nmap script scan; brute-forced credentials for user bob using Hydra with the unix_passwords wordlist
Logged into the WebDAV share via cadaver and retrieved Flag 1 from the share directory
Used davtest to confirm .asp file execution is permitted; uploaded a pre-built ASP webshell from /usr/share/webshells/asp/ via WebDAV
Accessed the webshell through the browser and ran dir C:\ to locate and read flag2.txt from the C drive root


Target 2 - SMB (Flags 3 and 4):

Confirmed SMB on port 445 via Nmap; used Metasploit smb_login auxiliary module to brute-force credentials and recover the Administrator account password
Used exploit/windows/smb/psexec with recovered Administrator credentials to establish a Meterpreter session
Dropped to shell from Meterpreter; retrieved Flag 3 from C:\ root and Flag 4 from C:\Users\Administrator\Desktop


Impact: Demonstrates a full multi-target engagement across two different protocols and attack surfaces. Both flags chains required zero CVE exploitation. All access was gained through credential abuse and misconfiguration, which reflects the most common real-world attack path in enterprise environments.
