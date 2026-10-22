# Formats

**Status:** Draft · **Prefix:** FMT

## Purpose

Which archive formats exist, what each can do, and the codec contract that adds more.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| FMT-1 | Every format is a registry entry `{id, extensions[], signatures[], caps}`; `caps` is a set of `create`, `extract`, `update`, `multi_file`, `solid`, `encrypt`, `encrypt_names`, `links`. Every command and dialog offers only what `caps` allows. | format `tar` chosen → password field disabled |
| FMT-2 | Read-write formats and caps: `7z` {create, extract, update, multi_file, solid, encrypt, encrypt_names, links}; `zip` {create, extract, update, multi_file, encrypt, links}; `tar` {create, extract, update, multi_file, links}; `gzip`, `bzip2`, `xz` {create, extract}. Output opens in other common tools for the same format. | manual: sample 7z, zip, tar.gz open in two other archivers |
| FMT-3 | `gzip`, `bzip2` and `xz` hold one stream: packing one file makes `<file>.gz`/`.bz2`/`.xz`; packing several items or a folder offers only `tar.gz`, `tar.bz2`, `tar.xz`, built in one pass. Opening `.tar.gz`, `.tgz`, `.tar.bz2`, `.tbz2`, `.tar.xz`, `.txz` goes straight to the tar listing. | 2 files + gzip → only `.tar.gz` offered; open `a.tgz` → tar entries |
| FMT-4 | Read-only formats (caps {extract, multi_file}): `rar`, `cab`, `iso`, `udf`, `wim`, `cpio`, `rpm`, `ar`/`deb`, `dmg`, `xar`, `lzh`, `arj`, `chm`, `msi`, `nsis`, `squashfs`, `cramfs`, `vhd`, `vhdx`, `vmdk`, `vdi`, `qcow2`, `fat`, `ntfs`, `ext`, `hfs`, `apfs`; single-stream read-only: `lzma`, `z`, `zstd`. Each is optional; the about view lists those built in. | open `.iso` → list and extract work, add disabled |
| FMT-5 | Codec contract: a read-only format is a module exposing `probe(first_bytes) → confidence 0..100`, `open(stream, volume_provider, password_provider) → archive`, `archive.items() → [{path, is_dir, size, packed_size?, mtime?, attributes?, crc32?, encrypted, link_target?}]`, `archive.read(item) → byte stream`, `archive.properties() → map`. A codec reports errors only as [INT-3](integrity.md) kinds and never writes to disk, so every [SAFE](safety.md) rule applies to it. | stub codec with items `a`, `../x` → `a` extracted, `../x` blocked |
| FMT-6 | Detection: the signature (at the start, or at the end for formats that keep it there) wins over the extension; on disagreement the archive opens as the detected format with a warning naming it; "open as" forces a format. No match → error `unknown_format`, never an empty archive. | `photo.zip` that is 7z → opens as 7z + warning; random bytes → `unknown_format` |
| FMT-7 | Methods: `7z`: `lzma2` (default), `lzma`, `ppmd`, `bzip2`, `deflate`, `deflate64`, `copy`; `zip`: `deflate` (default), `deflate64`, `bzip2`, `lzma`, `ppmd`, `store`; `gzip`: `deflate`; `bzip2`: `bzip2`; `xz`: `lzma2`; `tar`: no compression, layout `pax` (default) or `gnu`. | 7z defaults → method `lzma2` |
| FMT-8 | Levels: `0` store, `1` fastest, `3` fast, `5` normal (default), `7` maximum, `9` ultra. Allowed: `7z` 0–9; `zip` 0, 1, 3, 5, 7, 9; `gzip` 1, 5, 7, 9; `bzip2` 1, 3, 5, 7, 9; `xz` 1–9; `tar` 0. Another level is rejected before work starts. Level 0 stores data uncompressed. | gzip level 3 → rejected, nothing written; 10 MiB random at level 0 → archive ≥ 10 MiB |
| FMT-9 | Advanced parameters `dictionary_size`, `word_size`, `solid_block` ([CRT-6](create.md)), `threads` default from the level and may be overridden; the dialog shows estimated memory for packing and for unpacking. | dictionary 16 → 64 MiB → both estimates rise |
| FMT-10 | `zip` switches to 64-bit fields when any item or the archive exceeds 4 GiB − 1 B or holds more than 65,535 items; `7z` and `tar` (`pax`) hold items up to 2^63 B. | 5 GiB file zipped → extracts with equal hash |
| FMT-11 | Times: modified time is stored in every format; `7z` adds created and accessed time when `store_times` includes them (default: modified only); `zip` stores its 2-second local time plus an exact UTC time. | zip file modified 10:00:01 → extracted 10:00:01 |
| FMT-12 | Attributes: POSIX mode is stored in `tar`, `zip`, `7z`; Windows `read_only`, `hidden`, `system`, `archive` in `zip`, `7z`; each is restored only where the OS has it ([XTR-11](extract.md)). | mode 0755 packed on Linux → 0755 on macOS |
| FMT-13 | Volumes: any created archive may be split into raw slices `<archive>.001`, `.002`, … (3 digits; 4 past 999); concatenating them gives the unsplit archive. | 25 MiB archive, 10 MiB volumes → 3 files; concatenation hash = unsplit hash |

## Out of scope

- Creating any read-only format, including `rar` and `wim`.
- Self-extracting executables; format-native zip spanning (`.z01`).
- Executable pre-filters and other per-method tuning as user options; benchmarks.

## Decisions

- RAR is extract-only and only through the codec contract (FMT-5). The source's RAR decoder ships under a separate unRAR licence that forbids using it to re-create the RAR compressor; this spec was written without reading that code. Implementers who bundle a RAR codec must honour that licence or use a decoder under a compatible one, and may simply omit `rar`.
- Source licence: most of 7-Zip is LGPL-2.1-or-later; two decoders are BSD-3-Clause and one hash is BSD-2-Clause. `derived_from.license` records the main licence.
- `tar` defaults to `pax` (the source defaults to `gnu`): `pax` stores UTF-8 names, long paths and exact times portably.
- The read-only list is a menu, not a mandate: an implementation is complete with the six read-write formats plus any codecs it picks.
