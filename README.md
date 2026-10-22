---
specfork: 1
name: "@specfork/archiver"
summary: Desktop file archiver for Windows, macOS and Linux. Create, extract, list, test and update 7z, zip, tar, gzip, bzip2 and xz archives, with AES-256 and safe extraction.
license: CC-BY-4.0
tags: [desktop, files]
derived_from:
  repo: https://github.com/ip7z/7zip
  commit: b88c6bc09339de32f7d18bbab900686b244f42ba
  url: https://github.com/ip7z/7zip
  license: LGPL-2.1-or-later
---

# Archiver

A desktop program that packs files and folders into archives and unpacks them again. It creates
and updates 7z, zip, tar, gzip, bzip2 and xz archives, reads many more formats through a codec
contract, encrypts with AES-256 (optionally hiding file names), splits archives into volumes,
tests integrity, and plugs into the system file manager. Extraction is safe by default: nothing
is written outside the chosen folder. Behaviour derived from 7-Zip's open-source code
(LGPL-2.1-or-later), written fresh and stack-free.
Not affiliated with or endorsed by Igor Pavlov. The name is used only to credit the source.

| Area | Prefix | Covers |
|---|---|---|
| [formats.md](specs/formats.md) | FMT | Format registry, capabilities, methods, levels, codec contract, detection, metadata, volumes |
| [create.md](specs/create.md) | CRT | Creating and updating archives: inputs, naming, paths, solid blocks, update modes, links, delete-after |
| [extract.md](specs/extract.md) | XTR | Extraction modes, destination, overwrite prompts, volumes, free space, metadata restore |
| [safety.md](specs/safety.md) | SAFE | Path traversal, absolute paths, links, special files, reserved names, permissions |
| [names.md](specs/names.md) | NAM | Unicode names, legacy code pages, separators, collisions, length limits |
| [encryption.md](specs/encryption.md) | ENC | Passwords, AES-256, header encryption, legacy zip encryption, wrong-password handling |
| [browse.md](specs/browse.md) | BRW | Archive browser: listing, navigation, open, add, delete, rename, properties |
| [integrity.md](specs/integrity.md) | INT | Integrity test, error kinds, checksums of files |
| [jobs.md](specs/jobs.md) | JOB | Progress, pause, cancel, outcomes, messages, temporary files, performance |
| [integration.md](specs/integration.md) | SHL | File-manager menu actions, drag and drop, file associations, command line, settings |

Single user, local only: no account, no network, no telemetry. Every rule runs on one device.
Sizes use binary units (1 KiB = 1024 B); times are local; enum values are `snake_case`.
Keyboard shortcuts are defaults; a build may follow its platform's conventions instead.
