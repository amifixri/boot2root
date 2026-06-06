# Earth Machine Walkthrough

## Enumeration

### Host Discovery

#### Tujuan

Mengidentifikasi host aktif dalam jaringan lokal.

#### Metodologi

```bash
arp-scan -l
```

#### Hasil

Ditemukan alamat IP target yang aktif pada jaringan.

#### Analisis

Tahap ini digunakan untuk menemukan target sebelum melakukan enumerasi lebih lanjut terhadap service yang tersedia.

---

### Service Enumeration

#### Tujuan

Mengidentifikasi port terbuka, service yang berjalan, serta informasi versi.

#### Metodologi

```bash
nmap -sC -sV -A <TARGET_IP>
```

#### Hasil

Ditemukan beberapa service yang berjalan serta informasi domain internal:

```text
earth.local
terratest.earth.local
```

#### Analisis

Adanya virtual host mengindikasikan kemungkinan terdapat beberapa aplikasi web yang berjalan pada server yang sama.

---

### Virtual Host Configuration

#### Tujuan

Mengakses domain internal yang ditemukan selama scanning.

#### Metodologi

Menambahkan domain ke file hosts.

```bash
sudo nano /etc/hosts
```

```text
<TARGET_IP> earth.local
<TARGET_IP> terratest.earth.local
```

#### Hasil

Kedua domain berhasil diakses melalui browser.

---

### Web Enumeration

#### earth.local

Mengakses:

```text
http://earth.local
```

#### Hasil

Website menampilkan beberapa pesan terenkripsi.

#### Analisis

Keberadaan ciphertext menunjukkan kemungkinan terdapat mekanisme komunikasi internal yang dapat dianalisis lebih lanjut.

---

#### terratest.earth.local

Mengakses:

```text
http://terratest.earth.local
```

#### Hasil

Website hanya menampilkan halaman sederhana bertuliskan:

```text
Test Site
```

#### Analisis

Walaupun terlihat minim informasi, subdomain testing sering kali menyimpan konfigurasi atau file yang tidak tersedia pada production environment.

---

### Directory Enumeration

#### Tujuan

Mencari direktori dan file tersembunyi.

#### Metodologi

```bash
gobuster dir \
-u http://earth.local \
-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

#### Hasil

Ditemukan:

```text
/admin
```

#### Analisis

Endpoint tersebut menampilkan halaman login administrator dan menjadi target utama untuk tahap berikutnya.

---

### Robots.txt Analysis

#### Tujuan

Mencari informasi yang tidak ditautkan langsung dari website.

#### Metodologi

Mengakses:

```text
http://terratest.earth.local/robots.txt
```

#### Hasil

```text
Disallow: /testingnotes.*
```

#### Analisis

File robots.txt sering kali membocorkan direktori yang dianggap sensitif oleh pengembang.

---

### Sensitive Information Disclosure

#### Tujuan

Menganalisis file yang ditemukan pada robots.txt.

#### Hasil

Ditemukan catatan internal:

```text
Using XOR encryption as the algorithm.
testdata.txt was used to test encryption.
terra used as username for admin portal.
```

#### Informasi Penting

| Data           | Nilai        |
| -------------- | ------------ |
| Username       | terra        |
| Encryption     | XOR          |
| Reference File | testdata.txt |

#### Analisis

Developer secara tidak sengaja membocorkan informasi yang dapat digunakan untuk memperoleh kredensial administrator.

---

## Exploitation

### Credential Recovery

#### Tujuan

Memulihkan password administrator.

#### Metodologi

1. Mengambil ciphertext yang tersedia pada earth.local.
2. Mengambil isi testdata.txt.
3. Menganalisis implementasi XOR yang digunakan.
4. Mendapatkan plaintext hasil dekripsi.

#### Hasil

Berhasil memperoleh password yang valid untuk akun:

```text
Username: terra
Password: <decrypted_password>
```

#### Analisis

Penggunaan XOR dengan key yang dapat ditebak menyebabkan kerahasiaan data gagal dipertahankan.

---

### Admin Portal Access

#### Tujuan

Mendapatkan akses administratif.

#### Metodologi

Login menggunakan kredensial yang diperoleh pada tahap sebelumnya.

#### Hasil

Berhasil masuk ke panel administrator.

#### Analisis

Akses administratif memberikan area serangan yang lebih luas dibandingkan halaman publik.

---

### Remote Command Execution

#### Tujuan

Menguji kemampuan panel administrator.

#### Metodologi

Menggunakan fitur yang tersedia untuk menjalankan perintah sistem.

Contoh:

```bash
whoami
pwd
ls
cat file.txt
```

#### Hasil

Perintah sistem berhasil dijalankan.

#### Analisis

Panel administrator rentan terhadap Remote Command Execution (RCE).

Status akses saat ini:

```text
User Access Obtained
Command Execution Confirmed
Privilege Level: Low Privileged User
```

---

## Post Exploitation

### Linux Enumeration

Tahap berikutnya setelah memperoleh shell:

```bash
whoami
id
hostname
ip a
ss -tulnp
sudo -l
find / -perm -4000 2>/dev/null
find / -writable -type d 2>/dev/null
crontab -l
```

#### Tujuan

Mencari jalur privilege escalation menuju root.

---

## Attack Path Summary

```text
ARP Scan
   │
   ▼
Nmap Enumeration
   │
   ▼
Virtual Host Discovery
   │
   ▼
Directory Enumeration
   │
   ▼
robots.txt Disclosure
   │
   ▼
testingnotes Disclosure
   │
   ▼
Username Discovery
   │
   ▼
XOR Analysis
   │
   ▼
Password Recovery
   │
   ▼
Admin Login
   │
   ▼
Remote Command Execution
   │
   ▼
Linux Enumeration
   │
   ▼
Privilege Escalation
   │
   ▼
Root Access
```
