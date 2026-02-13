# Windows SSH Server Setup Guide

Complete documentation for setting up and troubleshooting OpenSSH server on Windows with key-based authentication.

## 📚 Contents

- **[windows-ssh-setup.md](windows-ssh-setup.md)** - Windows SSH server setup guide
- **[linux-ssh-setup.md](linux-ssh-setup.md)** - Linux SSH server setup guide
- **[git-cli-beginner-guide.md](git-cli-beginner-guide.md)** - Git command line beginner's guide
- **[claude-code-complete-guide.md](claude-code-complete-guide.md)** - Claude Code CLI complete reference

## 🚀 Quick Start

### Windows (For Administrators)

```powershell
# 1. Enable SSH services
Set-Service sshd,ssh-agent -StartupType Automatic
Start-Service sshd,ssh-agent

# 2. Add your public key
Set-Content -Path C:\ProgramData\ssh\administrators_authorized_keys -Value "your-ssh-public-key-here"

# 3. Set permissions
icacls C:\ProgramData\ssh\administrators_authorized_keys /inheritance:r
icacls C:\ProgramData\ssh\administrators_authorized_keys /grant "SYSTEM:(F)"
icacls C:\ProgramData\ssh\administrators_authorized_keys /grant "Administrators:(F)"

# 4. Disable password auth
Add-Content -Path C:\ProgramData\ssh\sshd_config -Value "`nPasswordAuthentication no`nPubkeyAuthentication yes"

# 5. Fix firewall
Set-NetFirewallRule -DisplayName "OpenSSH SSH Server (sshd)" -Profile Any

# 6. Restart SSH
Restart-Service sshd
```

### Linux

```bash
# Install SSH (Ubuntu/Debian)
sudo apt install openssh-server -y

# Generate key
ssh-keygen -t ed25519 -C "your_email@example.com"

# Add public key to authorized_keys
mkdir -p ~/.ssh
echo "your-ssh-public-key-here" >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# Configure SSH
sudo sed -i 's/^#*PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo sed -i 's/^#*PubkeyAuthentication.*/PubkeyAuthentication yes/' /etc/ssh/sshd_config
sudo sed -i 's/^#*PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config

# Enable firewall
sudo ufw allow ssh
sudo ufw enable

# Restart SSH
sudo systemctl restart sshd
```

## 🔍 Common Issues

| Issue | Solution |
|-------|----------|
| Permission denied | Check if using `administrators_authorized_keys` for admin users |
| Connection refused | Enable firewall rule: `Set-NetFirewallRule -Profile Any` |
| Still asks password | Verify `PasswordAuthentication no` in sshd_config |
| Keys keep disappearing | Don't use `Set-Content` repeatedly - see guide |

## 📖 Full Documentation

### Windows
See [windows-ssh-setup.md](windows-ssh-setup.md) for complete documentation including:
- Initial setup steps
- Key-based authentication configuration
- Firewall setup
- Troubleshooting guide
- Common mistakes to avoid
- Complete command reference

### Linux
See [linux-ssh-setup.md](linux-ssh-setup.md) for complete documentation including:
- Installation steps for all major distros
- Service management with systemd
- UFW, firewalld, and iptables configuration
- SELinux considerations
- Security hardening
- Fail2Ban setup
- Two-factor authentication

## 🎯 Key Points

### Windows

1. **Administrators** must use `C:\ProgramData\ssh\administrators_authorized_keys`
2. **Always set permissions** after modifying authorized_keys
3. **Use multi-line Set-Content** or Add-Content to avoid overwriting keys
4. **Enable firewall for all network profiles** (Private, Public, Domain)

### Linux
1. **Set correct permissions** (700 for .ssh, 600 for authorized_keys)
2. **Always test config** before restarting: `sudo sshd -t`
3. **Keep backup session open** when disabling password auth
4. **Configure firewall** (ufw/firewalld) to allow SSH

## 📝 Author

George Wu (gwu0738@gmail.com)

## 📅 Last Updated

2026-02-14
