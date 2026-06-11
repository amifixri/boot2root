# SMBClient Cheat Sheet

SMBClient adalah tool untuk mengakses SMB/CIFS share pada Windows maupun Samba Linux. Sangat sering digunakan dalam CTF untuk enumerasi file share, mencari credential, backup, dan flag.

---

## Cek Versi

```bash
smbclient --version
```

---

## Enumerasi Share

### Anonymous Login

```bash
smbclient -L //IP -N
```

Contoh:

```bash
smbclient -L //10.10.10.10 -N
```

Keterangan:

- `-L` : Menampilkan daftar share.
- `-N` : Tidak menggunakan password.

---

### Login Menggunakan User

```bash
smbclient -L //IP -U username
```

Atau:

```bash
smbclient -L //IP -U username%password
```

Contoh:

```bash
smbclient -L //10.10.10.10 -U administrator
```

---

## Masuk ke Share

### Anonymous

```bash
smbclient //IP/share -N
```

Contoh:

```bash
smbclient //10.10.10.10/Public -N
```

---

### Dengan Credential

```bash
smbclient //IP/share -U username
```

Contoh:

```bash
smbclient //10.10.10.10/Users -U administrator
```

Jika berhasil akan muncul:

```text
smb: \>
```

---

# Command Dasar SMBClient

## Melihat Isi Folder

```bash
ls
```

---

## Pindah Direktori

```bash
cd folder
```

Contoh:

```bash
cd backup
```

---

## Menampilkan Direktori Saat Ini

```bash
pwd
```

---

## Download File

```bash
get file.txt
```

Contoh:

```bash
get flag.txt
```

---

## Download Banyak File

```bash
mget *
```

---

## Upload File

```bash
put shell.php
```

---

## Upload Banyak File

```bash
mput *
```

---

## Membuat Folder

```bash
mkdir test
```

---

## Menghapus File

```bash
del file.txt
```

---

## Menghapus Folder

```bash
rmdir folder
```

---

# Download Semua Isi Share

Masuk ke share terlebih dahulu:

```bash
smbclient //IP/share -N
```

Kemudian:

```bash
recurse ON
prompt OFF
mget *
```

Keterangan:

- `recurse ON` : Download seluruh subfolder.
- `prompt OFF` : Tidak meminta konfirmasi.

---

# Menentukan SMB Version

### SMBv2

```bash
smbclient //IP/share -m SMB2
```

### SMBv3

```bash
smbclient //IP/share -m SMB3
```

---

# Login Dengan Domain

```bash
smbclient //IP/share -U DOMAIN\\user
```

Atau:

```bash
smbclient //IP/share -U DOMAIN/user
```

Contoh:

```bash
smbclient //10.10.10.10/Users -U WORKGROUP\\administrator
```

---

# Workflow CTF

## 1. Enumerasi Share

```bash
smbclient -L //IP -N
```

Output:

```text
Sharename       Type
---------       ----
IPC$            IPC
Public          Disk
Backup          Disk
```

---

## 2. Masuk ke Share

```bash
smbclient //IP/Public -N
```

---

## 3. Lihat Isi Folder

```bash
ls
```

---

## 4. Download File Menarik

```bash
get backup.zip
get passwords.txt
get flag.txt
```

---

## 5. Download Semua Isi Share

```bash
recurse ON
prompt OFF
mget *
```

---

# Tools Pendukung SMB Enumeration

## Nmap SMB Scripts

```bash
nmap -p445 --script smb-enum-shares IP
```

```bash
nmap -p445 --script smb-enum-users IP
```

```bash
nmap -p445 --script smb-os-discovery IP
```

---

## Enum4linux

```bash
enum4linux IP
```

---

## Enum4linux-ng

```bash
enum4linux-ng IP
```

---

## SMBMap

```bash
smbmap -H IP
```

---

## CrackMapExec

```bash
crackmapexec smb IP
```

---

# Checklist SMB CTF

- [ ] Scan port 445
- [ ] Enumerasi share dengan smbclient
- [ ] Coba anonymous login
- [ ] Cari file backup
- [ ] Cari credential
- [ ] Cari file konfigurasi
- [ ] Download seluruh share
- [ ] Periksa file ZIP
- [ ] Periksa dokumen Office
- [ ] Cari flag

---

# File yang Sering Menyimpan Flag atau Credential

```text
flag.txt
passwords.txt
creds.txt
backup.zip
config.ini
web.config
database.db
users.csv
notes.txt
id_rsa
authorized_keys
```

---

# Ringkasan Cepat

```bash
# Enumerasi share
smbclient -L //IP -N

# Masuk share
smbclient //IP/share -N

# Lihat file
ls

# Download file
get flag.txt

# Download semua
recurse ON
prompt OFF
mget *

# Enumerasi lanjutan
enum4linux-ng IP
smbmap -H IP
crackmapexec smb IP
```

> Tips CTF: Jika menemukan port **445/tcp**, selalu cek anonymous share terlebih dahulu. Banyak challenge SMB menyimpan flag, backup, credential, atau file konfigurasi yang dapat digunakan untuk pivoting ke service lain.
