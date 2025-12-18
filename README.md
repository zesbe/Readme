# Peringatan: Cryptominer Malware di VPS

Dokumentasi ini dibuat sebagai himbauan kepada teman-teman yang menggunakan VPS. Berikut adalah temuan malware cryptominer yang ditemukan di salah satu VPS.

---

## Ringkasan

Ditemukan **3 cryptominer** yang berjalan secara tersembunyi di VPS, mining cryptocurrency Monero (XMR) tanpa sepengetahuan pemilik. Malware ini menggunakan CPU dan RAM secara intensif, menyebabkan VPS menjadi lambat.

---

## Lokasi Malware

### 1. `/home/ubuntu/.config/.system-monitor/.sys-mon`

| Detail | Keterangan |
|--------|------------|
| Tanggal Dibuat | 5 Desember 2025 |
| Ukuran | 14 MB |
| Tipe | XMRig Miner (Monero) |
| Tersembunyi | Ya (folder `.system-monitor` dengan prefix titik) |

### 2. `/home/ubuntu/.../.x/` (Folder Tersembunyi)

| Detail | Keterangan |
|--------|------------|
| Tanggal Dibuat | 8-11 Desember 2025 |
| Mining Pool | `204.93.253.180:80` |
| Nama Folder | `...` (tiga titik, sangat tersembunyi) |

**Isi folder:**
```
/home/ubuntu/.../.x/
├── stak/xmrig    # Miner executable
├── upd           # Script persistence (auto-restart)
├── run           # Launcher script
├── bash          # Secondary launcher
├── h64           # Process hider (menyamarkan nama proses)
├── bash.pid      # PID file
└── cron.d        # Cron config
```

### 3. `/tmp/runnv/runnv`

| Detail | Keterangan |
|--------|------------|
| Tanggal Dibuat | 9 Desember 2025 |
| Ukuran | 8 MB |
| Mining Pool | `flarepulsar.top:9201` |
| Wallet Address | `49Qp2aEzUdEANd88muJEvDVKEzn9xbm5xEXjZ8QUeN1ndVxvtUuSjZAecFJHabrzYE2VXTu5sZM8H5GiKfKah1VJBwuWhYc` |

---

## Metode Persistence (Auto-Restart)

Malware menggunakan **cron job** untuk memastikan tetap berjalan:

```bash
# Cron job yang ditambahkan oleh malware
* * * * * /home/ubuntu/.../.x/upd >/dev/null 2>&1    # Jalan setiap MENIT
@reboot /home/ubuntu/.../.x/upd >/dev/null 2>&1      # Jalan setiap BOOT
```

Script `upd` akan mengecek apakah miner masih jalan, jika tidak akan otomatis restart.

---

## Cara Cek VPS Kamu

### 1. Cek Proses Mencurigakan

```bash
ps aux | grep -E "(xmrig|runnv|sys-mon|cryptonight)" | grep -v grep
```

Jika ada output, kemungkinan VPS kamu terinfeksi.

### 2. Cek Folder Tersembunyi

```bash
# Cek folder dengan nama aneh (tiga titik)
ls -la ~/.../ 2>/dev/null

# Cek folder .system-monitor
ls -la ~/.config/.system-monitor/ 2>/dev/null

# Cek folder di /tmp
ls -la /tmp/runnv/ 2>/dev/null
```

### 3. Cek Cron Job Mencurigakan

```bash
crontab -l | grep -E "(upd|\.x|runnv|miner)"
```

### 4. Cek Penggunaan CPU Tinggi

```bash
top -b -n 1 | head -20
```

Jika ada proses dengan CPU usage tinggi (>50%) yang tidak dikenal, patut dicurigai.

### 5. Cek Koneksi ke Mining Pool

```bash
netstat -an | grep -E "(9201|3333|5555|14444)"
ss -tp | grep -E "(miner|xmrig|pool)"
```

---

## Cara Membersihkan

### 1. Kill Semua Proses Miner

```bash
pkill -9 -f "xmrig"
pkill -9 -f "runnv"
pkill -9 -f "sys-mon"
pkill -9 -f "cryptonight"
```

### 2. Hapus Cron Job Malware

```bash
# Lihat cron yang ada
crontab -l

# Edit dan hapus baris mencurigakan
crontab -e

# Atau hapus semua cron (hati-hati jika ada cron penting)
crontab -r
```

### 3. Hapus File Malware

```bash
rm -rf /tmp/runnv
rm -rf ~/.config/.system-monitor
rm -rf ~/...
```

### 4. Cek Ulang

```bash
# Pastikan tidak ada proses miner
ps aux | grep -E "(xmrig|runnv|sys-mon)" | grep -v grep

# Pastikan cron bersih
crontab -l
```

---

## Sumber Infeksi: CVE-2025-55182 (React2Shell)

Setelah investigasi, ditemukan bahwa sumber infeksi adalah **vulnerability critical di React Server Components dan Next.js**.

### Detail CVE

| Detail | Keterangan |
|--------|------------|
| CVE ID | **CVE-2025-55182** |
| Nama | React2Shell |
| Severity | **CVSS 10.0** (CRITICAL - Maximum!) |
| Tipe | Remote Code Execution (RCE) |
| Penyebab | Insecure Deserialization di React Server Components |

### Versi yang Terdampak

| Package | Versi Vulnerable | Versi Patched |
|---------|------------------|---------------|
| React | 19.0, 19.1.0, 19.1.1, 19.2.0 | 19.0.1, 19.1.2, **19.2.1+** |
| Next.js | < 16.0.10 | **16.0.10+** |

### Framework Terdampak

- Next.js
- React Router
- Waku
- @parcel/rsc
- @vitejs/plugin-rsc
- rwsdk

### Timeline Serangan

| Tanggal | Kejadian |
|---------|----------|
| 29 Nov 2025 | Vulnerability dilaporkan ke Meta Bug Bounty |
| 3 Des 2025 | Fix dipublikasikan, CVE diumumkan |
| 5 Des 2025 | Eksploitasi massal dimulai |
| **5 Des 2025** | **Malware pertama muncul di VPS ini!** |

### Cara Kerja Serangan

1. Attacker mengirim HTTP request dengan payload berbahaya
2. React Server Components memproses request tersebut
3. Insecure deserialization memungkinkan **Remote Code Execution**
4. Attacker dapat menjalankan command apapun di server
5. Cryptominer di-download dan dijalankan

### Statistik (dari Wiz Research)

- **39%** cloud environments memiliki React/Next.js yang vulnerable
- **61%** Next.js deployments terekspos ke publik
- Exploit memiliki **~100% reliability**
- **Tidak butuh authentication** untuk eksploitasi

### Referensi

- [React Official Security Advisory](https://react.dev/blog/2025/12/03/critical-security-vulnerability-in-react-server-components)
- [Next.js Security Advisory CVE-2025-66478](https://nextjs.org/blog/CVE-2025-66478)
- [Wiz Blog - React2Shell Analysis](https://www.wiz.io/blog/critical-vulnerability-in-react-cve-2025-55182)
- [Palo Alto Unit42 - Exploitation Report](https://unit42.paloaltonetworks.com/cve-2025-55182-react-and-cve-2025-66478-next/)
- [Akamai Security Research](https://www.akamai.com/blog/security-research/cve-2025-55182-react-nextjs-server-functions-deserialization-rce)

---

## Cara Update (WAJIB!)

### Update React & Next.js

```bash
# Masuk ke folder project
cd /path/to/your/nextjs-project

# Update ke versi terbaru
npm update next react react-dom --save

# Fix vulnerabilities
npm audit fix --force

# Verifikasi versi
npm list next react
```

### Atau gunakan tool official:

```bash
npx fix-react2shell-next
```

### Setelah Update

**PENTING:** Jika aplikasi kamu online dan unpatched sejak 4 Desember 2025, **WAJIB rotate semua secrets**:
- Database credentials
- API keys
- JWT secrets
- Environment variables sensitif

---

## Pencegahan

1. **Update semua aplikasi** ke versi terbaru
2. **Gunakan SSH Key** instead of password
3. **Disable root login** via SSH
4. **Gunakan firewall** (UFW/iptables)
5. **Monitor penggunaan resource** secara berkala
6. **Jangan jalankan script** dari sumber tidak terpercaya
7. **Audit authorized_keys** secara berkala

---

## Command Monitoring Cepat

```bash
# One-liner untuk cek apakah terinfeksi
ps aux | grep -E "(xmrig|runnv|sys-mon|miner)" | grep -v grep && echo "TERINFEKSI!" || echo "Bersih"

# Cek cron
crontab -l 2>/dev/null | grep -E "(upd|\.x|runnv)" && echo "CRON TERINFEKSI!" || echo "Cron bersih"

# Cek folder
(ls -d ~/.../.x ~/.config/.system-monitor /tmp/runnv 2>/dev/null) && echo "FOLDER MALWARE ADA!" || echo "Folder bersih"
```

---

## Catatan

- Malware ini menggunakan **100% CPU** saat mining
- Menggunakan **37%+ RAM**
- Mining **Monero (XMR)** cryptocurrency
- Wallet penyerang: `49Qp2aEzUdEANd88muJEvDVKEzn9xbm5xEXjZ8QUeN1ndVxvtUuSjZAecFJHabrzYE2VXTu5sZM8H5GiKfKah1VJBwuWhYc`

---

*Dokumentasi ini dibuat untuk tujuan edukasi dan pencegahan. Tetap waspada dan selalu monitor VPS kalian!*
