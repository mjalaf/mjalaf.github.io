---
title: "Windows Run Commands & Admin Tools"
description: "Quick reference for Windows Run-dialog shortcuts: MMC consoles (.msc), system utilities, control panel applets, and shell:: locations."
category: "Windows"
---

Open the **Run** dialog with `Win + R`, then type any of the following. Most also work from PowerShell, `cmd`, or the Start menu search.

## Most-used MMC consoles (.msc)

| Command | Opens |
| --- | --- |
| `services.msc` | Services |
| `devmgmt.msc` | Device Manager |
| `diskmgmt.msc` | Disk Management |
| `compmgmt.msc` | Computer Management (umbrella for Event Viewer, Services, Shared Folders, etc.) |
| `eventvwr.msc` | Event Viewer |
| `taskschd.msc` | Task Scheduler |
| `gpedit.msc` | Local Group Policy Editor (Pro/Enterprise only) |
| `secpol.msc` | Local Security Policy |
| `lusrmgr.msc` | Local Users and Groups (Pro/Enterprise only) |
| `certmgr.msc` | Certificates – Current User |
| `certlm.msc` | Certificates – Local Machine |
| `perfmon.msc` | Performance Monitor |
| `wf.msc` | Windows Defender Firewall with Advanced Security |
| `fsmgmt.msc` | Shared Folders |
| `rsop.msc` | Resultant Set of Policy |
| `printmanagement.msc` | Print Management |
| `wmimgmt.msc` | WMI Control |
| `dsa.msc` | Active Directory Users and Computers (RSAT) |
| `dssite.msc` | Active Directory Sites and Services (RSAT) |
| `domain.msc` | Active Directory Domains and Trusts (RSAT) |
| `dnsmgmt.msc` | DNS Manager (RSAT / Server) |
| `dhcpmgmt.msc` | DHCP (RSAT / Server) |
| `servermanager.msc` | Server Manager (Server) |

## System utilities & dialogs

| Command | Opens |
| --- | --- |
| `regedit` | Registry Editor |
| `msconfig` | System Configuration (boot, services, startup) |
| `msinfo32` | System Information |
| `dxdiag` | DirectX Diagnostic Tool |
| `taskmgr` | Task Manager |
| `resmon` | Resource Monitor |
| `perfmon` | Performance Monitor |
| `eventvwr` | Event Viewer |
| `cleanmgr` | Disk Cleanup |
| `dfrgui` | Defragment and Optimize Drives |
| `winver` | About Windows (version/build dialog) |
| `sysdm.cpl` | System Properties (Environment Variables, Remote, Advanced) |
| `SystemPropertiesAdvanced` | System Properties → Advanced tab |
| `SystemPropertiesProtection` | System Properties → System Protection |
| `SystemPropertiesRemote` | System Properties → Remote |
| `rundll32 sysdm.cpl,EditEnvironmentVariables` | Environment Variables dialog directly |
| `optionalfeatures` | Turn Windows features on or off |
| `appwiz.cpl` | Programs and Features (uninstall apps) |
| `control` | Classic Control Panel |
| `ms-settings:` | Modern Settings (root) |
| `osk` | On-Screen Keyboard |
| `magnify` | Magnifier |
| `narrator` | Narrator |
| `snippingtool` / `ms-screenclip:` | Snipping Tool / Screen Snip |
| `mstsc` | Remote Desktop Connection |
| `mblctr` | Windows Mobility Center (laptops) |
| `sigverif` | File Signature Verification |
| `verifier` | Driver Verifier Manager |
| `psr` | Steps Recorder (a.k.a. Problem Steps Recorder) |
| `wscript` / `cscript` | Windows Script Host |
| `cmd` / `powershell` / `pwsh` / `wt` | Shells (Command Prompt / PowerShell / PS 7 / Windows Terminal) |

## Control Panel applets (`*.cpl`)

| Command | Opens |
| --- | --- |
| `appwiz.cpl` | Programs and Features |
| `desk.cpl` | Display Settings (legacy) |
| `firewall.cpl` | Windows Defender Firewall |
| `inetcpl.cpl` | Internet Properties |
| `intl.cpl` | Region |
| `joy.cpl` | Game Controllers |
| `main.cpl` | Mouse Properties |
| `mmsys.cpl` | Sound |
| `ncpa.cpl` | Network Connections (adapters) |
| `powercfg.cpl` | Power Options |
| `sysdm.cpl` | System Properties |
| `tabletpc.cpl` | Pen and Touch |
| `telephon.cpl` | Phone and Modem |
| `timedate.cpl` | Date and Time |
| `wscui.cpl` | Security and Maintenance |
| `hdwwiz.cpl` | Device Manager (alt entry) |
| `bthprops.cpl` | Bluetooth Devices |

## Modern Settings deep links (`ms-settings:`)

```
ms-settings:                 -> Settings home
ms-settings:about            -> System → About
ms-settings:display          -> Display
ms-settings:network          -> Network & Internet
ms-settings:network-status   -> Status
ms-settings:network-wifi     -> Wi-Fi
ms-settings:network-ethernet -> Ethernet
ms-settings:network-vpn      -> VPN
ms-settings:network-proxy    -> Proxy
ms-settings:windowsupdate    -> Windows Update
ms-settings:windowsupdate-history
ms-settings:windowsdefender  -> Windows Security
ms-settings:appsfeatures     -> Installed apps
ms-settings:defaultapps      -> Default apps
ms-settings:storagesense     -> Storage
ms-settings:privacy          -> Privacy & security
ms-settings:bluetooth        -> Bluetooth & devices
ms-settings:printers         -> Printers & scanners
ms-settings:signinoptions    -> Sign-in options
ms-settings:dateandtime      -> Date & time
ms-settings:regionlanguage   -> Language & region
ms-settings:developers       -> For developers
ms-settings:gaming-gamebar   -> Game Bar
```

Run from PowerShell with: `Start-Process "ms-settings:windowsupdate"`

## `shell::` GUIDs (My PC, special folders)

```
shell:startup            -> Current user's Startup folder
shell:common startup     -> All-users Startup folder
shell:sendto             -> "Send to" menu items
shell:profile            -> Your user profile (%USERPROFILE%)
shell:appdata            -> %APPDATA%
shell:local appdata      -> %LOCALAPPDATA%
shell:programs           -> Start Menu Programs
shell:common programs    -> Start Menu Programs (All users)
shell:recent             -> Recently used items
shell:fonts              -> Fonts folder
shell:my documents       -> Documents
shell:cookies            -> Cookies
shell:cache              -> INetCache
explorer shell:::{20D04FE0-3AEA-1069-A2D8-08002B30309D}   -> "This PC"
explorer shell:::{645FF040-5081-101B-9F08-00AA002F954E}   -> Recycle Bin
explorer shell:::{ED7BA470-8E54-465E-825C-99712043E01C}   -> "God Mode" (all settings flat list)
```

> **"God Mode" folder trick:** create a new folder named exactly
> `GodMode.{ED7BA470-8E54-465E-825C-99712043E01C}` — it turns into a single window listing every Control Panel item.

## Useful diagnostic & repair commands

```powershell
# System file checker + image repair (run elevated)
sfc /scannow
DISM /Online /Cleanup-Image /CheckHealth
DISM /Online /Cleanup-Image /ScanHealth
DISM /Online /Cleanup-Image /RestoreHealth

# Check disk
chkdsk C: /scan        # online (no reboot)
chkdsk C: /f /r        # offline, schedule on next reboot

# Reliability monitor (GUI)
perfmon /rel

# Power / battery report
powercfg /batteryreport /output C:\temp\battery.html
powercfg /energy   /output C:\temp\energy.html /duration 60

# Sleep diagnostics
powercfg /sleepstudy /output C:\temp\sleepstudy.html

# Boot configuration
bcdedit /enum
bcdedit /set {current} bootmenupolicy legacy

# Hardware / driver inventory
driverquery /fo table
pnputil /enum-drivers
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Boot Time"
```

## Quick service / process management

```powershell
# Services
Get-Service | Where-Object Status -eq 'Running'
Get-Service -Name 'W32Time'
Start-Service  -Name 'W32Time'
Stop-Service   -Name 'W32Time'
Restart-Service -Name 'W32Time' -Force
Set-Service    -Name 'W32Time' -StartupType Automatic

# Classic sc.exe (still common in scripts)
sc.exe query W32Time
sc.exe start W32Time
sc.exe config W32Time start= auto

# Processes
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10
Stop-Process -Name notepad -Force
```

## Bonus: launch from PowerShell

```powershell
# Open a Run-style command (no Win+R needed)
Start-Process services.msc
Start-Process regedit
Start-Process "ms-settings:windowsupdate"
Start-Process shell:startup

# Open Windows Terminal in a specific profile
wt -p "PowerShell" -d C:\src

# Elevate the current window without UAC clicking
Start-Process pwsh -Verb RunAs
```
