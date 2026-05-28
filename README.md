# Panduan Upgrade Uptime Kuma Standalone (v2.2.1 ke v2.3.2)

Upgrade dari 2.2.1 ke 2.3.2 pada instalasi standalone memerlukan perhatian khusus pada ketergantungan (*dependencies*) dan hak akses file, terutama karena Anda sebelumnya sempat mengalami masalah "nyangkut" di versi lama.

Berikut adalah panduan *Proper* & Aman untuk melakukan upgrade tersebut:

---

## 🛠️ Langkah-Langkah Upgrade

### 1. Persiapan & Backup (Wajib)
Sebelum melakukan perubahan, amankan database Anda. Jika terjadi kesalahan, Anda bisa kembali ke kondisi semula dengan mudah.

```bash
# Masuk ke direktori aplikasi
cd /home/monitor-kuma/uptime-kuma

# Hentikan layanan Uptime Kuma yang sedang berjalan
pm2 stop uptime-kuma

# Backup folder data ke home user sebagai cadangan
cp -r data/ ~/kuma-backup-v2-2-1

### 2. Verifikasi Node.js
Uptime Kuma v2.3.2 sangat disarankan berjalan di Node.js v20 (LTS), dengan batas minimal versi 18.

```bash
node -v
