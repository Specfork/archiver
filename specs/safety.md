# Extraction safety

**Status:** Draft · **Prefix:** SAFE

## Purpose

An archive from anywhere can never write outside the chosen folder or plant something dangerous.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| SAFE-1 | Every stored path is normalised before use: `\` and `/` are both separators, empty and `.` components are dropped, and `..` cancels the previous component. | `a/./b//c` → `a/b/c`; `a/x/../b` → `a/b` |
| SAFE-2 | An item whose normalised path climbs above the destination (a `..` with nothing left to cancel) is not written; it is reported `unsafe_path` and the job ends `completed_with_errors`. The check also covers codec paths ([FMT-5](formats.md)), `path_mode: none` and renamed names. | item `../../etc/x` → nothing outside, `unsafe_path` |
| SAFE-3 | Absolute stored paths (leading separator, drive letter `C:`, UNC `\\host\share`, device prefix `\\?\`) lose their root and land under the destination, with one warning per job; only `path_mode: absolute` with its confirmation ([XTR-4](extract.md)) keeps the root. | item `/etc/passwd` → `<dest>/etc/passwd` |
| SAFE-4 | Symbolic links are created when `extract_links` is on (default on where the user may create links); where the OS forbids it, the link becomes a small text file holding its target, with a warning. | Windows without link rights → text file + warning |
| SAFE-5 | A link whose target is absolute, or resolves outside the destination from the link's folder, is not created (`unsafe_link`) unless `allow_unsafe_links` (default off) is on for this job. | link `x → /etc` → skipped, `unsafe_link` |
| SAFE-6 | Links are created only after every regular file of the job is written, so no file of the same archive is written through a link it created. | link `d → /tmp`, then file `d/x` → `d/x` is a real file in `<dest>` |
| SAFE-7 | If a component of a file's target path inside the destination is an existing link pointing outside it, the file is not written (`unsafe_path`). | existing `<dest>/d → /tmp`, item `d/x` → refused |
| SAFE-8 | Setuid, setgid and sticky bits are never restored; file owners are never changed. | item mode 4755 → extracted 0755 |
| SAFE-9 | Hard-link items are created only when their target is an item this job already wrote inside the destination; otherwise `unsafe_link`. | hard link to `/etc/shadow` → skipped |
| SAFE-10 | Device files, FIFOs and sockets are never created; each is listed as skipped. | tar with a character device → skipped |
| SAFE-11 | When an item decodes to more than its declared size, decoding stops at that size and the item is `data_error`; total output never exceeds the sum of declared sizes. | entry declaring 1 KiB, yielding 1 GiB → stops, error |
| SAFE-12 | Extraction never runs or opens what it writes; nested archives stay files. Opening an executable, script or shortcut item from the browser ([BRW-6](browse.md)) asks for confirmation first. | open `setup.exe` from archive → confirmation |

## Out of scope

- Malware scanning; antivirus integration.

## Decisions

- Unsafe items are skipped, not renamed into the destination: a renamed traversal payload would still surprise.
- Links last (SAFE-6) plus the existing-link check (SAFE-7) close the link-then-write escape.
- Without link rights, a text file with the target beats silently dropping the item.
