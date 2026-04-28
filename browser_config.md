# WSL Browser Setup

## Problem
WSL interoperability was disabled, preventing `xdg-open` from launching Windows browsers.

## Fix

1. Add to `/etc/wsl.conf`:
   ```ini
   [interop]
   enabled=true
   appendWindowsPath=true
   ```

2. Restart WSL from PowerShell/CMD:
   ```powershell
   wsl --shutdown
   ```

3. Relaunch your WSL terminal.

## Result
`xdg-open https://google.com` now opens the default Windows browser.
