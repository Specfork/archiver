# Desktop integration

**Status:** Draft · **Prefix:** SHL

## Purpose

Reaching the archiver from the system file manager and the command line, and its settings.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| SHL-1 | File-manager menu actions: `open_archive`, `extract_with_options`, `extract_here`, `extract_to_folder`, `test_archive` (on archives); `add_with_options`, `add_to_7z`, `add_to_zip` (on any items); `checksum` (on files and folders). Each OS's supported extension mechanism is used. | right-click a zip → extract actions present |
| SHL-2 | `extract_here` writes into the archive's folder: a single top item goes there directly, several go into a folder named per [XTR-2](extract.md). `extract_to_folder` always uses [XTR-2](extract.md) and shows the folder name in the entry. | zip with 3 top files → `name/` created; menu shows `photos/` |
| SHL-3 | `add_to_7z` and `add_to_zip` create the archive at the default path ([CRT-2](create.md)) with default options and no dialog; an existing file of that name is updated with `add_replace`. | 2 files selected → `<folder>.7z` created |
| SHL-4 | Archive actions appear only on files whose extension or signature matches an `extract` format; with several archives selected they apply to each ([XTR-13](extract.md)). | right-click `.txt` → no extract actions |
| SHL-5 | `menu_actions` (which actions show; default all) and `menu_cascaded` (in one submenu; default on) apply at once. | turn off `checksum` → gone from menu |
| SHL-6 | File associations are opt-in per extension; none are claimed without the user's choice. Dropping files on the app opens the add dialog; dropping an archive opens the browser. | fresh install → `.zip` still opens in previous app |
| SHL-7 | Command line: `a <archive> <inputs…>` create/update, `x <archive> [-o <dir>]` extract with paths, `e` extract flat, `t` test, `l` list, `d <archive> <paths…>` delete items, `h <files…>` checksums; switches cover every option of [CRT-1](create.md) and [XTR-1](extract.md). Prompts become errors unless `-y` (assume yes) is given or stdin is a terminal. | `x a.zip -o out` → files in `out/` |
| SHL-8 | `l` prints one line per item (modified, attributes, size, packed size, path) and totals; `l -json` prints a JSON array with the fields of [BRW-2](browse.md). | `l -json a.zip` → valid JSON array |
| SHL-9 | Exit codes: 0 succeeded, 1 completed with warnings, 2 failed or completed with errors, 7 bad command line, 8 out of memory, 255 cancelled. | test corrupt archive → 2 |
| SHL-10 | Settings, stored locally per user: `default_format` (`7z`), `default_level` (5), `legacy_code_page`, `temp_folder`, `keep_history` (on: last 20 archive paths and last options per format), `keep_job_view_open` (off), `extract_links` (on), `origin_mark` (`all`), `menu_actions`, `menu_cascaded`, `language` (OS language). "Clear history" empties the remembered paths and options after a confirmation. | change default format → next add dialog preselects it; clear → no previous paths |
| SHL-11 | The app works fully offline and sends nothing over the network: no update checks, analytics or crash uploads. | manual: run every job type with network blocked, watch traffic |

## Out of scope

- Email-after-compress, cloud upload, installers.

## Decisions

- No `@specfork/privacy` requirement: the app keeps no personal records, only settings and path history, which SHL-10 clears. Privacy's local mode would add a ZIP export of nothing useful.
- Exit codes follow the source's widely scripted convention, so existing scripts keep working.
- Silent `add_to_*` actions trade options for speed; the options dialog is one entry away.
