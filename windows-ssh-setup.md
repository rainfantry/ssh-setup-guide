# Windows SSH Server Setup & Troubleshooting Guide

Complete guide for setting up, checking, and fixing SSH server on Windows with key-based authentication.

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

```powershell
# Check SSH server service
Get-Service sshd

# Check SSH agent service
Get-Service ssh-agent

# Check OpenSSH version
ssh -V
```

### Install OpenSSH (if not installed)

```powershell
# Run as Administrator
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```

---

## SSH Service Management

### Enable and Start SSH Services

```powershell
# Run as Administrator

# Enable SSH Server
Set-Service sshd -StartupType Automatic
Start-Service sshd

# Enable SSH Agent
Set-Service ssh-agent -StartupType Automatic
Start-Service ssh-agent

# Verify services are running
Get-Service sshd,ssh-agent | Select-Object Name, Status, StartType
```

### Check SSH Server Status

```powershell
# Service status
Get-Service sshd

# Check if SSH is listening on port 22
netstat -an | findstr ":22"

# Should show:
# TCP    0.0.0.0:22    0.0.0.0:0    LISTENING
# TCP    [::]:22       [::]:0       LISTENING
```

### Restart SSH Service

```powershell
# Method 1: Standard restart (may fail with MUI error)
Restart-Service sshd

# Method 2: Force restart
Stop-Service sshd -Force
Start-Service sshd

# Method 3: Kill and restart
taskkill /F /IM sshd.exe
Start-Service sshd
```

---

## Key-Based Authentication

### Understanding Windows SSH Key Locations

**For Administrator Users:**
- SSH looks in: `C:\ProgramData\ssh\administrators_authorized_keys`
- NOT in: `C:\Users\<username>\.ssh\authorized_keys`

**For Regular Users:**
- SSH looks in: `C:\Users\<username>\.ssh\authorized_keys`

### Check Your User Type

```powershell
# Check if you're in Administrators group
net user $env:USERNAME | findstr "Local Group"

# If output includes "Administrators", you must use administrators_authorized_keys
```

### Generate SSH Key Pair

```powershell
# Generate ED25519 key (recommended)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Default location: C:\Users\<username>\.ssh\id_ed25519
# Press Enter for no passphrase (or set one for extra security)
```

### View Your Public Key

```powershell
# Display public key
cat $env:USERPROFILE\.ssh\id_ed25519.pub

# Copy to clipboard
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub | Set-Clipboard
```

### Add Public Key to Authorized Keys

**For Administrator Users (CRITICAL):**

```powershell
# Run as Administrator

# Add a single key (replaces existing)
Set-Content -Path C:\ProgramData\ssh\administrators_authorized_keys -Value "ssh-ed25519 AAAA... user@host"

# Add multiple keys (preserves existing)
Add-Content -Path C:\ProgramData\ssh\administrators_authorized_keys -Value "ssh-ed25519 AAAA... user@host"

# Add multiple keys at once
Set-Content -Path C:\ProgramData\ssh\administrators_authorized_keys -Value @"
ssh-ed25519 AAAA...key1... pc-key
ssh-ed25519 AAAA...key2... phone-key
ssh-ed25519 AAAA...key3... laptop-key
"@

# Set correct permissions (REQUIRED)
icacls C:\ProgramData\ssh\administrators_authorized_keys /inheritance:r
icacls C:\ProgramData\ssh\administrators_authorized_keys /grant "SYSTEM:(F)"
icacls C:\ProgramData\ssh\administrators_authorized_keys /grant "Administrators:(F)"

# Restart SSH
Restart-Service sshd
```

**For Regular Users:**

```powershell
# Create .ssh directory if it doesn't exist
mkdir $env:USERPROFILE\.ssh -ErrorAction SilentlyContinue

# Add key
Add-Content -Path $env:USERPROFILE\.ssh\authorized_keys -Value "ssh-ed25519 AAAA... user@host"

# Set permissions
icacls $env:USERPROFILE\.ssh\authorized_keys /inheritance:r
icacls $env:USERPROFILE\.ssh\authorized_keys /grant "$env:USERNAME:(F)"
```

### Configure SSH Server for Key-Only Authentication

```powershell
# Run as Administrator

# Edit SSH config
notepad C:\ProgramData\ssh\sshd_config

# Add these lines at the end:
# PasswordAuthentication no
# PubkeyAuthentication yes

# Or append via command:
Add-Content -Path C:\ProgramData\ssh\sshd_config -Value @"

# Force key-based authentication only
PasswordAuthentication no
PubkeyAuthentication yes
"@

# Restart SSH
Restart-Service sshd
```

### Verify SSH Configuration

```powershell
# Check authentication settings
Select-String -Path C:\ProgramData\ssh\sshd_config -Pattern "PasswordAuthentication|PubkeyAuthentication"

# View authorized keys
Get-Content C:\ProgramData\ssh\administrators_authorized_keys

# Check permissions
icacls C:\ProgramData\ssh\administrators_authorized_keys
```

---

## Firewall Configuration

### Check Firewall Rules

```powershell
# Check SSH firewall rule
Get-NetFirewallRule -DisplayName "*SSH*" | Select-Object DisplayName, Enabled, Direction, Action

# Check which network profiles are allowed
Get-NetFirewallRule -DisplayName "OpenSSH SSH Server (sshd)" | Get-NetFirewallProfile | Select-Object Name
```

### Fix Firewall for All Network Types

```powershell
# Run as Administrator

# Allow SSH on all network profiles (Private, Public, Domain)
Set-NetFirewallRule -DisplayName "OpenSSH SSH Server (sshd)" -Profile Any

# Verify
Get-NetFirewallRule -DisplayName "OpenSSH SSH Server (sshd)" | Get-NetFirewallProfile
```

### Check Network Profile Type

```powershell
# See current network profile
Get-NetConnectionProfile | Select-Object Name, NetworkCategory

# Change from Public to Private (if needed)
Set-NetConnectionProfile -Name "Your-Network-Name" -NetworkCategory Private
```

### Create Firewall Rule (if missing)

```powershell
# Run as Administrator
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH SSH Server' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22 -Profile Any
```

---

## Troubleshooting

### Find Your IP Address

```powershell
# All IPv4 addresses
ipconfig | findstr "IPv4"

# Just your main network IP
(Get-NetIPAddress -AddressFamily IPv4 | Where-Object {$_.IPAddress -notlike "127.*" -and $_.IPAddress -notlike "169.*"}).IPAddress

# Your computer name
hostname
```

### View SSH Logs

```powershell
# Last 20 SSH events
Get-WinEvent -LogName 'OpenSSH/Operational' -MaxEvents 20 | Select-Object TimeCreated, Message | Format-List

# Filter for errors
Get-WinEvent -LogName 'OpenSSH/Operational' -MaxEvents 50 | Where-Object {$_.LevelDisplayName -eq "Error"}

# Watch for new connections (run before attempting connection)
Get-WinEvent -LogName 'OpenSSH/Operational' -MaxEvents 5 | Format-List
```

### Common Error Messages and Fixes

#### "Permission denied (publickey)"
**Cause:** Key mismatch or wrong authorized_keys location

**Fix:**
```powershell
# Verify you're using administrators_authorized_keys (if admin)
Get-Content C:\ProgramData\ssh\administrators_authorized_keys

# Check permissions
icacls C:\ProgramData\ssh\administrators_authorized_keys

# Should show ONLY:
# BUILTIN\Administrators:(F)
# NT AUTHORITY\SYSTEM:(F)
```

#### "Connection could not be established"
**Cause:** Firewall blocking or wrong IP address

**Fix:**
```powershell
# Check firewall
Get-NetFirewallRule -DisplayName "OpenSSH SSH Server (sshd)"

# Allow on all profiles
Set-NetFirewallRule -DisplayName "OpenSSH SSH Server (sshd)" -Profile Any

# Verify IP address
ipconfig | findstr "IPv4"
```

#### "Failed password" (when using keys)
**Cause:** Password auth still enabled or key not loaded

**Fix:**
```powershell
# Disable password authentication
Add-Content -Path C:\ProgramData\ssh\sshd_config -Value "PasswordAuthentication no"
Restart-Service sshd
```

#### "The resource loader failed to find MUI file"
**Cause:** Windows issue when restarting SSH service

**Fix:**
```powershell
# Use force restart
taskkill /F /IM sshd.exe
Start-Service sshd
```

### Test SSH Locally

```powershell
# Test from local machine
ssh username@localhost

# Test with verbose output (shows authentication process)
ssh -v username@localhost

# Test from specific IP
ssh username@192.168.1.40
```

### Add SSH Key to Agent

```powershell
# Start SSH agent (if not running)
Start-Service ssh-agent

# Add your key
ssh-add $env:USERPROFILE\.ssh\id_ed25519

# List loaded keys
ssh-add -l
```

---

## Common Commands Reference

### Quick Status Check

```powershell
# Check everything at once
Get-Service sshd,ssh-agent | Select-Object Name, Status, StartType
netstat -an | findstr ":22"
ipconfig | findstr "IPv4"
Get-NetFirewallRule -DisplayName "*SSH*" | Select-Object DisplayName, Enabled
```

### Connection Info Summary

```bash
# Your connection details:
# Host: <your-ip-address> (from ipconfig)
# Port: 22
# Username: <windows-username>
# Auth: SSH Key (no password)
```

### Mobile SSH Client (Termius) Setup

1. **Host:** Your PC's IP address (e.g., 192.168.1.40)
2. **Port:** 22
3. **Username:** Your Windows username
4. **Authentication:** Key
5. **Key:** Import or paste your private key

**Export private key from PC:**
```powershell
cat $env:USERPROFILE\.ssh\id_ed25519
```

Copy the entire output (including BEGIN/END lines) to your mobile SSH client.

---

## Critical Mistakes to Avoid

### 1. ❌ Using Set-Content repeatedly (overwrites keys)

**Wrong:**
```powershell
Set-Content -Path authorized_keys -Value "key1"  # Adds key1
Set-Content -Path authorized_keys -Value "key2"  # DELETES key1, adds key2
```

**Right:**
```powershell
# For multiple keys, use multi-line Set-Content once:
Set-Content -Path authorized_keys -Value @"
key1
key2
key3
"@

# Or use Add-Content to append:
Add-Content -Path authorized_keys -Value "new-key"
```

### 2. ❌ Wrong authorized_keys location for Administrators

**Wrong:**
```powershell
# If you're an Administrator, this won't work:
C:\Users\username\.ssh\authorized_keys
```

**Right:**
```powershell
# Administrators must use:
C:\ProgramData\ssh\administrators_authorized_keys
```

### 3. ❌ Forgetting to set permissions

**Always run after modifying authorized_keys:**
```powershell
icacls C:\ProgramData\ssh\administrators_authorized_keys /inheritance:r
icacls C:\ProgramData\ssh\administrators_authorized_keys /grant "SYSTEM:(F)"
icacls C:\ProgramData\ssh\administrators_authorized_keys /grant "Administrators:(F)"
```

### 4. ❌ Firewall only allows Private networks

**Check and fix:**
```powershell
# Check current profile
Get-NetFirewallRule -DisplayName "OpenSSH SSH Server (sshd)" | Get-NetFirewallProfile

# Allow on all network types
Set-NetFirewallRule -DisplayName "OpenSSH SSH Server (sshd)" -Profile Any
```

### 5. ❌ Using wrong username

**Your Windows username might not be what you think:**
```powershell
# Check actual username
echo $env:USERNAME

# Use this exact username when connecting via SSH
```

---

## Complete Setup Checklist

- [ ] SSH Server installed and running
- [ ] SSH Agent installed and running
- [ ] SSH key pair generated
- [ ] Public key added to correct authorized_keys location
- [ ] Authorized_keys permissions set correctly
- [ ] SSH config set to PasswordAuthentication no
- [ ] SSH config set to PubkeyAuthentication yes
- [ ] Firewall rule enabled for SSH
- [ ] Firewall allows all network profiles (Private, Public, Domain)
- [ ] Know your current IP address
- [ ] Tested SSH connection locally
- [ ] SSH logs show successful publickey authentication

---

## Quick Setup Script

```powershell
# Run entire setup as Administrator

# 1. Enable services
Set-Service sshd -StartupType Automatic
Set-Service ssh-agent -StartupType Automatic
Start-Service sshd
Start-Service ssh-agent

# 2. Configure SSH for key-only auth
Add-Content -Path C:\ProgramData\ssh\sshd_config -Value @"

# Force key-based authentication only
PasswordAuthentication no
PubkeyAuthentication yes
"@

# 3. Add your public keys
Set-Content -Path C:\ProgramData\ssh\administrators_authorized_keys -Value @"
ssh-ed25519 AAAA...your-key-1... device1
ssh-ed25519 AAAA...your-key-2... device2
"@

# 4. Set permissions
icacls C:\ProgramData\ssh\administrators_authorized_keys /inheritance:r
icacls C:\ProgramData\ssh\administrators_authorized_keys /grant "SYSTEM:(F)"
icacls C:\ProgramData\ssh\administrators_authorized_keys /grant "Administrators:(F)"

# 5. Configure firewall
Set-NetFirewallRule -DisplayName "OpenSSH SSH Server (sshd)" -Profile Any

# 6. Restart SSH
taskkill /F /IM sshd.exe
Start-Service sshd

# 7. Show connection info
Write-Host "`nYour SSH Server is ready!"
Write-Host "IP Address: $((Get-NetIPAddress -AddressFamily IPv4 | Where-Object {$_.IPAddress -notlike '127.*' -and $_.IPAddress -notlike '169.*'}).IPAddress)"
Write-Host "Username: $env:USERNAME"
Write-Host "Port: 22"
Write-Host "Auth: Key-based only (no password)"
```

---

**Created:** 2026-02-14
**Author:** George Wu
**Purpose:** Complete reference for Windows SSH server setup and troubleshooting
