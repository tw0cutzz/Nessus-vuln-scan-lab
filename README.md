# Nessus Vulnerability Scan Lab

A small home lab where I scan a Windows 10 VM with Nessus, fix what I can,
and scan again to see what changed.

## What's in this lab

- **Host:** Fedora, running KVM/libvirt
- **Scanner VM:** Ubuntu with Nessus Essentials installed
- **Target VM:** Windows 10 (deliberately weakened)
- **Network:** libvirt default NAT network, both VMs on the same network

Nessus Essentials is free, but the license lasts 30 days and covers up to
5 IPs. That's plenty for 2 VMs, but it means exporting reports as you go.

## What I did

1. Set up both VMs and took snapshots of the clean state.
2. Prepped Windows for credentialed scanning (allow ping, file sharing,
   Remote Registry, and the `LocalAccountTokenFilterPolicy` setting).
3. Ran an **uncredentialed scan** to see what an outsider sees.
4. Made the Windows VM weaker on purpose:
   - Turned on SMBv1
   - Left SMB signing off
   - Turned on WinRM (port 5985)
   - Enabled the Guest account
   - Added a user with a weak password
   - Turned the firewall off
   - Installed old versions of Firefox and VLC
5. Ran a **credentialed scan** with an admin account.
6. Fixed what could be fixed.
7. Rescanned and compared.

## Results

| Scan | Critical | High | Medium | Low | Info |
|------|----------|------|--------|-----|------|
| Uncredentialed (baseline) | [x] | [x] | 1 | [x] | [x] |
| Credentialed (vulnerable) | [x] | [x] | [x] | [x] | [x] |
| Credentialed (after fixes) | [x] | [x] | [x] | [x] | [x] |

### Uncredentialed vs credentialed

The first scan only found one Medium (SMB signing not required) and the
rest was Info. Without logging in, Nessus can only see what the network
shows it. The credentialed scan found way more: missing patches, old
software, and config problems. Same machine, very different picture.

## What I fixed

| Finding | Fix |
|---------|-----|
| SMBv1 enabled | Disabled SMBv1 (server setting and Windows feature) |
| SMB signing not required | Set signing to required |
| WinRM exposed | Disabled PSRemoting, stopped the service, closed the port |
| Guest account enabled | Disabled it |
| Weak-password user | Deleted the account |
| Firewall off | Turned it back on |
| Old Firefox | Updated |
| Old VLC | Updated |
| WinGet / App Installer < 1.29.280 (CVE-2026-68821) | Updated App Installer |
| [anything else you fixed] | [fix] |

## What couldn't be fixed

Windows 10 reached end of support in October 2025, so Microsoft no longer
releases patches for it. Some findings stay open no matter what I do:

| Finding | Why it stays |
|---------|--------------|
| [finding name] | [e.g. needs a patch that doesn't exist for Win10] |
| [finding name] | [reason] |

In a real environment the answer here would be to upgrade to a supported
Windows version, or isolate the machine if that isn't possible.

## Things I learned

- Credentialed scans show far more than uncredentialed ones.
- Nessus didn't flag the enabled Guest account for me, so I confirmed it
  manually with PowerShell. Scanners miss things, so manual checks still matter.
- A High score on paper isn't always urgent. For example, the WinGet CVE
  needs local access to exploit, so it's lower priority than something
  reachable over the network.
- End-of-life systems leave findings you can't patch away.
- Networking in a lab can break in annoying ways. Docker on my host messed
  with VM internet access, and I had to remove it to fix that.

## Screenshots

See the [`screenshots`](./screenshots) folder.

- `01-uncredentialed-scan.png`
- `02-credentialed-scan-before.png`
- `03-credentialed-scan-after.png`
- [add yours]

## Repo layout

```
.
├── README.md
├── screenshots/
├── reports/        # exported Nessus PDF/CSV reports
└── commands.md     # PowerShell commands used to weaken and fix the VM
```

## Notes

- Everything here was done on my own VMs in an isolated lab.
- Don't scan machines you don't own or have permission to test.
- The vulnerable Windows VM was never exposed to my home network.
