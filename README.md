# Windows RDP History & Credential Cleaner

A PowerShell utility for cleaning saved Remote Desktop Protocol (RDP) connection information from Windows.

This project helps Windows users remove previously saved RDP credentials, usernames, server IP addresses, connection history, cached RDP data, and Start Menu recent connection entries.

## Features

- **Saved Credentials:** Removes saved Remote Desktop passwords and `TERMSRV` credentials from Windows Credential Manager.
- **Connection History:** Clears previously connected server IP addresses and hostnames.
- **Username History:** Removes saved RDP username hints from the registry.
- **Start Menu History:** Clears Remote Desktop entries from Windows Jump Lists.
- **RDP Cache:** Removes cached Remote Desktop client data.
- **RDP Profiles:** Deletes `.rdp` connection files from selected user folders.
- **Explorer Refresh:** Restarts Windows Explorer to refresh the Start Menu.

## Requirements

- Windows 10 or Windows 11.
- Windows PowerShell 5.1 or later.
- Access to the Windows user profile being cleaned.
- No additional software required.

## Installation

### Option 1: Download from GitHub

1. Download or clone this repository.
2. Extract the downloaded ZIP file.
3. Locate the PowerShell script.
4. Open PowerShell under the Windows account whose RDP history you want to clean.

### Option 2: Clone the repository

```powershell
git clone https://github.com/YOUR-USERNAME/windows-rdp-cleaner.git
cd windows-rdp-cleaner
```

## Usage

Run the script in PowerShell.

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\RDP-Cleaner.ps1
```

Run it under the Windows account whose saved RDP information you want to remove. Administrator privileges are generally not required for current-user cleanup.

## PowerShell Script

Save the following as `RDP-Cleaner.ps1`.

```powershell
# =====================================================
# WINDOWS RDP HISTORY & CREDENTIAL CLEANER
# =====================================================

Write-Host "`n========================================" -ForegroundColor Cyan
Write-Host "       WINDOWS RDP CLEANUP STARTED" -ForegroundColor Cyan
Write-Host "========================================" -ForegroundColor Cyan

# 1. Remove saved RDP credentials
Write-Host "`n[1/7] Removing saved RDP credentials..." -ForegroundColor Yellow

$credentials = cmdkey /list

foreach ($line in $credentials) {
    if ($line -match "Target:\s*(.+)") {
        $target = $matches[1].Trim()

        if ($target -match "TERMSRV") {
            Write-Host "Deleting: $target"
            cmdkey "/delete:$target"
        }
    }
}

# 2. Clear RDP registry history
Write-Host "`n[2/7] Clearing RDP registry history..." -ForegroundColor Yellow

$rdp = "HKCU:\Software\Microsoft\Terminal Server Client"

Remove-Item "$rdp\Servers" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item "$rdp\Default" -Recurse -Force -ErrorAction SilentlyContinue

# 3. Remove saved RDP connection files
Write-Host "`n[3/7] Removing saved RDP profiles..." -ForegroundColor Yellow

$folders = @(
    "$env:USERPROFILE\Documents",
    "$env:USERPROFILE\Downloads",
    "$env:USERPROFILE\Desktop"
)

foreach ($folder in $folders) {
    if (Test-Path $folder) {
        Get-ChildItem -Path $folder -Filter "*.rdp" -File -Recurse -ErrorAction SilentlyContinue |
        ForEach-Object {
            Write-Host "Deleting: $($_.FullName)"
            Remove-Item $_.FullName -Force -ErrorAction SilentlyContinue
        }
    }
}

# 4. Clear RDP client cache
Write-Host "`n[4/7] Clearing RDP cache..." -ForegroundColor Yellow

$cache = "$env:LOCALAPPDATA\Microsoft\Terminal Server Client\Cache"

if (Test-Path $cache) {
    Remove-Item "$cache\*" -Recurse -Force -ErrorAction SilentlyContinue
}

# 5. Clear Windows Jump Lists
Write-Host "`n[5/7] Clearing Start Menu recent history..." -ForegroundColor Yellow

Stop-Process -Name explorer -Force -ErrorAction SilentlyContinue

$auto = "$env:APPDATA\Microsoft\Windows\Recent\AutomaticDestinations"
$custom = "$env:APPDATA\Microsoft\Windows\Recent\CustomDestinations"

if (Test-Path $auto) {
    Remove-Item "$auto\*" -Force -ErrorAction SilentlyContinue
}

if (Test-Path $custom) {
    Remove-Item "$custom\*" -Force -ErrorAction SilentlyContinue
}

# 6. Restart Windows Explorer
Write-Host "`n[6/7] Restarting Windows Explorer..." -ForegroundColor Yellow

Start-Process explorer.exe

# 7. Verify remaining RDP credentials
Write-Host "`n[7/7] Checking remaining RDP credentials..." -ForegroundColor Yellow

$remaining = cmdkey /list | Select-String "TERMSRV"

if ($remaining) {
    Write-Host "Some RDP credentials may remain:" -ForegroundColor Red
    $remaining
}
else {
    Write-Host "No TERMSRV credentials found." -ForegroundColor Green
}

Write-Host "`n========================================" -ForegroundColor Green
Write-Host "       RDP CLEANUP COMPLETED!" -ForegroundColor Green
Write-Host "========================================" -ForegroundColor Green

Write-Host "`nRestart Windows if necessary." -ForegroundColor Cyan
```

## What Gets Cleaned?

| Windows Component | Cleanup |
|---|---|
| Credential Manager | Saved TERMSRV credentials |
| Registry | RDP server connection history |
| Registry | RDP MRU history |
| Registry | Saved username hints |
| Documents | Saved `.rdp` files |
| Downloads | Saved `.rdp` files |
| Desktop | Saved `.rdp` files |
| RDP Client | Cached connection data |
| Start Menu | Recent RDP connection entries |
| Windows Explorer | Restarted to refresh history |

## Important Warnings

**Please read before running:**

1. The script deletes saved `.rdp` files recursively from Documents, Downloads, and Desktop. Back up any connection profiles you want to keep.
2. Clearing Windows Jump Lists also removes recent-item history for other applications, not just Remote Desktop.
3. The script cleans the current Windows user's profile, not every account on the computer.
4. It does not delete Windows user accounts or disable Remote Desktop.
5. It does not erase server-side authentication logs, Windows Security event logs, or records of incoming RDP connections.
6. Removing saved credentials means you may need to enter your passwords again when reconnecting.

## Verification

After running the script, execute:

```powershell
cmdkey /list
```

Check that no saved `TERMSRV` credentials remain.

You can also open:

```powershell
mstsc.exe
```

Check the Remote Desktop Connection application to confirm that previous connections no longer appear in its Recent section.

## Troubleshooting

**Problem: Some IP addresses still appear.**

Close Remote Desktop Connection, run the cleanup again under the correct Windows account, and restart Explorer.

**Problem: Credentials still appear in Credential Manager.**

Check whether the remaining entry is actually a `TERMSRV` credential or belongs to another application.

**Problem: Start Menu still displays old entries.**

Restart Windows and check the Remote Desktop Connection application's Recent section again.

## Project Purpose

This project is intended for legitimate Windows administration, personal device maintenance, and privacy-conscious cleanup of saved Remote Desktop connection information.

## License

MIT License.

## Contributions

Contributions, bug reports, and improvements are welcome.

If you find an issue, please open a GitHub Issue with:

- Windows version.
- PowerShell version.
- Error message.
- Steps to reproduce the issue.

---

**Disclaimer:** This utility only cleans selected local RDP client data. It is not an anti-forensics tool and does not remove system audit records, server logs, or other security monitoring records.
