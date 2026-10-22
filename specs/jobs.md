# Jobs

**Status:** Draft · **Prefix:** JOB

## Purpose

Every create, update, extract, test or checksum run is a job the user can watch, pause and stop.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| JOB-1 | A job runs off the interface thread; windows stay responsive and several jobs can run at once, each with its own progress view. | start 2 extractions → both progress, UI responds |
| JOB-2 | Progress shows percent, elapsed, estimated remaining, speed (bytes/s), bytes and files processed of total, current item path, compression ratio (create), and error count; it updates at least every 500 ms. | manual: watch a 2 GiB create job |
| JOB-3 | Pause holds the job; resume continues where it stopped; `background` lowers its CPU priority. | pause 10 s, resume → same result as unpaused |
| JOB-4 | Cancel stops the job within 1 s and cleans up per [CRT-10](create.md) and [XTR-8](extract.md); files finished before cancel stay. Quitting the app with running jobs asks first, listing them, then cancels them. | cancel at 40% of 10 files → finished files remain, no partial |
| JOB-5 | Outcome: `succeeded`, `completed_with_warnings`, `completed_with_errors`, `failed`, `cancelled`, shown with counts. A `succeeded` view closes by itself unless `keep_job_view_open` (default off); others stay until dismissed. | 1 skipped input → `completed_with_warnings`, view stays |
| JOB-6 | Messages list every warning and error as `{severity, item_path, kind, detail}` ([INT-3](integrity.md)); the list copies as text. | copy → one line per message |
| JOB-7 | On an item error the user picks `skip_item` or `stop`, except where this spec skips automatically. | unreadable source → skip/stop offered |
| JOB-8 | Temporary files go to `temp_folder` (default: the target's volume, so the final rename is atomic) and are removed at job end whatever the outcome. | after cancel → no temp files |
| JOB-9 | Packing 1 GiB of mixed files to `7z` (level 5, all threads) keeps all logical CPUs busy for most of the run and stays within the memory estimate ([FMT-9](formats.md)) + 10%. | manual: monitor CPU and memory |
| JOB-10 | Extracting a 4 GiB `zip` of 50,000 files runs at ≥ 70% of the speed of the OS copying the same unpacked tree on the same disk. | manual: time both |

## Out of scope

- Job queues that survive restarts; scheduling.

## Decisions

- Pause and cancel are checked between data blocks, which bounds their latency (JOB-4).
