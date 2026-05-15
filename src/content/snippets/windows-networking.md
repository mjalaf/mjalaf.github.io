---
title: "Windows Networking"
description: "Networking commands and PowerShell cmdlets for Windows: DNS, firewall, TCP/IP, TLS, captures, and connectivity troubleshooting."
category: "Networking"
---

## DNS & Resolution

```powershell
# Resolve a name (replaces nslookup for modern usage)
Resolve-DnsName example.com
Resolve-DnsName example.com -Type MX
Resolve-DnsName example.com -Type TXT
Resolve-DnsName example.com -Server 8.8.8.8
Resolve-DnsName 8.8.8.8 -Type PTR        # reverse lookup

# Classic nslookup (still useful)
nslookup example.com
nslookup -type=MX example.com 8.8.8.8

# View / clear the DNS resolver cache
Get-DnsClientCache
Clear-DnsClientCache
ipconfig /displaydns
ipconfig /flushdns

# Current DNS servers per adapter
Get-DnsClientServerAddress | Format-Table InterfaceAlias,AddressFamily,ServerAddresses
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 1.1.1.1,8.8.8.8

# HOSTS file
notepad C:\Windows\System32\drivers\etc\hosts
```

## IP configuration

```powershell
# Quick view
ipconfig /all

# PowerShell equivalents (more scriptable)
Get-NetIPConfiguration
Get-NetIPAddress -AddressFamily IPv4 | Format-Table InterfaceAlias,IPAddress,PrefixLength
Get-NetAdapter | Format-Table Name,Status,MacAddress,LinkSpeed
Get-NetRoute -AddressFamily IPv4 | Sort-Object InterfaceMetric

# Renew DHCP lease
ipconfig /release
ipconfig /renew

# Add a static route (survives reboot with -p)
route -p add 10.20.0.0 mask 255.255.0.0 192.168.1.1
route delete 10.20.0.0

# Reset the TCP/IP stack (last-resort fix for broken networking)
netsh int ip reset
netsh winsock reset
# Reboot required
```

## Connectivity testing

```powershell
# TCP port test (replaces telnet for "is the port open?")
Test-NetConnection example.com -Port 443
Test-NetConnection example.com -Port 443 -InformationLevel Detailed

# Ping with continuous + timestamp
ping -t example.com
Test-Connection example.com -Count 4 -BufferSize 1400

# Traceroute
tracert example.com
Test-NetConnection example.com -TraceRoute

# pathping (combines ping + tracert + per-hop loss stats)
pathping example.com

# HTTP timing with curl (curl ships with Windows 10+ / Server 2019+)
curl.exe -w "`n  DNS: %{time_namelookup}`n  Connect: %{time_connect}`n  TLS: %{time_appconnect}`n  Total: %{time_total}`n" -o NUL -s https://example.com

# Invoke-WebRequest with timing
Measure-Command { Invoke-WebRequest https://example.com -UseBasicParsing }
```

## Listening ports & connections

```powershell
# All TCP/UDP connections + owning process
netstat -anob          # requires elevated shell
netstat -ano | findstr :443

# PowerShell equivalents
Get-NetTCPConnection -State Listen |
  Sort-Object LocalPort |
  Select-Object LocalAddress,LocalPort,OwningProcess,
                @{n='Process';e={(Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue).ProcessName}}

Get-NetUDPEndpoint | Sort-Object LocalPort

# Kill the process holding a port
$pid = (Get-NetTCPConnection -LocalPort 8080 -State Listen).OwningProcess
Stop-Process -Id $pid -Force
```

## Windows Defender Firewall

```powershell
# Profiles (Domain / Private / Public)
Get-NetFirewallProfile | Format-Table Name,Enabled,DefaultInboundAction,DefaultOutboundAction
Set-NetFirewallProfile -Profile Public -Enabled True

# List rules
Get-NetFirewallRule | Where-Object Enabled -eq True | Sort-Object DisplayName | Select-Object DisplayName,Direction,Action

# Find rules for a specific port
Get-NetFirewallPortFilter -Protocol TCP |
  Where-Object LocalPort -eq 3389 |
  Get-NetFirewallRule

# Create a rule (allow inbound HTTPS)
New-NetFirewallRule -DisplayName "Allow HTTPS Inbound" `
  -Direction Inbound -Protocol TCP -LocalPort 443 -Action Allow `
  -Profile Domain,Private

# Block an outbound IP
New-NetFirewallRule -DisplayName "Block bad host" `
  -Direction Outbound -RemoteAddress 1.2.3.4 -Action Block

# Disable / remove
Disable-NetFirewallRule -DisplayName "Allow HTTPS Inbound"
Remove-NetFirewallRule  -DisplayName "Allow HTTPS Inbound"

# Classic netsh equivalents (older scripts)
netsh advfirewall firewall add rule name="Allow 443" dir=in action=allow protocol=TCP localport=443
netsh advfirewall firewall show rule name=all
netsh advfirewall set allprofiles state on
```

## TLS / SSL

```powershell
# Inspect remote certificate (PowerShell — no openssl required)
$req = [Net.HttpWebRequest]::Create("https://example.com")
$req.GetResponse() | Out-Null
$cert = $req.ServicePoint.Certificate
[System.Security.Cryptography.X509Certificates.X509Certificate2]$cert | Format-List Subject,Issuer,NotBefore,NotAfter,Thumbprint

# With openssl (if installed / Git for Windows / WSL)
echo "Q" | openssl s_client -connect example.com:443 -servername example.com 2>$null | openssl x509 -noout -dates

# Which TLS protocols are enabled (system-wide registry)
Get-ChildItem 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols' -Recurse |
  Get-ItemProperty | Select-Object PSPath,Enabled,DisabledByDefault

# Force .NET / PowerShell to use TLS 1.2 (legacy systems)
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
```

## Packet captures

```powershell
# pktmon (built-in since Windows 10 1809 / Server 2019)
pktmon filter add HTTPS -p 443
pktmon start --capture --pkt-size 0 -f C:\temp\capture.etl
# ... reproduce the issue ...
pktmon stop
pktmon etl2pcap C:\temp\capture.etl -o C:\temp\capture.pcap   # open in Wireshark
pktmon filter remove

# netsh trace (older but very portable)
netsh trace start capture=yes tracefile=C:\temp\nettrace.etl maxsize=200
# ... reproduce ...
netsh trace stop
#   convert with Microsoft Message Analyzer or etl2pcapng

# Wireshark / tshark
tshark -i "Ethernet" -f "tcp port 443" -w C:\temp\capture.pcap
```

## Hyper-V / WSL networking

```powershell
# List virtual switches
Get-VMSwitch

# WSL networking is NAT'd by default (Windows 11 + WSL2)
wsl hostname -I                          # IP of the WSL2 VM
Get-NetIPConfiguration -InterfaceAlias 'vEthernet (WSL)'

# Port-forward a Windows host port into WSL2 (Linux process listening inside)
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=8080 connectaddress=(wsl hostname -I)
netsh interface portproxy show all
netsh interface portproxy delete v4tov4 listenport=8080 listenaddress=0.0.0.0
```

## SMB / file shares

```powershell
Get-SmbConnection
Get-SmbShare
Get-SmbOpenFile
Test-NetConnection fileserver -Port 445

# Map a network drive (persistent)
New-PSDrive -Name Z -PSProvider FileSystem -Root \\fileserver\share -Persist -Credential (Get-Credential)
net use Z: \\fileserver\share /persistent:yes
```

## Azure-specific from Windows

```powershell
# Test a private endpoint resolves to a private IP (not the public one)
Resolve-DnsName mystorage.blob.core.windows.net
Resolve-DnsName mystorage.privatelink.blob.core.windows.net

# Quick reachability test from a VM to a PaaS service
Test-NetConnection mystorage.blob.core.windows.net -Port 443

# Effective route table for a NIC (run on host with Azure CLI)
az network nic show-effective-route-table -g myRG -n myNIC -o table
```

## Quick troubleshooting checklist

```powershell
# 1. Layer 3 — is the host reachable?
Test-Connection example.com -Count 2

# 2. Layer 4 — is the port open?
Test-NetConnection example.com -Port 443

# 3. DNS — does it resolve to what you expect?
Resolve-DnsName example.com

# 4. Local listener — is something actually listening?
Get-NetTCPConnection -LocalPort 443 -State Listen

# 5. Firewall — is a rule blocking it?
Get-NetFirewallRule | Where-Object { $_.Enabled -eq 'True' -and $_.Action -eq 'Block' }

# 6. Routing — which interface and gateway?
Find-NetRoute -RemoteIPAddress 8.8.8.8

# 7. App layer — does HTTPS actually complete?
Invoke-WebRequest https://example.com -UseBasicParsing | Select StatusCode,StatusDescription
```
