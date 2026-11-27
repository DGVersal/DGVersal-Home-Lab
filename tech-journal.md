# DGVersal Home Lab tech journal

DGVersal Home Lab is where I build on my experience in end-user support and explore infrastructure, cloud, and automation through hands-on projects. I use this journal to record what I try, how I troubleshoot problems, and why a solution works. Over time, these entries will form a portfolio of the engineering skills I’m developing.

## 2026-11-27 — Finding Outlook data files with PowerShell

In support work, I’ve seen large Outlook files buried in a user’s AppData folder. I explored how a PowerShell script could find `.ost` files and help me choose a folder to inspect in WinDirStat.

- `$scanRoot = $env:LOCALAPPDATA` stores the current user’s local AppData path so the script can work across different Windows profiles.
- `Get-ChildItem` with `-Filter '*.ost' -File -Recurse` looks for matching files throughout the folder tree. `-Recurse` is what lets the search reach files several folders deep.
- `Sort-Object Length -Descending` puts the largest matches first. File size alone doesn’t tell me whether Outlook still uses a file.

I reviewed the first two sections of the script in discussion. A hands-on Windows scan is still ahead. Next time I’ll look at how the script reports its starting folder and flags an incomplete scan, then revisit my AppData question.
