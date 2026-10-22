# Integrity

**Status:** Draft · **Prefix:** INT

## Purpose

Checking that an archive decodes cleanly, and computing checksums of plain files.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| INT-1 | Test decodes every selected item (default all) without writing to disk and checks each stored checksum (CRC-32 in zip, 7z and gzip; block CRCs in bzip2; the stream check in xz). | good archive → `ok`, nothing written |
| INT-2 | Result: `ok`, or a list of `{item_path, error}`, with totals of files, folders, bytes tested and errors. | 2 corrupt items → list of 2 |
| INT-3 | Error kinds shared by test, extract and codecs: `crc_failed`, `data_error`, `unexpected_end`, `unsupported_method`, `wrong_password`, `headers_error`, `missing_volume`, `unavailable_data`, `data_after_end`. `data_after_end` (bytes after a complete archive) is a warning; the result stays `ok`. | truncated → `unexpected_end`; 100 bytes appended → `ok` + warning |
| INT-4 | Encrypted archives need the password to test ([ENC-5](encryption.md)); a header-encrypted 7z cannot be tested without it. | test without password → prompt |
| INT-5 | Checksums of plain files and folders: `crc32`, `crc64`, `xxh64`, `md5`, `sha1`, `sha256` (default), `sha384`, `sha512`, `blake2sp`. Output lists each file's value; for several files also one value over all data and one over data and names. Copy gives one `value  path` line per file. | `sha256` of an empty file → `e3b0c442…b855` |
| INT-6 | Testing a 1 GiB `7z` (`lzma2`, level 5) takes at most 1.2 × the time to extract it to a fast local disk. | manual: time test vs extract |

## Out of scope

- Repairing damaged archives; recovery records; digital signatures.

## Decisions

- One error enum (INT-3) for test, extract and codecs keeps messages and exit codes ([SHL-9](integration.md)) consistent.
