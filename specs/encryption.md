# Encryption

**Status:** Draft · **Prefix:** ENC

## Purpose

Password-protecting archive contents and, in 7z, the list of names.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| ENC-1 | `7z` encryption is AES-256 in CBC mode; the key derives from the UTF-16 password and a random salt by iterated SHA-256 (2^19 rounds); every archive gets a fresh random IV. | same files and password twice → different bytes, both open |
| ENC-2 | `zip` encryption: `aes256` (default; the common AES zip extension) or `legacy` (traditional zip encryption), which shows a weakness warning before start. | choose `legacy` → warning |
| ENC-3 | Password: 1–1024 characters, any Unicode; empty means none. Masked by default with a `show_password` toggle; while masked, a second entry must match before start. Formats without `encrypt` ([FMT-1](formats.md)) disable the fields. | masked entries differ → start blocked |
| ENC-4 | `encrypt_names` (`7z` only; default off) also encrypts the header: without the password nobody sees names, sizes or count, and opening asks for the password before listing. With `zip`, the dialog notes that names and sizes stay visible. | header-encrypted 7z → prompt before listing; zip + password → note shown |
| ENC-5 | A wrong password gives `wrong_password` (when the format can tell) or `data_error` with a hint that the password may be wrong; nothing partial stays ([XTR-8](extract.md)); the prompt returns, up to 3 attempts per job. | wrong password → error, prompt again, nothing written |
| ENC-6 | Updating an encrypted `7z`: new items use this job's password; kept items stay as they are; a header-encrypted archive needs its password before any update. | add to header-encrypted archive → password asked first |
| ENC-7 | Encrypted items carry a marker in the browser. Zip archives may mix plain and encrypted items; the password is asked once per job, only when an encrypted item is read, and again only for an item it fails on. | mixed zip → plain item extracts with no prompt; 100 encrypted items → one prompt |
| ENC-8 | Passwords live only in memory for one job; they never reach settings, history, logs or temporary files, and are wiped when the job ends. | search settings and logs after job → absent |

## Out of scope

- Public-key encryption, key files, hardware tokens, saved passwords, password recovery.

## Decisions

- No saved passwords (ENC-8): an archive password stored on the same disk adds little and leaks easily.
- `aes256` is the zip default; `legacy` stays for old receivers only.
