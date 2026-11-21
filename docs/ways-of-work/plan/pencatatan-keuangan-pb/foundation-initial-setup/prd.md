# PRD: Foundation / Initial Setup

## 1. Nama Fitur

Foundation / Initial Setup untuk aplikasi pencatatan keuangan PB.

## 2. Epic

- [PRD Epic Pencatatan Keuangan PB](../epic.md)
- [Arsitektur Epic Pencatatan Keuangan PB](../arch.md)

## 3. Tujuan

### Masalah

Proyek sudah memiliki aplikasi dasar Next.js, tetapi belum memiliki fondasi integrasi untuk kebutuhan keuangan PB. Pengembang dan operator memerlukan cara yang jelas untuk menjalankan aplikasi dan menghubungkannya ke layanan yang diperlukan. Tanpa konfigurasi yang aman dan pemeriksaan koneksi, kegagalan login atau akses data sulit dibedakan dari aplikasi yang siap digunakan. Kelayakan akses Google milik editor juga perlu dibuktikan sebelum fitur transaksi dibangun.

### Solusi

Siapkan aplikasi yang sudah ada, konfigurasi deployment, login Google melalui Stack Auth cloud, dan koneksi baca server ke Google Sheets privat. Validasi secara terbatas kelayakan akses Google yang dibutuhkan fitur editor berikutnya. Foundation ini belum menyediakan fungsi transaksi atau otorisasi editor yang siap dipakai pengguna.

### Dampak

- Pengembang dapat menjalankan dan memverifikasi fondasi dengan langkah yang terdokumentasi.
- Operator dapat mengonfigurasi layanan tanpa memasukkan rahasia ke repositori.
- Tim dapat mendeteksi masalah koneksi dan risiko akses editor sebelum membangun fitur keuangan.

## 4. Persona

- **Pengembang:** menyiapkan, menjalankan, menguji, dan mengembangkan aplikasi.
- **Operator PB:** mengelola konfigurasi deployment dan akses ke spreadsheet yang digunakan aplikasi.
- **Pengguna Google:** masuk untuk memverifikasi koneksi login; pada fitur berikutnya pengguna dengan izin editor dapat menggunakan fungsi khusus editor.
- **Pengunjung:** membuka aplikasi tanpa login; tidak memperoleh akses ke kredensial layanan.

## 5. User Stories

1. Sebagai **pengembang**, saya ingin menjalankan proyek yang sudah ada dengan konfigurasi yang terdokumentasi agar saya dapat melanjutkan pengembangan.
2. Sebagai **operator PB**, saya ingin mengonfigurasi dan menjalankan aplikasi melalui container agar aplikasi siap dipasang pada hosting yang dipilih.
3. Sebagai **pengguna Google**, saya ingin masuk dan keluar melalui Stack Auth agar koneksi autentikasi dasar dapat diverifikasi.
4. Sebagai **operator PB**, saya ingin aplikasi membaca spreadsheet privat yang dikonfigurasi agar fitur publik berikutnya dapat menggunakan Google Sheets sebagai sumber data.
5. Sebagai **pengembang**, saya ingin membuktikan kelayakan akses Google akun editor agar risiko integrasi diketahui sebelum fitur khusus editor dibuat.
6. Sebagai **operator PB**, saya ingin mengetahui jika konfigurasi atau koneksi gagal agar saya tidak menganggap layanan siap ketika belum berfungsi.
7. Sebagai **pengunjung**, saya ingin membuka aplikasi tanpa login agar akses publik pada epic tetap memungkinkan.

## 6. Persyaratan

### Fungsional

**Persiapan proyek dan deployment**

- Gunakan proyek Next.js App Router, React, TypeScript, dan Yarn yang sudah ada; jangan membuat ulang proyek.
- Sediakan konfigurasi dan petunjuk untuk menjalankan aplikasi secara lokal serta membangun dan menjalankan image Docker.
- Sediakan daftar konfigurasi yang diperlukan beserta contoh nilai nonrahasia dan petunjuk penyediaannya. Pisahkan konfigurasi yang boleh terlihat di browser dari secret server.
- Sediakan cara bagi operator untuk memeriksa koneksi Stack Auth dan Google Sheets tanpa menampilkan nilai secret atau token.
- Dokumentasikan cara menyiapkan proyek Stack Auth cloud, OAuth client Google milik aplikasi, API Google yang diperlukan, spreadsheet uji, dan akun layanan pembaca. Kebutuhan verifikasi OAuth harus dicatat bila berlaku.
- Pertahankan jalur publik yang dapat dibuka tanpa login. Konten bisnis dashboard dan transaksi belum menjadi bagian Foundation.

**Autentikasi dasar**

- Pengguna dapat memulai login Google melalui Stack Auth cloud dan kembali ke aplikasi setelah berhasil.
- Aplikasi dapat mengenali sesi pengguna yang masuk dan menyediakan cara keluar.
- Jika login dibatalkan atau gagal, pengguna mendapat pesan yang jelas dan tetap dapat mengakses jalur publik.
- Login berhasil hanya membuktikan identitas; Foundation tidak boleh menampilkan atau mengklaim bahwa pengguna memiliki izin editor.

**Koneksi Google Sheets**

- Operator mengonfigurasi satu spreadsheet dan menyediakan akses baca bagi akun layanan yang digunakan backend.
- Backend dapat membaca data uji dari spreadsheet yang dikonfigurasi tanpa meminta pengunjung masuk atau memiliki izin Google Sheets.
- Spreadsheet uji tetap privat; akun layanan Foundation hanya diberi izin baca. Foundation tidak menyediakan operasi tulis.
- Jika konfigurasi tidak lengkap, spreadsheet tidak ditemukan, izin baca ditolak, atau Google Sheets tidak tersedia, tampilkan kegagalan yang dapat ditindaklanjuti; jangan menggantinya dengan data uji yang tampak berhasil.
- Pemeriksaan koneksi harus memakai data uji nonrahasia dan tidak membuka seluruh isi spreadsheet melalui endpoint diagnostik publik.

**Validasi kelayakan akses editor**

- Dalam lingkungan uji, buktikan bahwa akun Google terhubung dapat memberikan token yang dapat digunakan backend untuk memeriksa kemampuan edit pada spreadsheet yang ditentukan.
- Uji alur persetujuan dan pemberian akses aplikasi ke file, termasuk apakah scope `drive.file` dan Google Picker sesuai kebutuhan. Periksa kondisi editor, bukan editor, izin dicabut, dan akses yang ditolak.
- Catat hasil, keterbatasan, scope yang diperlukan, serta keputusan atau hambatan pada dokumentasi teknis. Jika alur tidak layak, jangan menyatakan integrasi editor siap; buat keputusan arsitektur sebelum melanjutkan fitur editor.
- Validasi ini tidak menggantikan pemeriksaan otorisasi server pada setiap tindakan khusus editor dalam fitur berikutnya.

### Nonfungsional

- Secret, token, dan kredensial tidak disimpan dalam repositori, dikirim ke pengunjung, atau dicatat dalam log. Token Google berumur pendek yang diperlukan Picker pada lingkungan uji hanya digunakan untuk keperluan tersebut dan tidak disimpan permanen di browser.
- Konfigurasi produksi menggunakan HTTPS. Akses diagnostik dan pesan kesalahan tidak boleh membocorkan kredensial atau data spreadsheet.
- Kegagalan layanan eksternal harus dapat dibedakan dari keberhasilan; data lama atau kosong tidak boleh ditampilkan sebagai hasil koneksi terkini yang berhasil.
- Login dan jalur publik dapat digunakan pada desktop dan perangkat seluler, dengan label serta pesan yang dapat dipahami.
- Belum ada target numerik kinerja atau ketersediaan; target tersebut mengikuti keputusan epic.
- Foundation tidak menambahkan database transaksi atau layanan infrastruktur lain yang tidak diperlukan untuk cakupan ini.

## 7. Kriteria Penerimaan

### Menjalankan dan mengonfigurasi

- [ ] Dengan dependensi dan konfigurasi yang terdokumentasi, pengembang dapat menjalankan aplikasi lokal dan membuka jalur publik tanpa login.
- [ ] Operator dapat membangun image dan menjalankan aplikasi dalam Docker menggunakan konfigurasi eksternal.
- [ ] Contoh konfigurasi tidak mengandung secret nyata. Jika konfigurasi wajib tidak tersedia, operator mendapat pesan yang menunjukkan konfigurasi yang perlu diperbaiki tanpa mengungkap nilainya.

### Autentikasi

- [ ] Dengan Stack Auth cloud dan OAuth Google yang dikonfigurasi, pengguna dapat masuk, dikenali sebagai pengguna masuk, lalu keluar.
- [ ] Jika login dibatalkan atau layanan autentikasi gagal, aplikasi menampilkan pesan yang jelas dan jalur publik tetap dapat dibuka.
- [ ] Setelah login, aplikasi tidak menyatakan pengguna sebagai editor hanya berdasarkan keberhasilan login.

### Koneksi baca

- [ ] Dengan spreadsheet uji privat yang dibagikan hanya sebagai pembaca kepada akun layanan, backend berhasil membaca data uji yang ditentukan tanpa login pengunjung.
- [ ] Jika izin akun layanan dicabut, spreadsheet salah, atau layanan tidak tersedia, pemeriksaan menyatakan gagal dan tidak melaporkan data palsu sebagai keberhasilan.
- [ ] Akun layanan tidak dapat menulis ke spreadsheet uji. Diagnostik tidak membocorkan isi spreadsheet, token, atau kredensial kepada pengunjung.

### Validasi kelayakan editor

- [ ] Hasil uji mencatat apakah backend dapat menggunakan token akun Google terhubung untuk memeriksa spreadsheet yang dikonfigurasi, termasuk scope dan cara memberi akses aplikasi ke file.
- [ ] Uji membedakan editor, pengguna tanpa izin edit, izin yang dicabut, dan consent ditolak; kegagalan pemeriksaan tidak dicatat sebagai izin edit.
- [ ] Hasil uji mencatat perilaku token yang kedaluwarsa atau dicabut serta hambatan yang masih ada. Jika ada hambatan, hasil menyatakan belum siap dan menyebut keputusan lanjutan yang diperlukan.
- [ ] Tidak ada operasi tulis transaksi atau fitur berbagi yang dapat digunakan sebelum pemeriksaan izin server untuk fitur tersebut dibangun.

## 8. Di Luar Cakupan

- Formulir, daftar, pembuatan, perubahan, atau penghapusan transaksi.
- Dashboard keuangan, perhitungan saldo, laporan, dan berbagi WhatsApp.
- Implementasi lengkap izin editor untuk tindakan bisnis dan antarmuka pengelolaan izin.
- Migrasi data keuangan, penentuan skema transaksi final, dan penulisan ke spreadsheet produksi.
- Hosting Stack Auth sendiri, database transaksi, Turborepo, atau otomatisasi workflow.
- Deploy ke produksi dan pembelian layanan; kesiapan konfigurasi deployment termasuk dalam Foundation.

## 9. Ketergantungan dan Keputusan Terbuka

- Operator perlu menyediakan atau menyiapkan proyek Stack Auth cloud, proyek Google Cloud, OAuth client, dan spreadsheet uji. Penyediaan layanan dapat memiliki biaya atau batas penggunaan.
- Domain dan hosting aplikasi belum dipilih; callback OAuth dan konfigurasi HTTPS harus disesuaikan saat deployment.
- Struktur transaksi final, mata uang, zona waktu, dan migrasi data mengikuti keputusan epic; Foundation menggunakan data uji terpisah.
- Scope OAuth, kebutuhan verifikasi Google, serta kelayakan koneksi token dan Picker harus disimpulkan dari validasi sebelum fitur editor dibangun.
