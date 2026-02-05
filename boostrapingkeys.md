# 🛡️ OpenPGP + YubiKey SSH & Crypto Runbook

> **Purpose:** This document describes a best‑practice workflow for generating, managing, and backing up OpenPGP keys with YubiKey hardware tokens. It covers secure machine preparation, key generation, backups, SSH integration, usage patterns, and record templates.

---

## 🧠 Secure Machine Preparation (Before Key Generation)

Before generating cryptographic keys, start with a **secure, clean system** so no residual data or malware can compromise key creation.

### Clean‑machine requirements

* Use a **fresh OS install** or a **securely wiped device**.
* Ideally disconnect networking during key generation.
* This machine should **never retain private keys** after backups are completed.

### Secure wipe guidance

**Windows**

* Settings → Recovery → Reset this PC → *Remove everything*
* Enable *clean data* (overwrites free space)

**macOS**

* System Settings → General → Transfer or Reset → *Erase All Content and Settings*

**Linux**

* Boot from trusted live media
* Use disk secure‑erase tools (e.g., `nwipe`, `blkdiscard`, or vendor SSD secure erase)

> **Why:** Simple file deletion leaves recoverable data (data remanence). Secure erase or crypto‑erase ensures key material cannot be recovered.

---

## 🧾 1. Generate Master OpenPGP Key (Offline)

The **master key** represents your cryptographic identity. It should be created once, backed up securely, and kept offline.

```bash
gpg --full-generate-key
```

* Use strong parameters (e.g., RSA 4096 or modern ECC)
* Set a strong passphrase

### Backup master key

```bash
gpg --export-secret-keys --armor <KEYID> > master-secret-backup.asc
```

Store this backup **offline only** (safe, encrypted removable media).

---

## 🔑 2. Create Subkeys

Create dedicated subkeys for day‑to‑day use:

| Subkey         | Purpose        |
| -------------- | -------------- |
| Authentication | SSH login      |
| Signing        | Git, documents |
| Encryption     | Files, email   |

```bash
gpg --expert --edit-key <KEYID>
# use addkey to create subkeys
```

---

## 💾 3. Back Up Subkeys

Before moving subkeys into hardware, back them up:

```bash
gpg --export-secret-subkeys --armor <KEYID> > subkeys-secret-backup.asc
```

Store this backup offline alongside the master key backup.

---

## 🪪 4. Load Subkeys to YubiKeys

Repeat the following for **each YubiKey** (primary and backup):

```bash
gpg --card-status
gpg --edit-key <KEYID>
# select each subkey and run keytocard
```

Subkeys are removed from disk once transferred to the YubiKey.

---

## 🛡️ 5. YubiKey Hardening

Enable touch confirmation:

```bash
ykman openpgp touch sig on
ykman openpgp touch aut on
ykman openpgp touch enc on
```

This requires physical presence for every cryptographic operation.

---

## 🔄 6. SSH Authentication Setup

### Configure gpg‑agent as SSH agent

```bash
export SSH_AUTH_SOCK=$(gpgconf --list-dirs agent-ssh-socket)
gpg-connect-agent updatestartuptty /bye
```

### Export SSH public key

```bash
gpg --export-ssh-key <KEYID> > yubikey_ssh.pub
```

Install it on servers:

```bash
ssh-copy-id -i yubikey_ssh.pub user@server
```

---

## ✉️ 7. GPG Operations

| Operation | Command                                      |
| --------- | -------------------------------------------- |
| Encrypt   | `gpg --encrypt --recipient <EMAIL> file.txt` |
| Decrypt   | `gpg --decrypt file.txt.gpg`                 |
| Sign      | `gpg --sign file.txt`                        |
| Verify    | `gpg --verify file.txt.sig`                  |

---

## 🔄 8. Backup & Testing Procedures

### Backup inventory

| Item           | Location       | Media          | Notes           |
| -------------- | -------------- | -------------- | --------------- |
| Master key     | Offline safe   | Encrypted USB  | Never networked |
| Subkeys        | Offline safe   | Encrypted USB  | Recovery only   |
| Backup YubiKey | Secure storage | Hardware token | Same subkeys    |

### Functional testing checklist

| Test      | Primary YubiKey | Backup YubiKey | Result |
| --------- | --------------- | -------------- | ------ |
| SSH login | ☐               | ☐              |        |
| Encrypt   | ☐               | ☐              |        |
| Decrypt   | ☐               | ☐              |        |
| Sign      | ☐               | ☐              |        |

### Scheduled testing log

| Date       | Tested by | Backup used | Result    | Notes |
| ---------- | --------- | ----------- | --------- | ----- |
| YYYY‑MM‑DD | Name      | Yes/No      | Pass/Fail |       |

---

## 🔄 9. Revocation & Rotation

### Revocation certificate

```bash
gpg --gen-revoke <KEYID> > revocation.asc
```

Store offline.

### Rotation schedule

| Subkey  | Expiration | Next rotation |
| ------- | ---------- | ------------- |
| Auth    | 1 year     | YYYY‑MM‑DD    |
| Sign    | 1 year     | YYYY‑MM‑DD    |
| Encrypt | 1 year     | YYYY‑MM‑DD    |

---

## 📁 Record‑Keeping Templates

### Key fingerprints

| Key            | Fingerprint | Created | Notes        |
| -------------- | ----------- | ------- | ------------ |
| Master         |             |         | Offline only |
| Auth subkey    |             |         | YubiKey      |
| Sign subkey    |             |         | YubiKey      |
| Encrypt subkey |             |         | YubiKey      |

### Backup locations

| Item           | Location | Access controls | Stored date |
| -------------- | -------- | --------------- | ----------- |
| Master key     |          |                 |             |
| Subkeys        |          |                 |             |
| Backup YubiKey |          |                 |             |

---

## 🧾 Change Log

| Date       | Change           | Author | Reason   |
| ---------- | ---------------- | ------ | -------- |
| YYYY‑MM‑DD | Initial creation | Name   | Baseline |

---

## 📌 Final Notes

* The generation workstation **must not retain keys** after backup.
* All long‑term secrets live only on YubiKeys and offline encrypted backups.
* This document should be reviewed annually.
