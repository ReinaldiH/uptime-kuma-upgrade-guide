# Panduan Upgrade Uptime Kuma Standalone (v2.2.1 ke v2.3.2)

Dokumen ini berisi panduan langkah-demi-langkah untuk melakukan *upgrade* Uptime Kuma dari versi **2.2.1** ke **2.3.2** pada instalasi *standalone*. Panduan ini disusun khusus untuk mengatasi kendala ketergantungan (*dependencies*), hak akses file, serta mencegah masalah versi "nyangkut" yang sempat terjadi sebelumnya.

---

Upgrade dari 2.2.1 ke 2.3.2 pada instalasi standalone memerlukan perhatian khusus pada ketergantungan (dependencies) dan hak akses file, terutama karena Anda sebelumnya sempat mengalami masalah "nyangkut" di versi lama.

Berikut adalah panduan Proper & Aman untuk melakukan upgrade tersebut:

1. Persiapan & Backup (Wajib)
Sebelum melakukan perubahan, amankan database Anda. Jika terjadi kesalahan, Anda bisa kembali ke kondisi semula dengan mudah.

Bash



cd /home/monitor-kuma/uptime-kuma
# Hentikan layanan
pm2 stop uptime-kuma
# Backup folder data ke home user
cp -r data/ ~/kuma-backup-v2-2-1
2. Verifikasi Node.js
Uptime Kuma v2.3.2 sangat disarankan berjalan di Node.js v20 (LTS). Minimal versi 18.



Bash


node -v
Catatan: Jika versi Node Anda masih v16 atau di bawahnya, Anda wajib upgrade Node.js terlebih dahulu sebelum lanjut, karena versi 2.3.x tidak akan bisa jalan di Node lama.

3. Update Source Code (Metode "Force Upgrade")
Agar tidak terjadi lagi masalah versi yang tidak berubah di package.json, kita akan memaksa Git menyamakan file lokal dengan tag resmi 2.3.2.



Bash


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

4. Instalasi & Build Bersih
Karena ada perubahan besar di sisi library, kita akan menghapus modul lama dan mengunduh aset tampilan yang baru.



Bash


# Hapus modul lama dan aset dist lama agar tidak konflik
rm -rf node_modules dist
# Install library baru
npm install --production
# Download aset tampilan (frontend) versi 2.3.2
npm run download-dist
5. Perbaikan Hak Akses (Mencegah Port 3001 Gagal Up)
Masalah port tidak up sebelumnya sering disebabkan oleh kegagalan aplikasi menulis ke file data/db-config.json atau database SQLite.



Bash


# Kembalikan kepemilikan folder ke user monitor-kuma
sudo chown -R monitor-kuma:monitor-kuma /home/monitor-kuma/uptime-kuma
6. Jalankan Ulang dengan PM2
Kita akan menghapus cache proses lama di PM2 agar dia membaca konfigurasi baru secara segar.



Bash


pm2 delete uptime-kuma
pm2 start server/server.js --name uptime-kuma
pm2 save
7. Cara Memastikan Semuanya OK
Setelah menjalankan perintah di atas, pantau log-nya selama 1 menit. Versi 2.3.x mungkin melakukan migrasi database kecil saat pertama kali dijalankan.



Bash


pm2 logs uptime-kuma --lines 50
Tanda-tanda Berhasil:

Muncul pesan: Welcome to Uptime Kuma

Muncul pesan: Version: 2.3.2

Muncul pesan: Listening on 3001

Tips Jika Port 3001 Masih Tidak Up:
Jika port tetap tidak mau terbuka, jalankan perintah ini untuk melihat apakah ada proses lain yang "menggantung":



Bash


sudo netstat -tulpn | grep :3001
Jika ada proses yang muncul (selain PM2 yang baru saja Anda buat), matikan proses tersebut dengan sudo kill -9 <PID> lalu restart PM2-nya kembali.
