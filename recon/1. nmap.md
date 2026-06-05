# Nmap

## Scan Semua Port

```bash
nmap -p- TARGET
```

Fungsi:
Mencari seluruh port terbuka.

---

## Scan Detail

```bash
nmap -sC -sV TARGET
```

Fungsi:
- Deteksi service
- Deteksi versi
- Menjalankan default NSE scripts

---

## Scan Agresif

```bash
nmap -A TARGET
```

Fungsi:
- OS Detection
- Version Detection
- NSE Scripts
- Traceroute
