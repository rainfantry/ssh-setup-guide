# Windows SSH Server Setup Guide

Complete documentation for setting up and troubleshooting OpenSSH server on Windows with key-based authentication.

## 📚 Contents

- **[windows-ssh-setup.md](windows-ssh-setup.md)** - Complete setup and troubleshooting guide

## 🚀 Quick Start

### For Administrators

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

## 🔍 Common Issues

| Issue | Solution |
|-------|----------|
| Permission denied | Check if using `administrators_authorized_keys` for admin users |
| Connection refused | Enable firewall rule: `Set-NetFirewallRule -Profile Any` |
| Still asks password | Verify `PasswordAuthentication no` in sshd_config |
| Keys keep disappearing | Don't use `Set-Content` repeatedly - see guide |

## 📖 Full Documentation

See [windows-ssh-setup.md](windows-ssh-setup.md) for complete documentation including:
- Initial setup steps
- Key-based authentication configuration
- Firewall setup
- Troubleshooting guide
- Common mistakes to avoid
- Complete command reference

## 🎯 Key Points

1. **Administrators** must use `C:\ProgramData\ssh\administrators_authorized_keys`
2. **Always set permissions** after modifying authorized_keys
3. **Use multi-line Set-Content** or Add-Content to avoid overwriting keys
4. **Enable firewall for all network profiles** (Private, Public, Domain)

## 📝 Author

George Wu (gwu0738@gmail.com)

## 📅 Last Updated

2026-02-14
