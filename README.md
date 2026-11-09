# ShredX 1.1.0 

## What is ShredX?

ShredX deletes files and folders so they cannot be recovered: before deletion, the content is overwritten several times and the name is changed randomly several times. ShredX can also securely clean the free space of a drive and the Recycle Bin.

## Adding files and folders

- Drag & drop files or folders onto the window.
- Explorer context menu "Securely delete with ShredX": enable it in the settings. On Windows 11 it is under "Show more options". The command line works too: `ShredX.exe file1 file2`.
- "Add files" opens a file dialog, also for several files at once.
- "Add folder" selects an entire folder.
- For folders, the direct content is shown indented. These sub-rows are for display only.
- Duplicate entries are ignored.
- "Remove" takes the selected entries off the list (multi-select with Ctrl/Shift). The files themselves are kept.
- "Clear list" removes all entries from the list.

## Shred permanently

Add items, click "Shred permanently" and confirm the prompt.

- Before the prompt, ShredX shows the number and size of the files, the amount of data to write and, once measured, the estimated time.
- Each file is overwritten with the selected method. Every pass is written straight to the disk.
- The file is then renamed randomly three times and deleted, so the original name does not remain.
- Before deletion the file is truncated to 0 bytes and its timestamps are reset. With decoy files the size is kept.
- Folders are deleted after their content, deepest first.
- Read-only files are unlocked automatically.
- For links (symbolic links, junctions) only the link is removed; the target is not touched.
- Files with further hard links (the same content under several names, e.g. in the pnpm store or in hard-link backups) are not overwritten; only the selected name is removed. Otherwise the file would also be destroyed under the other names. If all names are selected, the last one is overwritten normally.
- If files are in a cloud folder, on a ReFS drive or have further hard links, a warning appears at the top of the confirmation.
- Hidden data streams (alternate data streams) of files and folders are overwritten as well.
- For protection, drives and system folders (e.g. Windows, Program Files, the user profile) cannot be added.
- A progress bar, the current file and the estimated time remaining (ETA) show the status.
- "Cancel" stops the operation. Files already processed cannot be recovered.
- Deleted entries disappear from the list; entries that could not be deleted stay. Errors are shown in full in a separate window, with "Copy" and "Open log".
- Optionally, ShredX verifies after overwriting that the data really is on the disk (settings).
- "Shut down when finished" shuts the computer down after an error-free run. Windows waits one minute, during which you can cancel.

## Wipe free space

Overwrites remnants of files that were deleted earlier without ShredX.

- Under "Drive" select an internal or removable drive, then click "Wipe free space" and confirm.
- ShredX fills the free space with temporary files (1 GB each) and deletes them afterwards.
- 1 GB is left free so Windows and running programs keep working.
- The "Default drive erasure method" from the settings is used.
- Depending on the drive size this can take a long time. You can cancel at any time; the temporary files are cleaned up.
- SSDs are marked in the drive list. Overwriting free space is not reliable there; Windows cleans up free areas automatically via TRIM.
- Optional (settings): overwrite the cluster tips of all files. Each file is briefly extended, the rest of its last cluster is overwritten and the file is truncated again. Locked, read-only and system files are skipped.
- Optional (settings, NTFS only): overwrite free MFT records, which can hold small deleted files. Without administrator rights this stops after 100,000 records.
- "Overwrite the 1 GB reserve" (settings, on by default): at the end of each pass the reserve is also filled to the limit and released immediately. The drive is completely full for a few seconds.
- Cluster tips: folders of cloud services (OneDrive, Dropbox, Google Drive, iCloud …) are skipped. If ShredX is interrupted, it restores the size of the last processed file automatically at the next start.

## Empty recycle bin

- Select a drive, click "Empty recycle bin" and confirm.
- All files in the Recycle Bin of that drive are overwritten with the file method and deleted.
- Recycle Bin folders of other user accounts are usually not readable and are skipped.

## Remove traces

"Remove traces…" overwrites and deletes what Windows stores about used files:

- Recently used files and taskbar jump lists
- Explorer thumbnail cache (`thumbcache_*.db`). Explorer usually keeps these files open; they can often only be deleted after signing in again.
- Recycle bins of all drives

The file method from the settings is used.

## Deletion report and error log

- If "Save a deletion report" is on, ShredX creates a text file after each operation (default: `Documents\ShredX Reports`). It contains date, computer, user, method, count, amount of data and number of errors.
- "Hard links" counts files whose name was only removed because they have further hard links. Lines with "Warning" name files in cloud folders and ReFS drives.
- File names are not included, only the SHA-256 of the full path in lower case. Anyone who knows the path can use it to prove that the file was deleted.
- The deletion report is in English so that it can be processed automatically.
- The error log (`error.log`) contains no file or folder names. It can be turned off and securely deleted in the settings.

## "Send to" and command line

- "Send to" (settings) passes all selected files at once; with large selections it is more reliable than the context menu.
- Without a window, e.g. for scripts and Task Scheduler:

```text
ShredX.exe --silent [--method Name] [--verify] [--report Folder] paths…
ShredX.exe --silent --wipe-free D:
ShredX.exe --silent --recycle-bin D: | all
ShredX.exe --help
```

- `--silent` does not ask for confirmation. Drives, system folders and the user profile are refused.
- Exit codes: 0 success, 1 some items failed, 2 invalid arguments, 3 protected path, 4 canceled.
- To wait for the end: `start /wait ShredX.exe …` (cmd) or `Start-Process -Wait` (PowerShell).

## Erasure methods

Configurable via the gear icon ("Erase settings"):

- "Default file erasure method": for files, folders and the Recycle Bin. Default: Pseudorandom Data.
- "Default drive erasure method": for free space. Default: Pseudorandom Data.

Methods (number of passes):

- Gutmann (35)
- US DoD 5220.22-M (8-306./E, C & E) (7)
- RCMP TSSIT OPS-II (7)
- Schneier 7 pass (7)
- German VSITR (7)
- US DoD 5220.22-M (8-306./E) (3)
- British HMG IS5 (Enhanced) (3)
- US Air Force 5020 (3)
- US Army AR380-19 (3)
- Russian GOST P50739-95 (2)
- British HMG IS5 (Baseline) (1)
- Pseudorandom Data (1)
- First/last 16KB Erasure (1)

First/last 16KB only overwrites the first and last 16 KB of a file (files up to 32 KB completely). It is very fast but not complete.

Tip: on modern hard drives one pass of random data is usually enough. More passes take correspondingly longer.

## Locked files

If the option is enabled in the settings and a file is open in another program, ShredX shows the programs involved and asks whether they should be asked to close. The programs are asked to close via the Windows Restart Manager, but are not forcibly terminated. Deletion is then retried.

## Decoy files (plausible deniability)

If the option is enabled, the content of the selected decoy files is written into the file as the last pass after overwriting. Anyone who recovers the deleted file only finds this content.

- Manage the list with "Add…" and "Remove".
- At most 64 MB of decoy data is used, repeated if needed.

## App options (top bar)

- Language: Deutsch, English, Español, Français, Indonesia, Italiano, Português, Русский, Türkçe, العربية, 中文, 日本語, 한국어, বাংলা, हिन्दी. Arabic is displayed right-to-left.
- Color scheme: System (Auto), Light Mode, Dark Mode, Dracula, Emerald, Gruvbox, High Contrast Dark, High Contrast Light, Monokai, Nord, Rose Gold, Purple, Solarized Dark, Solarized Light, Amber. "System (Auto)" follows the Windows setting; the title bar adapts automatically.
- Lightning icon: start with Windows. ShredX then starts in the background as a tray icon. A circle around the lightning shows that the option is active.
- Question mark: opens this Help Center.
- Updates: at every start ShredX asks GitHub for a new version. "Update" opens the download page, "Skip this version" stops reporting it, "Cancel" asks again at the next start. This is ShredX's only internet connection; no file or folder names are sent. Without internet nothing happens.

## Tray icon

- When minimized, ShredX disappears from the taskbar and keeps running in the notification area.
- Double-clicking the icon reopens the window. The context menu (right-click) lets you open or exit the app.
- Only one instance of ShredX runs at a time.

## Important notes

- Deleted data cannot be recovered. Check the list carefully before confirming.
- SSDs and USB sticks: because of wear leveling and TRIM, complete overwriting is not guaranteed. Multiple passes do not help there.
- ReFS drives (including the Windows 11 "Dev Drive"): changed data is written to a new location (copy-on-write), so overwriting does not reach the old content. ShredX warns about this.
- Cloud folders (OneDrive, Dropbox etc.): files stored locally are overwritten, but the overwritten data is uploaded. Older versions remain in the cloud, in the version history and in the online recycle bin; ShredX warns about this. Files stored only in the cloud are only removed locally.
- Shadow copies, restore points, File History and backups may contain older versions.
- NTFS-compressed files are decompressed before overwriting. Their old content may remain in free space; run "Wipe free space" afterwards.
- Windows also records file names in the NTFS change journal (`$UsnJrnl`, `$LogFile`) and the thumbnail cache. ShredX cannot remove these entries completely.
- "First/last 16KB" only overwrites the beginning and end of a file; the rest stays readable.

