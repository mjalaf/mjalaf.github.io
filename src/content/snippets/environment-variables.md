---
title: 'Environment Variables: Windows & Linux'
description: 'Where environment variables live by default, how to add user/system/boot variables on Windows and Linux, and the precedence rules that decide which value wins.'
category: 'Environment'
---

A practical reference for **environment variables** on both platforms — defaults, where to add them, scope (user / system / process / boot), and the precedence that decides which value actually wins at runtime.

---

## 1. The mental model

| Scope | Lives until | Visible to |
| --- | --- | --- |
| **Process** | The process exits | Only that process and children it spawns |
| **Shell / Session** | The terminal closes / logout | That shell and children only |
| **User** | Persistent, survives reboot | Every process launched by that user |
| **System / Machine** | Persistent, survives reboot | Every user and service on the box |
| **Boot / Kernel** | Set before userspace starts | Kernel + early init (services, mount, network) |

Rule of thumb: **process > shell > user > system**. The most specific scope wins.

---

## 2. Linux — where variables come from

### Default sources (in load order)

1. **Kernel boot parameters** (`/proc/cmdline`) — passed by the bootloader; mostly kernel-level, but `init=` and `systemd.setenv=` reach userspace.
2. **PAM** (`/etc/security/pam_env.conf`, `/etc/environment`) — applied at login for every user.
3. **systemd manager** (`/etc/systemd/system.conf` → `DefaultEnvironment=`) — for services.
4. **Shell rc files** — login vs. interactive vs. non-interactive matter (see table below).
5. **Process inheritance** — every child inherits its parent's environment unless overridden.

### Where to add them (by intent)

| Goal | File | Scope | Notes |
| --- | --- | --- | --- |
| One-off in current shell | `export VAR=value` | Process + children | Lost on exit |
| For my user, always | `~/.profile` (or `~/.bash_profile`, `~/.zprofile`) | Login shells | Read once at login |
| For every interactive shell of mine | `~/.bashrc` / `~/.zshrc` | Interactive shells | Re-sourced per terminal |
| For all users (login) | `/etc/environment` | All login sessions | **KEY=VALUE only**, no shell syntax, no `export` |
| For all users (any shell) | `/etc/profile` or drop a file in `/etc/profile.d/*.sh` | Login shells | Full shell syntax allowed |
| For a systemd service | `Environment=` / `EnvironmentFile=` in the unit | That service only | Survives reboot |
| Globally for all services | `/etc/systemd/system.conf` → `DefaultEnvironment=` | systemd-managed processes | Needs `systemctl daemon-reexec` |

```bash
# Process-scoped (only this command sees it)
DEBUG=1 ./run.sh

# Shell-scoped (this session + children)
export EDITOR=nvim

# User-scoped, persistent — append to ~/.profile or ~/.bashrc
echo 'export JAVA_HOME=/usr/lib/jvm/temurin-21' >> ~/.profile

# System-wide (preferred for non-shell consumers like GUI apps)
sudo tee -a /etc/environment <<'EOF'
JAVA_HOME=/usr/lib/jvm/temurin-21
EOF
```

> **Tip:** `/etc/environment` is parsed by PAM, **not** a shell. Don't use `export`, don't use `$OTHERVAR` expansion — it won't work.

### bash vs. zsh: which file is read?

| Shell type | bash reads | zsh reads |
| --- | --- | --- |
| Login | `/etc/profile` → `~/.bash_profile` → `~/.bash_login` → `~/.profile` | `/etc/zprofile` → `~/.zprofile` → `/etc/zlogin` → `~/.zlogin` |
| Interactive non-login | `~/.bashrc` | `/etc/zshrc` → `~/.zshrc` |
| Non-interactive (script) | `$BASH_ENV` if set | `$ZDOTDIR/.zshenv` → `~/.zshenv` |

> **Trick:** Put exports you want available to *every* zsh process (including IDE-spawned shells and `ssh user@host command`) in `~/.zshenv`, not `~/.zshrc`.

### Inspect and edit at runtime

```bash
# Print all env vars
printenv
env

# Just one
echo "$PATH"
printenv PATH

# See the env of a *running* PID (read-only)
cat /proc/<pid>/environ | tr '\0' '\n'

# Run a command with a clean env
env -i PATH=/usr/bin:/bin bash

# Unset
unset DEBUG

# Modify PATH safely (prepend, avoid duplicates)
case ":$PATH:" in *":$HOME/bin:"*) ;; *) export PATH="$HOME/bin:$PATH" ;; esac
```

### systemd services

```ini
# /etc/systemd/system/myapp.service
[Service]
Environment="APP_ENV=production"
Environment="LOG_LEVEL=info"
EnvironmentFile=-/etc/myapp.env   # leading '-' = optional
ExecStart=/usr/local/bin/myapp
```

```bash
sudo systemctl daemon-reload
sudo systemctl restart myapp
systemctl show myapp -p Environment   # inspect what the service will see
```

> **Trick:** Set a variable for a *running* service without editing the unit:
> `sudo systemctl set-environment FOO=bar && sudo systemctl restart myapp`

### Boot-time / kernel variables (Linux)

These are set **before** userspace and aren't classic env vars, but they shape what userspace sees:

```bash
# Inspect what the kernel was booted with
cat /proc/cmdline

# Persisted in GRUB
sudo vi /etc/default/grub        # edit GRUB_CMDLINE_LINUX
sudo update-grub                 # Debian/Ubuntu
sudo grub2-mkconfig -o /boot/grub2/grub.cfg   # RHEL/Fedora

# systemd can also inject userspace env from the kernel cmdline:
# add to GRUB_CMDLINE_LINUX:  systemd.setenv=MYVAR=hello
```

`/etc/default/*` files (e.g., `/etc/default/locale`, `/etc/default/keyboard`) hold boot/system defaults read by init scripts.

---

## 3. Windows — where variables come from

### Default sources (in load order at logon)

1. **Registry — System scope:** `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Environment`
2. **Registry — User scope:** `HKEY_CURRENT_USER\Environment`
3. **Volatile / process scope:** what the current process / shell sets at runtime.

User overrides System for the **same** variable name — except `PATH`, which is **concatenated** (System PATH first, then User PATH).

### Where to add them (by intent)

| Goal | How | Scope |
| --- | --- | --- |
| One-off in current `cmd.exe` | `set VAR=value` | Process |
| One-off in current PowerShell | `$env:VAR = 'value'` | Process |
| Persistent for **my user** | `setx VAR "value"` (cmd) **or** GUI: *System Properties → Environment Variables → User* | User (Registry) |
| Persistent for **all users** | `setx /M VAR "value"` from an **elevated** prompt **or** GUI: *System variables* | Machine (Registry) |
| Quick GUI shortcut | `Win+R` → `sysdm.cpl` → *Advanced* → *Environment Variables…* | User or System |
| Run dialog shortcut | `Win+R` → `rundll32 sysdm.cpl,EditEnvironmentVariables` | Same dialog |

```powershell
# Process-scoped (only this session)
$env:APP_ENV = 'dev'

# User-scoped, persistent (no admin needed)
[Environment]::SetEnvironmentVariable('APP_ENV', 'dev', 'User')

# Machine-scoped, persistent (needs admin)
[Environment]::SetEnvironmentVariable('APP_ENV', 'production', 'Machine')

# Read across scopes
[Environment]::GetEnvironmentVariable('PATH', 'Machine')
[Environment]::GetEnvironmentVariable('PATH', 'User')
[Environment]::GetEnvironmentVariable('PATH', 'Process')

# Delete a persistent variable
[Environment]::SetEnvironmentVariable('APP_ENV', $null, 'User')
```

```cmd
:: cmd.exe
set VAR=value                  :: process only
setx VAR "value"               :: persistent, current user
setx VAR "value" /M            :: persistent, machine-wide (admin)
set VAR=                       :: unset for this session
reg delete "HKCU\Environment" /v VAR /f   :: delete persistent user var
```

> **Trick — `setx` truncates at 1024 chars.** For long `PATH` values, edit via the GUI or `[Environment]::SetEnvironmentVariable(...)` to avoid silent truncation.

### Why my new variable "doesn't show up"

`setx` writes to the registry but **does not refresh existing processes** (including the shell you ran it from, Explorer, your IDE). Three options:

1. **Open a new shell** — simplest.
2. **Broadcast a settings change** so Explorer picks it up without logoff:
   ```powershell
   # Force-refresh environment in Explorer (and new child processes it spawns)
   $sig = '[DllImport("user32.dll", SetLastError=true, CharSet=CharSet.Auto)] public static extern IntPtr SendMessageTimeout(IntPtr hWnd, uint Msg, UIntPtr wParam, string lParam, uint fuFlags, uint uTimeout, out UIntPtr lpdwResult);'
   $type = Add-Type -MemberDefinition $sig -Name NativeMethods -Namespace Win32 -PassThru
   [UIntPtr]$result = [UIntPtr]::Zero
   $type::SendMessageTimeout([IntPtr]0xffff, 0x1A, [UIntPtr]::Zero, 'Environment', 2, 5000, [ref]$result) | Out-Null
   ```
3. **Refresh only this shell** by re-reading the registry:
   ```powershell
   $env:PATH = [Environment]::GetEnvironmentVariable('PATH','Machine') + ';' + [Environment]::GetEnvironmentVariable('PATH','User')
   ```

### PATH hygiene on Windows

```powershell
# View System PATH as a list
[Environment]::GetEnvironmentVariable('PATH','Machine') -split ';'

# Append to User PATH safely (no duplicates, no truncation)
$user = [Environment]::GetEnvironmentVariable('PATH','User')
$add  = 'C:\tools\bin'
if (($user -split ';') -notcontains $add) {
    [Environment]::SetEnvironmentVariable('PATH', ($user.TrimEnd(';') + ';' + $add), 'User')
}
```

### Windows services

A service inherits the **System** environment as it was at *service start* — not at logon. After changing a Machine variable used by a service, **restart the service** (or reboot):

```powershell
Restart-Service -Name 'MyService'
```

For per-service variables without polluting machine scope, use the registry:

```
HKLM\SYSTEM\CurrentControlSet\Services\<ServiceName>\Environment   (REG_MULTI_SZ)
```

Each line is `KEY=VALUE`. Restart the service to pick them up.

### WSL ↔ Windows interop

```bash
# In WSL, Windows PATH is appended to Linux PATH by default.
# Disable it in /etc/wsl.conf if it slows things down:
sudo tee /etc/wsl.conf <<'EOF'
[interop]
appendWindowsPath = false
EOF
# then in PowerShell:  wsl --shutdown
```

You can pass variables across the boundary with `WSLENV`:

```cmd
set WSLENV=APP_ENV/u:JAVA_HOME/p
wsl printenv APP_ENV
```

Flags: `/u` = uppercase forwarding, `/p` = translate Windows path → Linux path, `/l` = path list translation.

### Boot-time on Windows

Real "boot variables" on Windows live in **BCD** (Boot Configuration Data), not in the environment:

```cmd
:: List entries (admin)
bcdedit /enum

:: Toggle safe boot
bcdedit /set {current} safeboot minimal
bcdedit /deletevalue {current} safeboot

:: Disable driver signature enforcement for this boot
bcdedit /set nointegritychecks on
```

UEFI firmware variables (the closest analogue to "boot env vars") are visible from PowerShell:

```powershell
Get-SecureBootPolicy
Get-SecureBootUEFI -Name PK    # requires admin
```

---

## 4. Cross-platform gotchas

- **Case sensitivity:** Linux is case-sensitive (`$Path` ≠ `$PATH`). Windows is case-insensitive.
- **Separator:** Linux uses `:` in `PATH`. Windows uses `;`.
- **Quoting on Windows:** `setx PATH "%PATH%;C:\tools"` will expand `%PATH%` *now* and bake the result in — usually not what you want. Prefer the PowerShell `[Environment]` API.
- **`.env` files:** Not read by the OS — only by the app/runtime (dotenv, docker, vite, dotnet). Don't expect `cat /etc/environment` semantics from them.
- **Docker:** `-e KEY=value`, `--env-file path.env`, or `ENV` in the Dockerfile. The container has its **own** environment; the host's is not inherited.
- **CI/CD:** GitHub Actions uses `echo "KEY=value" >> $GITHUB_ENV` to expose a var to *subsequent* steps in the same job.

---

## 5. Quick reference — verify what a process actually sees

```bash
# Linux
cat /proc/$(pgrep -f myapp | head -1)/environ | tr '\0' '\n'

# Windows (PowerShell, requires Sysinternals Process Explorer or:)
Get-Process myapp | ForEach-Object {
    (Get-Process -Id $_.Id).StartInfo.EnvironmentVariables
}
```

When in doubt: launch a **brand new** terminal, then inspect — that rules out stale shell state every time.
