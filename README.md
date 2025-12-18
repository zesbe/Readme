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

## Kemungkinan Sumber Infeksi

1. **SSH Key yang Compromised** - Cek `~/.ssh/authorized_keys` untuk key yang tidak dikenal
2. **Password Lemah** - Jika SSH password authentication aktif
3. **Service yang Vulnerable** - Aplikasi web yang outdated
4. **Script dari Internet** - Menjalankan script curl/wget tanpa verifikasi

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
