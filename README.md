# Panduan Upgrade Uptime Kuma Standalone (v2.2.1 ke v2.3.2)

Uptime Kuma adalah aplikasi pemantauan (monitoring tool) berbasis open-source yang berfungsi untuk mendeteksi status keaktifan (availability/uptime) dan mengukur waktu respons (response time) dari berbagai layanan infrastruktur IT secara real-time.

Komponen dan Fungsi Utama

Metode Pemantauan Multi-Protokol: Dapat melakukan pengecekan secara periodik terhadap performa situs web (HTTP/HTTPS), konektivitas jaringan (Ping, TCP Port, DNS Record), hingga status operasional kontainer Docker.

Integrasi Notifikasi Instan: Jika sistem mendeteksi adanya kegagalan layanan (downtime), Uptime Kuma akan segera mengirimkan peringatan (alert) melalui berbagai media komunikasi seperti Telegram, Discord, Slack, Webhook, maupun Email.

Halaman Status Publik (Status Page): Menyediakan fitur untuk membuat halaman transparansi publik yang menampilkan status operasional infrastruktur kepada pengguna atau klien.

Visualisasi Data Analitis: Menyajikan dasbor interaktif yang memetakan grafik performa server dan persentase uptime untuk mempermudah analisis keandalan sistem (system reliability).

## Upgrade dari 2.2.1 ke 2.3.2 pada instalasi standalone memerlukan perhatian khusus pada ketergantungan (*dependencies*) dan hak akses file, terutama karena Anda sebelumnya sempat mengalami masalah "nyangkut" di versi lama.

<img width="1180" height="974" alt="telegram-cloud-photo-size-5-6084640882637345803-y" src="https://github.com/user-attachments/assets/af9565e3-0afc-4063-a515-2ec988339365" />


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
```
<img width="972" height="225" alt="image" src="https://github.com/user-attachments/assets/3fcb70d4-48fd-4f48-bef8-52c9c454dbaa" />

<img width="764" height="95" alt="image" src="https://github.com/user-attachments/assets/754c2615-d432-4ebb-ae2a-2e7ebc5f4455" />


### 2. Verifikasi Node.js
Uptime Kuma v2.3.2 sangat disarankan berjalan di Node.js v20 (LTS), dengan batas minimal versi 18.

```bash
node -v
```
Catatan Penting: Jika versi Node.js Anda masih v16 atau di bawahnya, Anda wajib upgrade Node.js terlebih dahulu sebelum melanjutkan proses, karena versi 2.3.x tidak akan bisa berjalan di lingkungan Node lama.

<img width="428" height="35" alt="image" src="https://github.com/user-attachments/assets/52fcf5c4-947f-4ebd-97bd-fb79a9abfce5" />


### 3. Update Source Code (Metode "Force Upgrade")
Agar tidak terjadi lagi masalah versi yang tidak berubah di `package.json`, kita akan memaksa Git menyamakan file lokal dengan tag resmi 2.3.2.

```bash
# Pastikan berada di folder aplikasi
cd /home/monitor-kuma/uptime-kuma

# Ambil informasi tag terbaru dari GitHub
git fetch --all --tags --prune

# Paksa reset file lokal ke versi 2.3.2
git reset --hard tags/2.3.2
Verifikasi: Cek apakah file sudah berubah:

Bash
grep '"version":' package.json
(Hasilnya harus menunjukkan "version": "2.3.2",)
```
<img width="993" height="687" alt="image" src="https://github.com/user-attachments/assets/0577c500-b35b-44de-9e1b-7957cf03ca9b" />

<img width="585" height="68" alt="image" src="https://github.com/user-attachments/assets/99a4941a-ffbf-4a5b-8bdc-e5331bd44ea4" />

### 4. Instalasi & Build Bersih
Karena ada perubahan besar di sisi library, kita akan menghapus modul lama dan mengunduh aset tampilan yang baru agar tidak terjadi konflik.

```bash
# Hapus modul lama dan aset dist lama agar tidak konflik
rm -rf node_modules dist

# Install library baru versi produksi
npm install --production

# Download aset tampilan (frontend) versi 2.3.2
npm run download-dist
```

<img width="1137" height="350" alt="image" src="https://github.com/user-attachments/assets/47f36847-f066-4a95-9e55-c5099ea66618" />

<img width="1139" height="229" alt="image" src="https://github.com/user-attachments/assets/13fe976e-f328-4e0e-8757-776406d1fb6c" />


### 5. Perbaikan Hak Akses (Mencegah Port 3001 Gagal Up)
Masalah port tidak *up* sebelumnya sering disebabkan oleh kegagalan aplikasi menulis ke file `data/db-config.json` atau database SQLite.

```bash
# Kembalikan kepemilikan folder ke user monitor-kuma
sudo chown -R monitor-kuma:monitor-kuma /home/monitor-kuma/uptime-kuma
```

<img width="783" height="25" alt="image" src="https://github.com/user-attachments/assets/2b0586c3-6b6e-49fa-8470-2e59777be9f5" />

### 6. Jalankan Ulang dengan PM2
Kita akan menghapus *cache* proses lama di PM2 agar sistem membaca konfigurasi baru secara segar.

```bash
# Hapus proses lama dari daftar PM2 untuk membersihkan cache
pm2 delete uptime-kuma

# Jalankan kembali server Uptime Kuma menggunakan konfigurasi yang baru
pm2 start server/server.js --name uptime-kuma

# Simpan konfigurasi PM2 agar otomatis berjalan kembali saat server reboot
pm2 save
```
<img width="1109" height="391" alt="image" src="https://github.com/user-attachments/assets/e79495a5-31f3-4f35-9f50-9151950de11a" />


### 7. Cara Memastikan Semuanya OK
Setelah menjalankan perintah di atas, pantau log aplikasi selama 1 menit. Versi 2.3.x mungkin akan melakukan proses migrasi database skala kecil saat pertama kali dijalankan.

```bash
pm2 logs uptime-kuma --lines 50
```
Tanda-tanda Berhasil:

Muncul pesan: Welcome to Uptime Kuma

Muncul pesan: Version: 2.3.2

Muncul pesan: Listening on 3001

<img width="1154" height="1160" alt="telegram-cloud-photo-size-5-6084640882637345810-y" src="https://github.com/user-attachments/assets/3fdb8af2-4d45-46a5-924d-61f1ef0701b1" />


## 💡 Tips Jika Port 3001 Masih Tidak Up

Jika port tetap tidak mau terbuka setelah melakukan *restart*, jalankan perintah ini untuk memeriksa apakah ada proses lain yang "menggantung" pada sistem:

```bash
sudo netstat -tulpn | grep :3001
```
