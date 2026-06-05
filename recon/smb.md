# SMB Enumeration

Tujuan:

Mencari share yang dapat diakses.

---

## List Share

```bash
smbclient -L //TARGET -N
```

---

## Akses Share

```bash
smbclient //TARGET/share -N
```

---

## Enum4linux

```bash
enum4linux-ng TARGET
```

Mencari:

- User
- Share
- SID
- Policy

---

## CrackMapExec

```bash
crackmapexec smb TARGET
```

Mencari informasi SMB.
