# Warning: Cryptominer Malware on VPS

This documentation is created as a warning for fellow VPS users. Below are findings of cryptominer malware discovered on a VPS server.

---

## Summary

**3 cryptominers** were found running secretly on the VPS, mining Monero (XMR) cryptocurrency without the owner's knowledge. This malware uses CPU and RAM intensively, causing the VPS to become slow.

---

## Root Cause: CVE-2025-55182 (React2Shell)

After investigation, the source of infection was identified as a **critical vulnerability in React Server Components and Next.js**.

### CVE Details

| Detail | Information |
|--------|-------------|
| CVE ID | **CVE-2025-55182** |
| Name | React2Shell |
| Severity | **CVSS 10.0** (CRITICAL - Maximum!) |
| Type | Remote Code Execution (RCE) |
| Cause | Insecure Deserialization in React Server Components |

### Affected Versions

| Package | Vulnerable Version | Patched Version |
|---------|-------------------|-----------------|
| React | 19.0, 19.1.0, 19.1.1, 19.2.0 | 19.0.1, 19.1.2, **19.2.1+** |
| Next.js | < 16.0.10 | **16.0.10+** |

### Affected Frameworks

- Next.js
- React Router
- Waku
- @parcel/rsc
- @vitejs/plugin-rsc
- rwsdk

### Attack Timeline

| Date | Event |
|------|-------|
| Nov 29, 2025 | Vulnerability reported to Meta Bug Bounty |
| Dec 3, 2025 | Fix published, CVE announced |
| Dec 5, 2025 | Mass exploitation began |
| **Dec 5, 2025** | **First malware appeared on this VPS!** |

### How The Attack Works

1. Attacker sends HTTP request with malicious payload
2. React Server Components processes the request
3. Insecure deserialization allows **Remote Code Execution**
4. Attacker can run any command on the server
5. Cryptominer is downloaded and executed

### Statistics (from Wiz Research)

- **39%** of cloud environments have vulnerable React/Next.js
- **61%** of Next.js deployments are publicly exposed
- Exploit has **~100% reliability**
- **No authentication required** for exploitation

### References

- [React Official Security Advisory](https://react.dev/blog/2025/12/03/critical-security-vulnerability-in-react-server-components)
- [Next.js Security Advisory CVE-2025-66478](https://nextjs.org/blog/CVE-2025-66478)
- [Wiz Blog - React2Shell Analysis](https://www.wiz.io/blog/critical-vulnerability-in-react-cve-2025-55182)
- [Palo Alto Unit42 - Exploitation Report](https://unit42.paloaltonetworks.com/cve-2025-55182-react-and-cve-2025-66478-next/)
- [Akamai Security Research](https://www.akamai.com/blog/security-research/cve-2025-55182-react-nextjs-server-functions-deserialization-rce)

---

## Malware Locations

### 1. `/home/ubuntu/.config/.system-monitor/.sys-mon`

| Detail | Information |
|--------|-------------|
| Date Created | December 5, 2025 |
| Size | 14 MB |
| Type | XMRig Miner (Monero) |
| Hidden | Yes (folder `.system-monitor` with dot prefix) |

### 2. `/home/ubuntu/.../.x/` (Hidden Folder)

| Detail | Information |
|--------|-------------|
| Date Created | December 8-11, 2025 |
| Mining Pool | `204.93.253.180:80` |
| Folder Name | `...` (three dots, very hidden) |

**Folder contents:**
```
/home/ubuntu/.../.x/
├── stak/xmrig    # Miner executable
├── upd           # Persistence script (auto-restart)
├── run           # Launcher script
├── bash          # Secondary launcher
├── h64           # Process hider (disguises process name)
├── bash.pid      # PID file
└── cron.d        # Cron config
```

### 3. `/tmp/runnv/runnv`

| Detail | Information |
|--------|-------------|
| Date Created | December 9, 2025 |
| Size | 8 MB |
| Mining Pool | `flarepulsar.top:9201` |
| Wallet Address | `49Qp2aEzUdEANd88muJEvDVKEzn9xbm5xEXjZ8QUeN1ndVxvtUuSjZAecFJHabrzYE2VXTu5sZM8H5GiKfKah1VJBwuWhYc` |

---

## Persistence Method (Auto-Restart)

The malware uses **cron jobs** to ensure it keeps running:

```bash
# Cron jobs added by malware
* * * * * /home/ubuntu/.../.x/upd >/dev/null 2>&1    # Runs every MINUTE
@reboot /home/ubuntu/.../.x/upd >/dev/null 2>&1      # Runs on every BOOT
```

The `upd` script checks if the miner is still running, if not it will automatically restart.

---

## How to Check Your VPS

### 1. Check Suspicious Processes

```bash
ps aux | grep -E "(xmrig|runnv|sys-mon|cryptonight)" | grep -v grep
```

If there's output, your VPS is likely infected.

### 2. Check Hidden Folders

```bash
# Check folder with weird name (three dots)
ls -la ~/.../ 2>/dev/null

# Check .system-monitor folder
ls -la ~/.config/.system-monitor/ 2>/dev/null

# Check folder in /tmp
ls -la /tmp/runnv/ 2>/dev/null
```

### 3. Check Suspicious Cron Jobs

```bash
crontab -l | grep -E "(upd|\.x|runnv|miner)"
```

### 4. Check High CPU Usage

```bash
top -b -n 1 | head -20
```

If there's a process with high CPU usage (>50%) that you don't recognize, it's suspicious.

### 5. Check Connections to Mining Pools

```bash
netstat -an | grep -E "(9201|3333|5555|14444)"
ss -tp | grep -E "(miner|xmrig|pool)"
```

---

## How to Clean

### 1. Kill All Miner Processes

```bash
pkill -9 -f "xmrig"
pkill -9 -f "runnv"
pkill -9 -f "sys-mon"
pkill -9 -f "cryptonight"
```

### 2. Remove Malware Cron Jobs

```bash
# View existing cron
crontab -l

# Edit and remove suspicious lines
crontab -e

# Or remove all cron (be careful if you have important crons)
crontab -r
```

### 3. Delete Malware Files

```bash
rm -rf /tmp/runnv
rm -rf ~/.config/.system-monitor
rm -rf ~/...
```

### 4. Verify

```bash
# Make sure no miner processes
ps aux | grep -E "(xmrig|runnv|sys-mon)" | grep -v grep

# Make sure cron is clean
crontab -l
```

---

## How to Update (MANDATORY!)

### Update React & Next.js

```bash
# Go to project folder
cd /path/to/your/nextjs-project

# Update to latest version
npm update next react react-dom --save

# Fix vulnerabilities
npm audit fix --force

# Verify version
npm list next react
```

### Or use official tool:

```bash
npx fix-react2shell-next
```

### After Update

**IMPORTANT:** If your application was online and unpatched since December 4, 2025, you **MUST rotate all secrets**:
- Database credentials
- API keys
- JWT secrets
- Sensitive environment variables

---

## Prevention

1. **Update all applications** to latest version
2. **Use SSH Key** instead of password
3. **Disable root login** via SSH
4. **Use firewall** (UFW/iptables)
5. **Monitor resource usage** regularly
6. **Don't run scripts** from untrusted sources
7. **Audit authorized_keys** regularly
8. **Subscribe to security advisories** for frameworks you use

---

## Quick Monitoring Commands

```bash
# One-liner to check if infected
ps aux | grep -E "(xmrig|runnv|sys-mon|miner)" | grep -v grep && echo "INFECTED!" || echo "Clean"

# Check cron
crontab -l 2>/dev/null | grep -E "(upd|\.x|runnv)" && echo "CRON INFECTED!" || echo "Cron clean"

# Check folders
(ls -d ~/.../.x ~/.config/.system-monitor /tmp/runnv 2>/dev/null) && echo "MALWARE FOLDERS EXIST!" || echo "Folders clean"
```

---

## Notes

- This malware uses **100% CPU** when mining
- Uses **37%+ RAM**
- Mines **Monero (XMR)** cryptocurrency
- Attacker's wallet: `49Qp2aEzUdEANd88muJEvDVKEzn9xbm5xEXjZ8QUeN1ndVxvtUuSjZAecFJHabrzYE2VXTu5sZM8H5GiKfKah1VJBwuWhYc`

---

*This documentation is created for educational purposes and prevention. Stay vigilant and always monitor your VPS!*
