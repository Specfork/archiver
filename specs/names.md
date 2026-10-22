# Names

**Status:** Draft · **Prefix:** NAM

## Purpose

File names in any script survive packing and unpacking on every OS.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| NAM-1 | Names are Unicode internally. `7z` stores UTF-16; `tar` (`pax`) and `zip` store UTF-8, and `zip` sets its UTF-8 flag on each item with a non-ASCII name. Stored paths use `/`; a `\` from a Windows tool counts as a separator ([SAFE-1](safety.md)). | zip `résumé.txt` → UTF-8 flag set; zip `a\b.txt` → folder `a`, file `b.txt` |
| NAM-2 | Zip items without the UTF-8 flag decode with `legacy_code_page` (default: the OS's legacy console code page on Windows, CP437 elsewhere); the browser lets the user pick another and relists at once. | old Cyrillic zip garbled → pick CP866 → names correct |
| NAM-3 | Bytes invalid in a name's encoding show as U+FFFD, are written to disk as `_`, and the rename is listed; the data still extracts. | invalid byte in tar name → `a_b.txt` + rename listed |
| NAM-4 | Characters the destination OS forbids (Windows: `<>:"\|?*` and control characters) become `_`, and trailing dots and spaces are removed; each change is listed. | `a:b.txt` on Windows → `a_b.txt` |
| NAM-5 | Names reserved by the destination OS (Windows: `CON`, `PRN`, `AUX`, `NUL`, `COM1`–`COM9`, `LPT1`–`LPT9`, any extension) get a `_` prefix. | `NUL.txt` on Windows → `_NUL.txt` |
| NAM-6 | Names differing only by case, or only by Unicode normalisation (NFC vs NFD), collide on a destination that treats them as equal; the second follows `overwrite` ([XTR-5](extract.md)). | `Read.me` + `READ.ME` on a case-insensitive disk → prompt on the second |
| NAM-7 | A component longer than the destination allows (typically 255 bytes), or a path longer than the OS limit after any long-path support, fails that item with `name_too_long`; the job continues. | 300-byte name → `name_too_long`, others extracted |

## Out of scope

- Transliterating names.

## Decisions

- Prefixing reserved names keeps them readable and easy to fix by hand.
