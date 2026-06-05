# HTTP Enumeration

Tujuan:

Mengumpulkan informasi website.

---

## WhatWeb

```bash
whatweb http://TARGET
```

Fungsi:

Mengetahui:

- CMS
- Framework
- Web Server
- PHP Version

---

## Curl

### Header

```bash
curl -I http://TARGET
```

Fungsi:

Melihat:

- Server
- Redirect
- Cookie

---

### Source Code

```bash
curl http://TARGET
```

Menganalisis source website.

---

## Nikto

```bash
nikto -h http://TARGET
```

Fungsi:

Mencari:

- Misconfiguration
- Directory listing
- Backup file
- Default file

---

## FFUF

### Directory Enumeration

```bash
ffuf -u http://TARGET/FUZZ -w wordlist.txt
```

Mencari:

```text
/admin
/backup
/uploads
/dev
```

---

### File Enumeration

```bash
ffuf -u http://TARGET/FUZZ.php -w wordlist.txt
```

Mencari file tersembunyi.
