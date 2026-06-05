# SSH Enumeration

Tujuan:

Mengumpulkan informasi SSH.

---

## Banner

```bash
nmap -sV -p22 TARGET
```

---

## Fingerprint

```bash
ssh-keyscan TARGET
```

---

Yang Dicari

- Versi OpenSSH
- User yang valid
- Private key bocor
