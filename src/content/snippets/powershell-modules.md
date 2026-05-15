---
title: "PowerShell Popular Modules"
description: "Install and use the most popular PowerShell modules for Azure, Microsoft 365, AD, Git, JSON/YAML, and developer productivity — with handy tricks."
category: "PowerShell"
---

## Before installing anything

```powershell
# What version am I on? (PS 7+ recommended)
$PSVersionTable

# Install PowerShell 7 on Windows (winget) — runs side-by-side with Windows PowerShell 5.1
winget install --id Microsoft.PowerShell --source winget

# Trust the PSGallery so installs don't keep prompting
Set-PSRepository -Name PSGallery -InstallationPolicy Trusted

# Always install to the current user (no admin needed)
$PSDefaultParameterValues['Install-Module:Scope']  = 'CurrentUser'
$PSDefaultParameterValues['Install-Module:Force']  = $true
$PSDefaultParameterValues['Install-Module:AllowClobber'] = $true

# Modern package manager (replaces PowerShellGet for new modules)
Install-Module Microsoft.PowerShell.PSResourceGet -Scope CurrentUser
#   ...then use: Install-PSResource <Name>
```

> **Trick:** put the `$PSDefaultParameterValues` lines in your `$PROFILE` so every new install defaults to `CurrentUser` + `Force`. Edit your profile with `code $PROFILE` (creates it if missing).

## Azure & cloud

```powershell
# Az — the modern, cross-platform Azure module (replaces AzureRM, which is EOL)
Install-Module Az -Scope CurrentUser
Connect-AzAccount
Set-AzContext -Subscription "Visual Studio Enterprise"

#   Install only the sub-modules you need (much faster, smaller footprint)
Install-Module Az.Accounts, Az.Resources, Az.Compute, Az.Storage, Az.KeyVault -Scope CurrentUser

# Microsoft Graph — modern replacement for AzureAD / MSOnline (both deprecated)
Install-Module Microsoft.Graph -Scope CurrentUser
Connect-MgGraph -Scopes "User.Read.All","Group.Read.All"

# Microsoft Graph PowerShell SDK — Beta endpoints
Install-Module Microsoft.Graph.Beta -Scope CurrentUser

# Microsoft Entra (formerly Azure AD) — modern module
Install-Module Microsoft.Graph.Entra -Scope CurrentUser

# AWS Tools (modular like Az)
Install-Module AWS.Tools.Installer -Scope CurrentUser
Install-AWSToolsModule AWS.Tools.EC2,AWS.Tools.S3 -CleanUp
```

> **Trick:** when working with Az, `Connect-AzAccount -DeviceCode` lets you log in from a headless terminal (server, SSH, container) by entering a code in a browser elsewhere.

## Microsoft 365 / collaboration

```powershell
# Exchange Online
Install-Module ExchangeOnlineManagement -Scope CurrentUser
Connect-ExchangeOnline -UserPrincipalName admin@contoso.com

# Microsoft Teams
Install-Module MicrosoftTeams -Scope CurrentUser
Connect-MicrosoftTeams

# SharePoint Online (legacy admin module)
Install-Module Microsoft.Online.SharePoint.PowerShell -Scope CurrentUser
Connect-SPOService -Url https://contoso-admin.sharepoint.com

# PnP PowerShell — modern, far more powerful for SharePoint/Teams/Graph
Install-Module PnP.PowerShell -Scope CurrentUser
Connect-PnPOnline -Url https://contoso.sharepoint.com -Interactive

# Microsoft Intune / Endpoint Manager
Install-Module Microsoft.Graph.Intune -Scope CurrentUser
```

## Active Directory & on-prem

```powershell
# RSAT for Active Directory module (Windows 10/11)
Add-WindowsCapability -Online -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0

# Then it's available:
Import-Module ActiveDirectory
Get-ADUser -Filter * -SearchBase "OU=Users,DC=contoso,DC=com"

# Group Policy
Import-Module GroupPolicy
Get-GPO -All
```

## DevOps, Git, and source control

```powershell
# Posh-Git — branch / status in your prompt + tab completion for git
Install-Module posh-git -Scope CurrentUser
Add-PoshGitToProfile -AllHosts

# GitHub CLI (install + then auth)
winget install GitHub.cli
gh auth login

# Azure DevOps REST helpers
Install-Module VSTeam -Scope CurrentUser
Set-VSTeamAccount -Account contoso -PersonalAccessToken $env:AZP_TOKEN

# PSGitHub — typed wrapper around the GitHub API
Install-Module PSGitHub -Scope CurrentUser
```

## Data, files, and serialization

```powershell
# Excel without Excel installed
Install-Module ImportExcel -Scope CurrentUser
Get-Process | Sort-Object CPU -Desc | Select -First 20 |
  Export-Excel ./top.xlsx -AutoSize -AutoFilter -BoldTopRow

# YAML
Install-Module powershell-yaml -Scope CurrentUser
$config = ConvertFrom-Yaml (Get-Content ./config.yaml -Raw)
$config | ConvertTo-Yaml | Set-Content ./out.yaml

# Pretty SQL access
Install-Module dbatools -Scope CurrentUser              # SQL Server: thousands of commands
Install-Module SimplySql -Scope CurrentUser             # MSSQL / PostgreSQL / MySQL / Oracle

# JSON path / jq-like queries
Install-Module Newtonsoft.Json -Scope CurrentUser
```

> **Trick:** `Get-Process | ConvertTo-Json -Depth 4 | Out-File ps.json` then open in VS Code — best ad-hoc inspection of any cmdlet output.

## Terminal UX & productivity

```powershell
# Oh-My-Posh — themed prompt with git/k8s/az/aws indicators
winget install JanDeDobbeleer.OhMyPosh
oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH\paradox.omp.json" | Invoke-Expression
#   Add the line above to your $PROFILE to make it persistent.

# PSReadLine — predictive IntelliSense + fish-style history
Install-Module PSReadLine -AllowPrerelease -Force -Scope CurrentUser
Set-PSReadLineOption -PredictionSource HistoryAndPlugin -PredictionViewStyle ListView -EditMode Windows

# Terminal-Icons — file-type icons in `Get-ChildItem`
Install-Module Terminal-Icons -Scope CurrentUser
Import-Module Terminal-Icons

# z — fast directory jumping (like autojump)
Install-Module z -Scope CurrentUser

# PSFzf — fuzzy file/history/process picker (requires fzf binary)
winget install junegunn.fzf
Install-Module PSFzf -Scope CurrentUser
Set-PsFzfOption -PSReadlineChordProvider 'Ctrl+t' -PSReadlineChordReverseHistory 'Ctrl+r'
```

> **Trick (PSReadLine):** with PredictionView set to `ListView`, start typing any past command and press `→` to accept the inline suggestion, or `F2` to toggle between inline and dropdown.

## Cross-platform must-haves

```powershell
# Pester — the standard test framework
Install-Module Pester -Scope CurrentUser -Force -SkipPublisherCheck

# PSScriptAnalyzer — lint your scripts
Install-Module PSScriptAnalyzer -Scope CurrentUser
Invoke-ScriptAnalyzer -Path .\scripts -Recurse

# platyPS — generate help/markdown from comment-based help
Install-Module platyPS -Scope CurrentUser

# Plaster — scaffolding (project templates)
Install-Module Plaster -Scope CurrentUser
```

## Security & secrets

```powershell
# SecretManagement + SecretStore — replacement for plaintext creds in scripts
Install-Module Microsoft.PowerShell.SecretManagement -Scope CurrentUser
Install-Module Microsoft.PowerShell.SecretStore       -Scope CurrentUser
Register-SecretVault -Name LocalStore -ModuleName Microsoft.PowerShell.SecretStore -DefaultVault
Set-Secret -Name AzpToken -Secret 'xxxxxxxxxxxx'
Get-Secret  -Name AzpToken -AsPlainText

# Az.KeyVault provider for SecretManagement
Install-Module Az.KeyVault -Scope CurrentUser
Register-SecretVault -Name 'kv-prod' -ModuleName Az.KeyVault -VaultParameters @{
  AZKVaultName  = 'contoso-prod-kv'
  SubscriptionId = '00000000-0000-0000-0000-000000000000'
}
```

> **Trick:** combine with `$PSDefaultParameterValues` — e.g.
> `$PSDefaultParameterValues['Invoke-RestMethod:Headers'] = @{ Authorization = "Bearer $(Get-Secret AzpToken -AsPlainText)" }` so every API call is auto-authenticated.

## HTTP / API exploration

```powershell
# Built-in is excellent — no module needed:
Invoke-RestMethod -Uri https://api.github.com/repos/mjalaf/mjalaf.github.io |
  Select-Object name, stargazers_count, updated_at

# But these add structure:
Install-Module PSGraphQL -Scope CurrentUser           # GraphQL clients
Install-Module Microsoft.PowerShell.ConsoleGuiTools   # Out-ConsoleGridView (interactive!)
Get-Process | Out-ConsoleGridView -OutputMode Multiple
```

## Kubernetes, Docker, IaC

```powershell
# Kubernetes — wrappers over kubectl
Install-Module PSKubectl -Scope CurrentUser

# Docker (CLI is recommended; this module is for scripting interactions)
Install-Module PSDocker -Scope CurrentUser

# Terraform / IaC linters: install the binaries (winget) instead, then orchestrate from PS
winget install Hashicorp.Terraform
winget install Bicep
```

## Module management cheats

```powershell
# Find / list / update / uninstall
Find-Module *graph* -Tag azure
Get-InstalledModule
Get-Module -ListAvailable | Group-Object Name | Where-Object Count -gt 1   # duplicates!
Update-Module Az -Scope CurrentUser
Uninstall-Module -Name AzureRM -AllVersions

# Pin a version (great for CI reproducibility)
Install-Module Az -RequiredVersion 11.4.0 -Scope CurrentUser

# Save a module offline (no internet on the target machine)
Save-Module Az -Path C:\offline-modules
#   Copy to target, then: Copy-Item -Recurse C:\offline-modules\* "$env:USERPROFILE\Documents\PowerShell\Modules\"

# What does this module ship?
Get-Command -Module Az.Compute | Measure-Object
Get-Command -Module Az.Compute | Out-ConsoleGridView
```

> **Trick — "where is this command coming from?":**
> ```powershell
> (Get-Command Connect-AzAccount).Source
> (Get-Command Connect-AzAccount).Module.Path
> ```
> Indispensable when two modules expose the same verb-noun and you're not sure which wins.

## Profile starter (copy into `$PROFILE`)

```powershell
# Sensible defaults
Set-PSReadLineOption -PredictionSource HistoryAndPlugin -PredictionViewStyle ListView
$PSDefaultParameterValues['Install-Module:Scope'] = 'CurrentUser'
$PSDefaultParameterValues['Out-Default:OutVariable'] = 'LastOutput'   # always have $LastOutput

# Prompt + icons
Import-Module Terminal-Icons
oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH\paradox.omp.json" | Invoke-Expression

# Quality-of-life aliases
Set-Alias which Get-Command
Set-Alias touch New-Item
function .. { Set-Location .. }
function ... { Set-Location ../.. }

# Reload profile without opening a new shell
function Reload-Profile { . $PROFILE }
```

> **Trick:** every cmdlet output is captured in `$LastOutput` (because of the `Out-Default` default above), so after running anything you can `$LastOutput | Where-Object {...}` without re-running the command.
