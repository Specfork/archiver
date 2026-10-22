# Extract

**Status:** Draft · **Prefix:** XTR

## Purpose

Unpacking all or part of an archive to a folder, and what happens when files already exist.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| XTR-1 | An extract job takes an archive (or any volume of a split one), a selection (all, or browser items, kept relative to the folder shown) and options `{destination, path_mode, overwrite, password, flatten_single_root, keep_broken, restore_times, origin_mark, extract_links, allow_unsafe_links}`. | in `src/`, extract `lib/a.c` → `<dest>/lib/a.c` |
| XTR-2 | Default `destination`: the archive's folder + its base name with every archive extension stripped (`a.tar.gz` → `a`, `a.7z.001` → `a`). Missing folders, parents included, are created. | `photos.zip` in `Downloads/` → `Downloads/photos/` |
| XTR-3 | `flatten_single_root` (default on): when the top level holds exactly one folder named like the destination's last component, its contents go straight into the destination. | `proj.zip` with only `proj/…` → `proj/file`, not `proj/proj/file` |
| XTR-4 | `path_mode`: `full` (default; keep stored folders), `none` (all files into the destination), `absolute` (keep a stored root; needs a confirmation per job and stays subject to [SAFE-3](safety.md)). | `none` on `a/b/c.txt` → `<dest>/c.txt` |
| XTR-5 | `overwrite`: `ask` (default), `overwrite`, `skip`, `rename_new`, `rename_existing`. | `skip` with existing file → untouched, counted skipped |
| XTR-6 | The `ask` prompt shows both files' name, size and modified time, and offers `yes`, `yes_to_all`, `no`, `no_to_all`, `rename`, `cancel`; `*_to_all` lasts for this job only. | `no_to_all` → no more prompts, rest skipped |
| XTR-7 | Renaming picks the first free `name (1).ext`, `name (2).ext`, …; `rename_existing` renames the file on disk and writes the new one under the original name. | existing `a.txt`, `a (1).txt` → new file `a (2).txt` |
| XTR-8 | Items are written under a temporary name and renamed on success. On error or cancel the partial file is removed unless `keep_broken` (default off); an item failing its checksum is reported `crc_failed` and kept only with `keep_broken`. | cancel mid-file → no partial file; flipped byte → error, file absent |
| XTR-9 | Split archives: opening any volume finds siblings by name; a missing volume pauses the job with a prompt to locate it or cancel, naming the expected file. | delete `.002` → prompt names `a.7z.002` |
| XTR-10 | When the known unpacked total exceeds free space on the destination volume, a warning shows both numbers before writing; the user continues or cancels. | 2 GiB archive, 1 GiB free → warning |
| XTR-11 | `restore_times` (default on) sets modified time, plus created/accessed when stored and supported; folder times are set after their contents. POSIX modes (minus [SAFE-8](safety.md) bits) apply on POSIX systems, Windows attributes on Windows. | extracted mtime = stored mtime |
| XTR-12 | `origin_mark` (`all`, `documents_only`, `off`; default `all`): when the archive carries the OS's downloaded-from-the-internet mark, extracted files get it too; `documents_only` limits it to office documents and scripts. | marked archive → extracted `.docx` marked |
| XTR-13 | Several archives selected together extract each into its own default destination, one after another, as one job with a combined summary. | 3 zips → 3 folders, one summary |

## Out of scope

- Restoring owners, ACLs or extended attributes other than the origin mark.
- Extracting to remote or cloud locations.

## Decisions

- Rename uses `name (n).ext`, the convention most file managers use; the source uses another suffix.
- Write-then-rename (XTR-8) means a crash never leaves a half file under the real name.
- `flatten_single_root` is a default, not a menu entry, so extracting to a folder never doubles it.
