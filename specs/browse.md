# Archive browser

**Status:** Draft · **Prefix:** BRW

## Purpose

Looking inside an archive like a folder, and changing it where the format allows.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| BRW-1 | Opening an archive shows its top level as a folder; folders implied by item paths but not stored as entries are shown too. | zip with only `a/b.txt` → folder `a` listed |
| BRW-2 | Columns: `name`, `size`, `packed_size`, `modified`, plus optional `created`, `accessed`, `attributes`, `encrypted`, `crc32`, `method`, `block`, `host_os`; a column the format lacks is blank. Folder `size` = sum of its contents. | folder with 2 × 1 KiB files → 2 KiB |
| BRW-3 | Any column sorts ascending/descending on click and the choice persists. Names sort case-insensitively with digit runs compared as numbers, folders first. | `f2`, `f10`, `F1` → `F1`, `f2`, `f10` |
| BRW-4 | Navigation: open a folder (Enter or double-click), go up (Backspace), a path bar `archive › folder › …` with clickable parts, and back/forward history within the archive. | Enter on `a/`, Backspace → root |
| BRW-5 | A filter box narrows the current folder by case-insensitive name substring as the user types. | type `rep` → only names containing `rep` |
| BRW-6 | Opening a file item extracts it to a private temporary folder and opens it with the OS default app (after [SAFE-12](safety.md) where it applies); a nested archive opens inside the browser instead. | open `notes.txt` → default editor shows it |
| BRW-7 | When an item opened per BRW-6 changes on disk while the archive is open, the user is asked whether to write it back (formats with `update`); `yes` runs an update job. | edit and save → prompt; yes → item updated |
| BRW-8 | Dropping files onto the browser, or the add command, runs an update job (`add_replace`, [CRT-8](create.md)) into the current folder. Dragging items out to a file-manager folder extracts them there per [XTR](extract.md). | drop `x.txt` in `docs/` → `docs/x.txt`; drag `a.txt` to desktop → file there |
| BRW-9 | Delete (Delete key) asks for confirmation with the item count, says the change is permanent, then rewrites the archive per [CRT-10](create.md). | delete 2, confirm → 2 fewer items |
| BRW-10 | Rename (F2) and new folder change the archive in place; a name already in that folder is refused. For formats without `update`, and split archives, add, delete, rename and new folder are disabled and the window says read-only. | rename to existing name → refused; open `.iso` → delete disabled |
| BRW-11 | Properties: format, physical size, unpacked total, file and folder counts, methods, solid yes/no and block count, `encrypt_names`, volume count, offset when embedded in a larger file, and any open warnings. | solid 7z → 1 block shown |
| BRW-12 | An archive of 100,000 items shows its first screen within 3 s and scrolls without visible stalls. | manual: 100,000-item zip on a mid-range laptop |

## Out of scope

- A general two-pane file manager for the disk; editing archive comments.

## Decisions

- The browser is the only editing surface; the file-manager menu ([SHL](integration.md)) only creates, extracts, tests and hashes.
- Write-back of opened items is always asked, never automatic.
