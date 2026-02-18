---
title: "Networking Fundamentals"
description: "Networking commands and configurations for DNS, firewalls, load balancers, and troubleshooting connectivity."
category: "Networking"
---

## DNS & Resolution

```bash
# Dig with specific record types
dig example.com A +short
dig example.com MX +short
dig example.com TXT +short
dig example.com CNAME +short

# Reverse DNS lookup
dig -x 8.8.8.8

# Query a specific DNS server
dig @8.8.8.8 example.com

# Trace DNS resolution path
dig +trace example.com

# Flush DNS cache (Linux systemd-resolved)
sudo resolvectl flush-caches

# Check current DNS config
resolvectl status
cat /etc/resolv.conf
```

## Connectivity Testing

```bash
# Test TCP connectivity (without telnet)
nc -zv hostname 443
echo | openssl s_client -connect hostname:443 -servername hostname 2>/dev/null | head -5

# Test with curl (verbose + timing)
curl -w "\n  DNS: %{time_namelookup}\n  Connect: %{time_connect}\n  TLS: %{time_appconnect}\n  Total: %{time_total}\n" -o /dev/null -s https://example.com

# Continuous ping with timestamp
ping -D -i 2 hostname

# MTR (traceroute + ping combined)
mtr --report --report-cycles 10 hostname

# Test specific source interface
curl --interface eth0 https://example.com
```

## Firewall (iptables / nftables)

```bash
# List current rules
sudo iptables -L -n -v --line-numbers

# Allow inbound on port 443
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Block an IP
sudo iptables -A INPUT -s 1.2.3.4 -j DROP

# Allow established connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Save rules (Debian/Ubuntu)
sudo iptables-save > /etc/iptables/rules.v4

# UFW (simplified)
sudo ufw allow 22/tcp
sudo ufw allow from 10.0.0.0/8 to any port 8080
sudo ufw status verbose
```

## SSL/TLS

```bash
# Check certificate details
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null | openssl x509 -noout -text

# Check certificate expiry
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null | openssl x509 -noout -dates

# Generate self-signed certificate
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 365 -nodes \
  -subj "/CN=localhost"

# Test TLS version support
openssl s_client -connect example.com:443 -tls1_3
openssl s_client -connect example.com:443 -tls1_2
```

## Azure Network Troubleshooting

```bash
# NSG effective rules
az network nic show-effective-route-table -g myRG -n myNIC -o table

# Check NSG flow logs
az network watcher flow-log show -g myRG -n myFlowLog

# Test IP flow (check if traffic is allowed)
az network watcher test-ip-flow \
  --direction Inbound \
  --protocol TCP \
  --local 10.0.1.4:80 \
  --remote 203.0.113.5:* \
  --vm myVM \
  -g myRG

# DNS resolution from VNet
az network dns record-set list -g myRG -z example.com -o table

# List private endpoints
az network private-endpoint list -g myRG -o table
```

## tcpdump

```bash
# Capture traffic on port 443
sudo tcpdump -i eth0 port 443 -w capture.pcap

# Capture specific host traffic
sudo tcpdump -i any host 10.0.1.5 -nn

# Capture DNS traffic
sudo tcpdump -i any port 53 -nn

# Read a capture file
tcpdump -r capture.pcap -nn | head -50
```
