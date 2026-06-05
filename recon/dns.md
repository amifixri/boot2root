# DNS Enumeration

Tujuan:

Mengumpulkan informasi domain.

---

## Dig

```bash
dig TARGET
```

---

## NS Lookup

```bash
nslookup TARGET
```

---

## Zone Transfer

```bash
dig axfr @TARGET domain.com
```

Jika berhasil bisa mendapatkan:

- Subdomain
- Record Internal
- Infrastruktur DNS
