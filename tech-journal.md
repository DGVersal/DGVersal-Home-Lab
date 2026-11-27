# DGVersal Home Lab tech journal

DGVersal Home Lab is where I build on my experience in end-user support and explore infrastructure, cloud, and automation through hands-on projects. I use this journal to record what I try, how I troubleshoot problems, and why a solution works. Over time, these entries will form a portfolio of the engineering skills I’m developing.

## 2026-11-27 — Finding Outlook data files with PowerShell

In support work, I’ve seen large Outlook files buried in a user’s AppData folder. I explored how a PowerShell script could find `.ost` files and help me choose a folder to inspect in WinDirStat.

- `$scanRoot = $env:LOCALAPPDATA` stores the current user’s local AppData path so the script can work across different Windows profiles.
- `Get-ChildItem` with `-Filter '*.ost' -File -Recurse` looks for matching files throughout the folder tree. `-Recurse` is what lets the search reach files several folders deep.
- `Sort-Object Length -Descending` puts the largest matches first. File size alone doesn’t tell me whether Outlook still uses a file.

I finished the six-section walkthrough in discussion. My next hands-on step is to run the report on Windows and compare one result with its folder in WinDirStat.

### Reading the script again

| Step | PowerShell | How I read it |
| --- | --- | --- |
| 1. Prepare | `$scanRoot = $env:LOCALAPPDATA` and `$scanErrors = @()` | `$` introduces a variable. `env:` looks up a Windows environment variable. `=` stores the path in `$scanRoot`. Empty `@()` creates an array ready to collect errors. |
| 2. Find and sort | `Get-ChildItem -LiteralPath $scanRoot -Filter '*.ost' -File -Recurse -Force` | Start at this exact path. `*` matches any filename prefix. Return files, search subfolders, and include hidden items. `-Force` still respects permissions. The pipe (`\|`) sends matching file objects to `Sort-Object Length -Descending`. Wrapping the search in `@(...)` collects the sorted matches into an array, even with zero or one match. |
| 3. Report scan problems | `-ErrorAction SilentlyContinue -ErrorVariable +scanErrors` | These parameters belong to `Get-ChildItem`. Continue past non-terminating errors quietly and append them (`+`) to the named variable. `Write-Host` shows the scanned path every time. `if ($scanErrors.Count -gt 0) { ... }` runs `Write-Warning` only when the error count is greater than zero. An optional `else` can report that the scan recorded no errors. |
| 4. Handle zero matches | `if ($ostFiles.Count -eq 0) { ... }` | The dot reads the `Count` property. `-eq` asks “equal to?” When the answer is yes, the script prints the no-matches message and `return` ends this run. A scan can finish successfully and find zero files. |
| 5. Calculate the total | `($ostFiles \| Measure-Object -Property Length -Sum).Sum` | `Length` is each file’s size in bytes. `-Sum` requests addition. The parentheses group the calculation, then `.Sum` reads the resulting total. Dividing by `1GB` converts bytes to GiB. |
| 6. Report each file | `foreach ($file in $ostFiles) { ... }` | Run the block once per file. `[PSCustomObject]@{ ... }` turns labeled values into one report entry. `SizeGiB` is rounded with `[math]::Round($file.Length / 1GB, 2)`. `::` calls the `Round` method on `Math`. `LastWriteTime` records the last write. `FullPath` includes the filename. `Folder` is the directory I can open in WinDirStat. |

### Syntax I want to remember

- **`-f` fills in a string.** In `"Found {0} files, totaling {1:N2} GiB." -f 3, 4.5`, `{0}` takes the first value (`3`). `{1:N2}` takes the second (`4.5`) and displays `4.50`. The comma separates the values. `Write-Host` displays the finished string.
- **Braces depend on context.** In `if (...) { ... }`, they enclose instructions. In `@{ Folder = $file.DirectoryName }`, they enclose key-value pairs. `Folder` is the label and `$file.DirectoryName` supplies the value. In a format string, `{0}` is a placeholder.
- **Quotes and line breaks.** Double quotes expand variables, so `"Scanned: $scanRoot"` displays the stored path. Single quotes pass `*.ost` as written for the file filter to interpret. `#` starts a comment. A backtick at the very end of a line continues the command on the next line.

**Example interpretation:** Three OST files in one folder produce three entries with distinct `FullPath` values and the same `Folder`. If the scan reports an access error, those entries are still useful, but another folder may contain additional matches.
