# Hack The Box — Nibbles

**Platform:** Hack The Box  
**Machine:** Nibbles (retired)  
**Difficulty:** Easy  
**Operating system:** Linux  
**Date attempted:** 3 March 2026

> **Scope:** This write-up documents a practice exercise against the retired Hack The Box machine *Nibbles*. IP addresses and other instance-specific details have been replaced with placeholders. Commands are reconstructed from my original notes; sample output is deliberately omitted where it was not recorded.

## Overview

The assessment followed a standard workflow: discover exposed services, enumerate the web application, identify an authenticated file-upload weakness, gain a shell, and investigate an overly permissive `sudo` configuration for privilege escalation.

**Attack path:** Service enumeration → Nibbleblog discovery → authenticated upload → PHP reverse shell → local privilege escalation.

## 1. Reconnaissance and service enumeration

I started by identifying open ports and the services running on the target.

```bash
nmap -sV --open -oN initial-scan.txt <TARGET_IP>
```

My notes identified two services:

| Service | Observed version |
| --- | --- |
| SSH | OpenSSH 7.2p2 (Ubuntu) |
| HTTP | Apache httpd 2.4.18 |

I also used a full TCP port scan and considered banner grabbing and targeted NSE scripts to gather more information:

```bash
nmap -p- --open -oN full-scan.txt <TARGET_IP>
nmap -sC -sV -p 22,80 -oN script-scan.txt <TARGET_IP>
nc -nv <TARGET_IP> 80
nmap -sV --script=http-enum -p 80 -oN http-enum.txt <TARGET_IP>
```

The HTTP service became the main focus because it exposed a web application that could be enumerated further.

## 2. Web application enumeration

I opened the site in a browser, inspected the page source and used `whatweb` to fingerprint the application.

```bash
whatweb http://<TARGET_IP>/
```

The site revealed a **Nibbleblog** installation under `/nibbleblog/`. I enumerated that path and checked publicly accessible application files for version information.

```bash
whatweb http://<TARGET_IP>/nibbleblog/

gobuster dir \
  -u http://<TARGET_IP>/nibbleblog/ \
  -w /usr/share/seclists/Discovery/Web-Content/common.txt

curl -i http://<TARGET_IP>/nibbleblog/README
```

The application appeared to be **Nibbleblog 4.0.3**. I researched known weaknesses affecting that version and identified an authenticated file-upload attack path. Because authentication was required, I also considered whether the login had protections against repeated attempts.

**Why this mattered:** If an authenticated administrator can upload a server-executable PHP file into a web-accessible location, a file-upload weakness can become remote code execution.

## 3. Initial foothold

I examined the Nibbleblog administration interface for an upload feature that could accept a PHP payload. My notes record the use of the administrator account and the password `nibbles` in the lab environment.

After uploading a PHP reverse-shell payload, I started a listener on my attacking machine and navigated to the uploaded resource to trigger execution on the server.

```bash
# On the attacking machine, before triggering the uploaded payload
nc -lvnp 4444
```

The exact payload, upload location and resulting terminal output were not preserved in my notes, so I have not reconstructed them here.

### Stabilising the shell

After obtaining command execution, I used a Python PTY to make the shell more usable:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

My notes also record the usual terminal-stabilisation sequence: suspend the shell with `Ctrl+Z`, run `stty raw -echo; fg` in the local terminal, then run `reset` and set `TERM` in the remote shell as needed.

```bash
# In the remote shell, after regaining foreground control
export TERM=xterm-256color
```

## 4. Privilege escalation

With an initial shell established, I investigated the local environment and available privileges. My notes mention automated Linux enumeration and identifying a script that could be executed with elevated privileges.

Useful checks in this stage include:

```bash
id
sudo -l
find /home -type f -name '*.sh' 2>/dev/null
```

I identified an opportunity to modify a script permitted to run through `sudo`. The key issue was that a lower-privileged user could influence the contents of a script executed with higher privileges. My notes describe backing up the script first, then appending commands rather than replacing the original contents.

I have not included an exact privileged script path or a final command transcript because those details were not recorded clearly enough to verify. The security lesson is the same: **allowing privileged execution of a user-writable script defeats the intended privilege boundary**.

## 5. Alternative route: Metasploit

As an alternative to manual exploitation, I investigated a Metasploit module targeting Nibbleblog 4.0.3. The lab configuration in my notes used the `admin` account, the password `nibbles`, the `/nibbleblog/` application path and a reverse connection to my attacking machine.

The core settings to review when reproducing this approach are:

- Target host (`RHOSTS`) and application path (`TARGETURI`)
- Administrator credentials
- Attacking host (`LHOST`) and listening port (`LPORT`)

I have kept this as an alternative approach rather than claiming a second successful compromise; my notes do not preserve its outcome.

## 6. Remediation and lessons learned

| Finding | Mitigation |
| --- | --- |
| Outdated web application | Maintain supported versions and apply security updates. |
| Unsafe authenticated uploads | Enforce strict file-type validation, store uploads outside executable web paths and disable script execution in upload directories. |
| Weak administrative access | Require strong, unique credentials and apply rate limiting or other login protections. |
| Privileged execution of modifiable scripts | Restrict `sudo` permissions and ensure privileged scripts and their parent directories cannot be modified by unprivileged users. |

### Key takeaways

- **Enumeration drives the assessment.** Service identification and web-content discovery were essential to locating Nibbleblog.
- **Authentication does not make upload functionality safe.** An authenticated upload can still lead to code execution when executable content is accepted and served insecurely.
- **Privilege escalation often depends on permissions, not a kernel exploit.** Reviewing `sudo` rights and file ownership revealed a path to elevated execution.
- **Keep reproducible evidence.** In future write-ups, I will preserve key command output, discovered paths and screenshots so that each conclusion can be independently followed.

---

*This write-up is for a retired, intentionally vulnerable training machine. Perform security testing only on systems you own or are authorised to assess.*
