# WSL disk repair — run these in PowerShell (as Administrator)

The Ubuntu ext4.vhdx has filesystem corruption (symptoms: `EROFS: read-only
file system`, `sudo: Input/output error`). Fix = fsck the VHD from a second
tiny distro. ~10 minutes.

## 1. Shut everything down
```powershell
wsl --shutdown
```
(This kills the Claude session — resume afterwards with `claude --resume`.)

## 2. Back up Ubuntu first (safety net, ~few GB)
```powershell
wsl --export Ubuntu D:\ubuntu-backup.tar
```
If this errors, skip it and continue — project files are already on D:.

## 3. Find the Ubuntu disk file
```powershell
Get-ChildItem "$env:LOCALAPPDATA\Packages\*Ubuntu*\LocalState\ext4.vhdx"
```
If nothing found, try:
```powershell
Get-ChildItem "$env:LOCALAPPDATA\wsl" -Recurse -Filter ext4.vhdx -ErrorAction SilentlyContinue
```
Note the full path — call it VHDX_PATH below.

## 4. Install a tiny helper distro (Debian)
```powershell
wsl --install -d Debian
```
When it opens and asks for a username/password, pick anything (e.g. temp/temp).
Then close its window.

## 5. Attach the broken disk and repair it
```powershell
wsl --shutdown
wsl --mount "VHDX_PATH" --bare
wsl -d Debian
```
Now inside Debian:
```bash
lsblk                          # find the new disk, e.g. sdc or sdd (no MOUNTPOINT)
sudo e2fsck -f -y /dev/sdX     # replace sdX with the disk from lsblk
exit
```
`e2fsck` will print lots of "Fixing..." lines — that's it working. -y answers
yes to all repairs.

## 6. Detach and restart Ubuntu
```powershell
wsl --unmount
wsl -d Ubuntu
```
Test inside Ubuntu:
```bash
touch /tmp/writetest && echo FIXED
sudo echo sudo-works
```

## 7. Resume Claude
```powershell
claude --resume
```
Pick the n8n-ai-inbox-manager session. Tell Claude "filesystem fixed" and it
continues from: docker compose up → import workflows → end-to-end test.

## Optional cleanup afterwards
```powershell
wsl --unregister Debian
```

## If this recurs
Repeated VHD corruption usually means the underlying Windows drive has
problems. Check C: health: `chkdsk C: /scan` and look at SMART status
(CrystalDiskInfo). Also avoid hard power-offs while WSL is running.
