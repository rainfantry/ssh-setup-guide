# Linux SSH Server Setup & Troubleshooting Guide

Complete guide for setting up, checking, and fixing SSH server on Linux with key-based authentication.

## Table of Contents
1. [Initial Setup](#initial-setup)
2. [SSH Service Management](#ssh-service-management)
3. [Key-Based Authentication](#key-based-authentication)
4. [Firewall Configuration](#firewall-configuration)
5. [Troubleshooting](#troubleshooting)
6. [Common Commands Reference](#common-commands-reference)

---

## Initial Setup

### Check if SSH is Installed

```bash
# Check SSH server status
systemctl status sshd
# OR (on some distros)
systemctl status ssh

# Check OpenSSH version
ssh -V

# Check if SSH daemon is installed
which sshd
```

### Install OpenSSH Server

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install openssh-server
```

**CentOS/RHEL/Fedora:**
```bash
sudo dnf install openssh-server
# OR (older versions)
sudo yum install openssh-server
```

**Arch Linux:**
```bash
sudo pacman -S openssh
```

---

## SSH Service Management

### Enable and Start SSH Service

```bash
# Enable SSH to start on boot
sudo systemctl enable sshd

# Start SSH service
sudo systemctl start sshd

# Enable and start in one command
sudo systemctl enable --now sshd

# Verify service is running
sudo systemctl status sshd
```

### Check SSH Server Status

```bash
# Service status
sudo systemctl status sshd

# Check if SSH is listening on port 22
sudo ss -tlnp | grep :22
# OR
sudo netstat -tlnp | grep :22

# Should show:
# tcp   LISTEN  0.0.0.0:22
# tcp6  LISTEN  :::22
```

### Restart SSH Service

```bash
# Standard restart
sudo systemctl restart sshd

# Reload configuration without dropping connections
sudo systemctl reload sshd

# Stop and start
sudo systemctl stop sshd
sudo systemctl start sshd
```

---

## Key-Based Authentication

### Understanding Linux SSH Key Locations

**User-specific authorized keys:**
- Location: `~/.ssh/authorized_keys`
- Permissions: `600` (read/write for owner only)
- Directory `~/.ssh/` permissions: `700`

**System-wide (not recommended):**
- Location: `/etc/ssh/authorized_keys/<username>`

### Generate SSH Key Pair

```bash
# Generate ED25519 key (recommended - most secure)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Generate RSA key (compatible with older systems)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Default location: ~/.ssh/id_ed25519 (or id_rsa)
# Press Enter for no passphrase (or set one for extra security)
```

### View Your Public Key

```bash
# Display public key
cat ~/.ssh/id_ed25519.pub

# Copy to clipboard (if xclip installed)
cat ~/.ssh/id_ed25519.pub | xclip -selection clipboard

# Copy to clipboard (if xsel installed)
cat ~/.ssh/id_ed25519.pub | xsel --clipboard
```

### Add Public Key to Authorized Keys

**Method 1: Manual (from the server)**

```bash
# Create .ssh directory if it doesn't exist
mkdir -p ~/.ssh

# Set correct permissions
chmod 700 ~/.ssh

# Add public key (single key)
echo "ssh-ed25519 AAAA... user@host" >> ~/.ssh/authorized_keys

# Add multiple keys at once
cat >> ~/.ssh/authorized_keys << 'EOF'
ssh-ed25519 AAAA...key1... pc-key
ssh-ed25519 AAAA...key2... phone-key
ssh-ed25519 AAAA...key3... laptop-key
EOF

# Set correct permissions on authorized_keys
chmod 600 ~/.ssh/authorized_keys

# Verify contents
cat ~/.ssh/authorized_keys
```

**Method 2: Using ssh-copy-id (from client)**

```bash
# Copy your public key to remote server
ssh-copy-id username@remote-host

# Specify key file
ssh-copy-id -i ~/.ssh/id_ed25519.pub username@remote-host

# Specify custom port
ssh-copy-id -i ~/.ssh/id_ed25519.pub -p 2222 username@remote-host
```

**Method 3: Manual copy from client**

```bash
# Copy key to remote server
cat ~/.ssh/id_ed25519.pub | ssh username@remote-host "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

### Configure SSH Server for Key-Only Authentication

```bash
# Edit SSH config
sudo nano /etc/ssh/sshd_config
# OR
sudo vim /etc/ssh/sshd_config

# Find and modify these lines (remove # to uncomment):
PubkeyAuthentication yes
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM no

# Optional security enhancements:
PermitRootLogin no
MaxAuthTries 3
MaxSessions 2
```

**Quick configuration:**

```bash
# Backup original config
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# Disable password authentication
sudo sed -i 's/^#*PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config

# Enable public key authentication
sudo sed -i 's/^#*PubkeyAuthentication.*/PubkeyAuthentication yes/' /etc/ssh/sshd_config

# Disable root login
sudo sed -i 's/^#*PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config

# Verify changes
grep -E "PasswordAuthentication|PubkeyAuthentication|PermitRootLogin" /etc/ssh/sshd_config

# Test configuration
sudo sshd -t

# Restart SSH service
sudo systemctl restart sshd
```

### Verify SSH Configuration

```bash
# Check authentication settings
grep -E "^PasswordAuthentication|^PubkeyAuthentication" /etc/ssh/sshd_config

# View authorized keys
cat ~/.ssh/authorized_keys

# Check permissions
ls -la ~/.ssh/
ls -l ~/.ssh/authorized_keys

# Should show:
# drwx------ .ssh
# -rw------- authorized_keys

# Test configuration syntax
sudo sshd -t
```

---

## Firewall Configuration

### UFW (Ubuntu/Debian)

```bash
# Check UFW status
sudo ufw status

# Allow SSH
sudo ufw allow ssh
# OR specify port
sudo ufw allow 22/tcp

# Enable firewall
sudo ufw enable

# Verify rule
sudo ufw status numbered
```

### firewalld (CentOS/RHEL/Fedora)

```bash
# Check firewalld status
sudo firewall-cmd --state

# Allow SSH (permanent)
sudo firewall-cmd --permanent --add-service=ssh

# Allow specific port
sudo firewall-cmd --permanent --add-port=22/tcp

# Reload firewall
sudo firewall-cmd --reload

# Verify rules
sudo firewall-cmd --list-all
```

### iptables (Manual)

```bash
# Check current rules
sudo iptables -L -n -v

# Allow SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Save rules (Ubuntu/Debian)
sudo iptables-save | sudo tee /etc/iptables/rules.v4

# Save rules (CentOS/RHEL)
sudo iptables-save | sudo tee /etc/sysconfig/iptables
```

### SELinux (CentOS/RHEL/Fedora)

```bash
# Check SELinux status
sestatus

# Allow SSH in SELinux (usually allowed by default)
sudo semanage port -a -t ssh_port_t -p tcp 22

# Check SSH context
ls -Z ~/.ssh/
```

---

## Troubleshooting

### Find Your IP Address

```bash
# All IP addresses
ip addr show
# OR
ifconfig

# Just IPv4 addresses
ip -4 addr show | grep inet

# Public IP (from internet)
curl ifconfig.me
curl icanhazip.com
```

### View SSH Logs

**Ubuntu/Debian:**
```bash
# Real-time log monitoring
sudo tail -f /var/log/auth.log

# Last 50 SSH-related lines
sudo grep sshd /var/log/auth.log | tail -50

# Search for specific user
sudo grep "sshd.*username" /var/log/auth.log
```

**CentOS/RHEL/Fedora:**
```bash
# Using journalctl
sudo journalctl -u sshd -f

# Last 50 entries
sudo journalctl -u sshd -n 50

# Today's logs
sudo journalctl -u sshd --since today

# Specific time range
sudo journalctl -u sshd --since "2026-02-14 08:00:00" --until "2026-02-14 09:00:00"
```

### Common Error Messages and Fixes

#### "Permission denied (publickey)"

**Cause:** Key mismatch, wrong permissions, or config issue

**Fix:**
```bash
# Check SSH logs on server
sudo tail -20 /var/log/auth.log | grep sshd

# Verify authorized_keys permissions
ls -la ~/.ssh/authorized_keys
# Should be: -rw------- (600)

# Fix permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# Verify key is in authorized_keys
cat ~/.ssh/authorized_keys

# Check SSH config
grep -E "PubkeyAuthentication|AuthorizedKeysFile" /etc/ssh/sshd_config

# Restart SSH
sudo systemctl restart sshd
```

#### "Connection refused"

**Cause:** SSH service not running or firewall blocking

**Fix:**
```bash
# Check if SSH is running
sudo systemctl status sshd

# Start SSH if stopped
sudo systemctl start sshd

# Check if port 22 is listening
sudo ss -tlnp | grep :22

# Check firewall (UFW)
sudo ufw status
sudo ufw allow ssh

# Check firewall (firewalld)
sudo firewall-cmd --list-all
sudo firewall-cmd --add-service=ssh --permanent
sudo firewall-cmd --reload
```

#### "Too many authentication failures"

**Cause:** Too many keys in ssh-agent

**Fix:**
```bash
# From client, specify exact key
ssh -i ~/.ssh/id_ed25519 username@host

# Or reduce keys in ssh-agent
ssh-add -l  # List keys
ssh-add -D  # Remove all keys
ssh-add ~/.ssh/id_ed25519  # Add only needed key
```

#### "Host key verification failed"

**Cause:** Server's host key changed

**Fix:**
```bash
# Remove old host key from client
ssh-keygen -R hostname_or_ip

# Example
ssh-keygen -R 192.168.1.40
```

#### "No supported authentication methods available"

**Cause:** Server only allows methods client doesn't have

**Fix:**
```bash
# On server, check allowed authentication
grep -E "AuthenticationMethods|PubkeyAuthentication|PasswordAuthentication" /etc/ssh/sshd_config

# Enable public key auth
sudo sed -i 's/^#*PubkeyAuthentication.*/PubkeyAuthentication yes/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

### Test SSH Locally

```bash
# Test from local machine
ssh username@localhost

# Test with verbose output
ssh -v username@localhost
ssh -vv username@localhost  # Very verbose
ssh -vvv username@localhost  # Maximum verbosity

# Test specific key
ssh -i ~/.ssh/id_ed25519 username@host
```

### Debug Mode

```bash
# Start SSH daemon in debug mode (kills current sshd!)
sudo /usr/sbin/sshd -d -p 2222

# Connect from another terminal
ssh -p 2222 username@localhost

# Watch debug output in first terminal
```

### Check File Permissions

```bash
# Verify all SSH-related permissions
ls -la ~/ | grep .ssh
ls -la ~/.ssh/

# Should show:
# drwx------  .ssh (700)
# -rw-------  authorized_keys (600)
# -rw-------  id_ed25519 (600)
# -rw-r--r--  id_ed25519.pub (644)

# Fix all permissions at once
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

---

## Common Commands Reference

### Quick Status Check

```bash
# Check everything at once
sudo systemctl status sshd
sudo ss -tlnp | grep :22
ip -4 addr show | grep inet
sudo ufw status  # OR: sudo firewall-cmd --list-all
```

### Connection Info Summary

```bash
# Your connection details:
echo "Hostname: $(hostname)"
echo "IP: $(ip -4 addr show | grep -oP '(?<=inet\s)\d+(\.\d+){3}' | grep -v 127.0.0.1 | head -1)"
echo "Port: 22"
echo "Username: $USER"
```

### Mobile SSH Client (Termius) Setup

1. **Host:** Your server's IP address
2. **Port:** 22 (or custom if changed)
3. **Username:** Your Linux username
4. **Authentication:** Key
5. **Key:** Import or paste your private key

**Export private key:**
```bash
cat ~/.ssh/id_ed25519
```

Copy the entire output (including BEGIN/END lines) to your mobile SSH client.

---

## Security Best Practices

### 1. Disable Password Authentication

```bash
sudo sed -i 's/^#*PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

### 2. Disable Root Login

```bash
sudo sed -i 's/^#*PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

### 3. Change Default SSH Port (Optional)

```bash
# Edit SSH config
sudo sed -i 's/^#*Port.*/Port 2222/' /etc/ssh/sshd_config

# Update firewall
sudo ufw allow 2222/tcp
sudo ufw delete allow 22/tcp

# Restart SSH
sudo systemctl restart sshd

# Connect with new port
ssh -p 2222 username@host
```

### 4. Limit User Access

```bash
# Add to /etc/ssh/sshd_config
AllowUsers username1 username2
# OR
AllowGroups sshusers

# Create SSH group
sudo groupadd sshusers
sudo usermod -aG sshusers username
```

### 5. Use Fail2Ban

```bash
# Install fail2ban
sudo apt install fail2ban  # Ubuntu/Debian
sudo dnf install fail2ban  # Fedora/CentOS

# Enable and start
sudo systemctl enable --now fail2ban

# Check status
sudo fail2ban-client status sshd
```

### 6. Two-Factor Authentication (Advanced)

```bash
# Install Google Authenticator
sudo apt install libpam-google-authenticator

# Set up for user
google-authenticator

# Edit PAM config
sudo nano /etc/pam.d/sshd
# Add: auth required pam_google_authenticator.so

# Edit SSH config
sudo nano /etc/ssh/sshd_config
# Set: ChallengeResponseAuthentication yes

# Restart SSH
sudo systemctl restart sshd
```

---

## Critical Mistakes to Avoid

### 1. ❌ Wrong Permissions

**Wrong:**
```bash
chmod 777 ~/.ssh  # Too permissive!
chmod 644 ~/.ssh/authorized_keys  # Others can read!
```

**Right:**
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### 2. ❌ Locking Yourself Out

**Always test before disabling password auth:**

```bash
# In one terminal, keep SSH session open
ssh username@host

# In another terminal, test new key
ssh -i ~/.ssh/new_key username@host

# Only after successful test, disable passwords
# Don't close first session until verified!
```

### 3. ❌ Editing Config Without Backup

**Always backup:**
```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup.$(date +%Y%m%d)
```

### 4. ❌ Not Testing Config

**Always test before restarting:**
```bash
sudo sshd -t  # Test configuration
# Only if successful:
sudo systemctl restart sshd
```

### 5. ❌ Forgetting Firewall

**Remember to allow SSH:**
```bash
# UFW
sudo ufw allow ssh

# firewalld
sudo firewall-cmd --add-service=ssh --permanent
sudo firewall-cmd --reload
```

---

## Complete Setup Checklist

- [ ] SSH server installed
- [ ] SSH service enabled and running
- [ ] SSH key pair generated
- [ ] Public key added to ~/.ssh/authorized_keys
- [ ] File permissions set correctly (700 for .ssh, 600 for authorized_keys)
- [ ] SSH config: PubkeyAuthentication yes
- [ ] SSH config: PasswordAuthentication no
- [ ] SSH config: PermitRootLogin no
- [ ] Firewall configured to allow SSH
- [ ] Configuration tested with `sshd -t`
- [ ] Tested SSH connection with new key
- [ ] Verified in SSH logs
- [ ] Kept backup session open before final changes

---

## Quick Setup Script

```bash
#!/bin/bash
# Complete SSH setup script - Run as regular user

# Install SSH server (uncomment for your distro)
# sudo apt install openssh-server -y  # Ubuntu/Debian
# sudo dnf install openssh-server -y  # Fedora/CentOS

# Generate SSH key (if not exists)
if [ ! -f ~/.ssh/id_ed25519 ]; then
    ssh-keygen -t ed25519 -C "$USER@$(hostname)" -f ~/.ssh/id_ed25519 -N ""
fi

# Create authorized_keys with your keys
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Add your public keys (replace with actual keys)
cat > ~/.ssh/authorized_keys << 'EOF'
ssh-ed25519 AAAA...your-key-1... device1
ssh-ed25519 AAAA...your-key-2... device2
EOF

chmod 600 ~/.ssh/authorized_keys

# Backup SSH config
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# Configure SSH for security
sudo tee -a /etc/ssh/sshd_config > /dev/null << 'EOF'

# Security hardening
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
MaxAuthTries 3
MaxSessions 2
EOF

# Test configuration
sudo sshd -t && echo "SSH config is valid" || echo "ERROR: SSH config has errors!"

# Enable and restart SSH
sudo systemctl enable --now sshd

# Configure firewall (UFW)
if command -v ufw &> /dev/null; then
    sudo ufw allow ssh
    sudo ufw --force enable
fi

# Configure firewall (firewalld)
if command -v firewall-cmd &> /dev/null; then
    sudo firewall-cmd --add-service=ssh --permanent
    sudo firewall-cmd --reload
fi

# Display connection info
echo ""
echo "=== SSH Server Ready ==="
echo "IP: $(ip -4 addr show | grep -oP '(?<=inet\s)\d+(\.\d+){3}' | grep -v 127.0.0.1 | head -1)"
echo "Port: 22"
echo "Username: $USER"
echo "Auth: Key-based only"
echo ""
echo "Your public key:"
cat ~/.ssh/id_ed25519.pub
```

---

**Created:** 2026-02-14
**Author:** George Wu
**Purpose:** Complete reference for Linux SSH server setup and troubleshooting
