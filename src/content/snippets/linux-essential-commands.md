---
title: "Essential Linux Commands"
description: "Common Linux commands for file management, process control, disk usage, and system administration."
category: "Linux"
---

## File & Directory Operations

```bash
# Find files modified in the last 24 hours
find /var/log -type f -mtime -1

# Find and delete files older than 30 days
find /tmp -type f -mtime +30 -delete

# Recursively search for text in files
grep -rn "search_term" /path/to/dir/

# Compare two directories
diff -rq dir1/ dir2/

# Rsync with progress and compression
rsync -avz --progress source/ user@host:/destination/
```

## Process Management

```bash
# Find process by name
ps aux | grep process_name

# Kill all processes matching a name
pkill -f "process_name"

# Run command immune to hangups (persists after logout)
nohup long_running_command &

# Monitor system resources in real-time
top -o %MEM    # sort by memory
htop           # interactive (if installed)

# List open files by process
lsof -p <PID>

# List processes using a specific port
lsof -i :8080
```

## Disk & Storage

```bash
# Disk usage summary (human readable, sorted)
du -sh * | sort -rh | head -20

# Check disk space
df -h

# Find large files (>100MB)
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null

# Check inode usage
df -i
```

## Networking

```bash
# Test connectivity
curl -vvv https://example.com

# DNS lookup
dig example.com +short
nslookup example.com

# Check listening ports
ss -tulnp
netstat -tulnp

# Download file with progress
wget -c --show-progress https://example.com/file.tar.gz

# Trace route
traceroute example.com
```

## User & Permissions

```bash
# Add user to a group
sudo usermod -aG groupname username

# Set permissions recursively
chmod -R 755 /path/to/dir
chown -R user:group /path/to/dir

# Find files with specific permissions
find / -perm -4000 -type f 2>/dev/null  # SUID files
```

## Systemd Services

```bash
# Check service status
systemctl status service_name

# Enable and start a service
sudo systemctl enable --now service_name

# View service logs
journalctl -u service_name -f --since "1 hour ago"

# List failed services
systemctl --failed
```
