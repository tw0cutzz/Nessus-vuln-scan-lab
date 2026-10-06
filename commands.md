# Commands

PowerShell commands (run as admin on the Win10 VM) used to make the VM vulnerable and then fix it.

Only run these on your own lab VM.

---

## Create the vulnerabilities

```powershell
# Guest account on
net user guest /active:yes

# Weak-password user
net user labuser Password1 /add

# Firewall off
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False

# WinRM on (port 5985)
Enable-PSRemoting -Force -SkipNetworkProfileCheck

# SMBv1, step 1: install the feature, then reboot the VM
Enable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -NoRestart
```

```powershell
# SMBv1, step 2: after the reboot
Set-SmbServerConfiguration -EnableSMB1Protocol $true -Force
```

- SMB signing not required is the Win10 default, so there's nothing to run for it.
- Old Firefox and VLC came from installing old versions.

---

## Fix them

```powershell
# SMBv1 off, SMB signing required
Set-SmbServerConfiguration -EnableSMB1Protocol $false -RequireSecuritySignature $true -Force
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -NoRestart

# Guest off, weak user deleted
net user guest /active:no
net user labuser /delete

# Firewall back on
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True

# Close WinRM
Disable-PSRemoting -Force
Remove-Item -Path WSMan:\Localhost\Listener\* -Recurse -Force
Disable-NetFirewallRule -DisplayName "Windows Remote Management*"
Stop-Service WinRM
Set-Service WinRM -StartupType Disabled
```

Also:
- Update Firefox and VLC (run the new installer over the old one)
- Update App Installer (WinGet) to 1.29.280 or higher
- Run Windows Update, then reboot
