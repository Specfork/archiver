# Create and update

**Status:** Draft · **Prefix:** CRT

## Purpose

Packing files and folders into a new archive, or changing an existing one.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| CRT-1 | A create job takes inputs (one or more files or folders) and options `{archive_path, format, level, method, dictionary_size, word_size, solid_block, threads, memory_limit, update_mode, path_mode, volume_sizes[], password, encrypt_names, excludes[], links, delete_after, test_after}`; unset options take this spec's defaults. | only inputs given → `7z`, level 5, `lzma2` |
| CRT-2 | Default `archive_path`: the first input's folder + a base name (one file: its name without the last extension; one folder: its name; several items: their parent folder's name) + the format extension. A typed name without the extension gets it; changing format swaps it. | `report.txt` → `report.7z`; `a.txt`+`b.txt` in `docs/` → `docs/docs.7z`; switch to zip → `docs.zip` |
| CRT-3 | Folders are added recursively with every file, hidden ones included, and with empty folders as folder entries. | folder with empty `sub/` → archive lists `sub/` |
| CRT-4 | `path_mode`: `relative` (default; paths start at each input's own name), `full` (from the volume root, without drive letter or leading separator), `absolute` (with root; `7z` and `tar` only). | `/home/u/docs/a.txt`: relative → `a.txt`; full → `home/u/docs/a.txt` |
| CRT-5 | `excludes`: wildcards (`*` any run, `?` one character) matched against each name, or against the relative path when the pattern has a separator; a matching folder is skipped with its contents. The archive itself, its volumes and its temporary file are never added. | exclude `*.tmp` → no `.tmp`; pack `x` into `x/x.7z` → no `x.7z` inside |
| CRT-6 | `solid_block` (`7z`, `xz`): `off`, 1 MiB–64 GiB, or `full`; default from level (≥ 64 MiB at level 5). Files in a block are ordered by extension, then name. The dialog's help says that reading or replacing one file touches its whole block. | 1,000 small text files → solid archive smaller than `off` |
| CRT-7 | If `archive_path` exists in the same format, the job updates it per `update_mode`; another format fails with `format_mismatch`; a split archive fails with `volumes_read_only`. Nothing changes on failure. | add to `a.7z` as zip → `format_mismatch`; add to `a.7z.001` → `volumes_read_only` |
| CRT-8 | `update_mode`: `add_replace` (default: add all, replace same paths), `update_add` (add new, replace only where the disk copy is newer), `freshen` (replace existing paths where newer, add nothing), `sync` (`update_add` plus remove archive items absent from the inputs). "Newer" compares modified times at the precision both sides keep; equal is not newer. | `freshen` with a new file → not added; same file `update_add` → not rewritten |
| CRT-9 | `sync` that would remove ≥ 1 item shows the count and the first 20 paths and needs explicit confirmation; declining cancels the whole job. | sync removing 3 → confirm lists 3; No → archive unchanged |
| CRT-10 | Updating writes a new temporary archive beside the old one and replaces it only on success; on error or cancel the original stays byte-identical. | cancel at 50% → original hash unchanged |
| CRT-11 | `volume_sizes`: sizes as number + unit `B`, `K`, `M`, `G` (binary); each volume takes the next size and the last repeats; minimum 64 KiB. Presets: 10M, 100M, 650M, 700M, 4092M, 4480M. | `10M` on 25 MiB → volumes 10, 10, 5 MiB |
| CRT-12 | Unreadable or locked inputs are skipped, listed in the job messages, and the job ends `completed_with_warnings` ([JOB-5](jobs.md)). | 1 locked file of 5 → archive has 4, warning names it |
| CRT-13 | `links`: `store_link` (default where the format has `links`; the link itself is stored) or `follow` (store target content, loops stored once with a warning); formats without `links` always follow. `tar` stores the second and later paths of one hard-linked file as hard-link entries. | link to `a.txt` in tar → link entry; 2 hard links of 1 MiB → tar ≈ 1 MiB |
| CRT-14 | `delete_after` (default off) removes inputs only after the archive is complete and, when `test_after` (default off) is on, tested `ok` ([INT-1](integrity.md)); skipped inputs are never removed. A failed test ends the job `failed` and keeps the archive. | 1 input skipped → that input still on disk |
| CRT-15 | Turning `delete_after` on needs a confirmation naming the item count and that removal is permanent where no trash exists; on the command line it needs an explicit switch. Inputs go to the system trash when one exists. | enable → confirm shows "5 items" before start |
| CRT-16 | `threads`: 1 to logical CPU count, default all; `memory_limit` default 80% of RAM. While the packing estimate exceeds the limit, start is blocked showing both numbers. | dictionary 1.5 GiB on 4 GiB RAM → start blocked |

## Out of scope

- Archive comments, rename-on-add rules, scheduled backups.

## Decisions

- A single file's base name drops its last extension: `photo.jpg` → `photo.7z`.
- `delete_after` prefers the trash; the source deletes permanently.
- Minimum volume size 64 KiB: smaller slices mostly create thousands of files by mistake.
