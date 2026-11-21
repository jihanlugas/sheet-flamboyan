# Epic: Pencatatan Keuangan PB

## 1. Tujuan

### Masalah

PB membutuhkan pencatatan pemasukan dan pengeluaran yang terstruktur dan mudah diperiksa bersama. Pencatatan manual menyulitkan pengurus melihat transaksi dan menghitung saldo secara konsisten. Penyusunan ringkasan keuangan untuk dibagikan juga membutuhkan pekerjaan tambahan. Akses perubahan perlu dibatasi agar tidak semua pengunjung dapat mengubah catatan keuangan.

### Solusi

Sediakan aplikasi Next.js dengan Google Sheets sebagai penyimpanan data keuangan. Pengunjung dapat melihat dashboard, daftar transaksi, dan saldo tanpa masuk. Editor Google Sheets dapat masuk dengan akun Google untuk mencatat transaksi dan membagikan ringkasan laporan melalui WhatsApp.

### Dampak yang diharapkan

- Pencatatan transaksi lebih terstruktur dan mudah dilakukan.
- Saldo dihitung otomatis dari transaksi yang tersimpan.
- Informasi keuangan dapat dilihat bersama, sementara perubahan dibatasi kepada editor.
- Pengurus lebih mudah menyiapkan ringkasan keuangan untuk dibagikan.

## 2. Persona

- **Editor/pengurus PB:** memiliki izin editor pada Google Sheets yang digunakan aplikasi; mencatat pemasukan dan pengeluaran serta membagikan laporan.
- **Pengunjung:** melihat dashboard dan daftar transaksi tanpa perlu masuk; tidak dapat mencatat transaksi atau menggunakan fitur berbagi laporan di aplikasi.

## 3. Perjalanan Pengguna

1. Pengunjung membuka dashboard dan melihat total pemasukan, total pengeluaran, serta saldo PB.
2. Pengunjung membuka halaman transaksi dan melihat daftar pemasukan dan pengeluaran yang tersimpan.
3. Pengurus masuk menggunakan akun Google. Aplikasi memeriksa izin editor pada Google Sheets yang digunakan.
4. Editor mencatat transaksi pemasukan atau pengeluaran. Setelah penyimpanan berhasil, transaksi muncul dalam daftar dan saldo diperbarui.
5. Editor memilih periode laporan dan melihat ringkasan keuangan untuk periode tersebut.
6. Editor memilih fitur berbagi. Aplikasi membuka WhatsApp dengan teks ringkasan yang telah disiapkan; editor memilih penerima dan mengirimnya secara manual.
7. Jika izin editor tidak tersedia atau telah dicabut, pengguna tetap dapat melihat data publik tetapi tidak dapat mencatat transaksi atau menggunakan fitur berbagi di aplikasi.

## 4. Persyaratan Bisnis

### Fungsional

- **Akses dan otorisasi**
  - Dashboard, daftar transaksi, dan ringkasan laporan dapat dilihat publik tanpa autentikasi.
  - Pengguna masuk dengan akun Google untuk menggunakan fungsi khusus editor.
  - Hanya akun yang memiliki izin editor pada Google Sheets yang digunakan aplikasi dapat membuat transaksi dan menggunakan fitur berbagi laporan.
  - Pemeriksaan hak akses berlaku pada sisi server untuk setiap permintaan pembuatan transaksi dan penggunaan fitur berbagi, bukan hanya melalui tampilan.
  - Jika izin tidak dapat dipastikan, aplikasi menolak fungsi khusus editor dan memberi pesan yang jelas.
  - Perubahan izin pada Google Sheets harus tercermin pada pemeriksaan akses berikutnya.
- **Transaksi**
  - Editor dapat membuat transaksi pemasukan atau pengeluaran dengan tanggal, jenis, jumlah, dan keterangan.
  - Jumlah transaksi harus lebih besar dari nol. Aplikasi menolak data yang tidak valid dan menjelaskan kesalahannya.
  - Aplikasi menyimpan transaksi yang berhasil ke Google Sheets dan menampilkan konfirmasi hanya setelah penyimpanan berhasil.
  - Pengunjung dapat melihat tanggal, jenis, jumlah, dan keterangan setiap transaksi yang tersimpan.
  - Kegagalan penyimpanan ditampilkan dengan jelas; aplikasi tidak menganggap transaksi berhasil dicatat.
- **Dashboard dan saldo**
  - Dashboard menampilkan total pemasukan, total pengeluaran, dan saldo PB berdasarkan transaksi yang tersimpan.
  - Saldo awal adalah nol. Saldo dihitung sebagai total pemasukan dikurangi total pengeluaran.
  - Setelah transaksi tersimpan, daftar dan dashboard menampilkan data terbaru. Perubahan data langsung di Google Sheets juga harus tercermin saat data dimuat ulang.
- **Laporan dan WhatsApp**
  - Pengunjung dapat memilih periode tanggal dan melihat ringkasan pemasukan dan pengeluaran dalam periode tersebut, beserta saldo PB terkini.
  - Perhitungan ringkasan menggunakan transaksi yang sama dengan daftar transaksi dan dashboard.
  - Editor dapat membuka WhatsApp dengan teks yang memuat periode laporan, total pemasukan dan pengeluaran periode tersebut, serta saldo PB terkini.
  - Aplikasi tidak mengirim pesan otomatis; editor memilih penerima dan mengonfirmasi pengiriman di WhatsApp.
  - Jika tidak ada transaksi dalam periode yang dipilih, ringkasan menampilkan total periode nol dan tetap menampilkan saldo PB terkini.

### Nonfungsional

- Google Sheets menjadi sumber data yang digunakan aplikasi untuk pencatatan dan referensi laporan.
- Pengunjung tidak memerlukan izin Google Sheets untuk membaca data publik melalui aplikasi.
- Kredensial dan akses tulis Google Sheets tidak boleh dibuka kepada pengunjung. Pembatasan perubahan harus tetap berlaku meskipun permintaan dibuat di luar antarmuka aplikasi.
- Data keuangan yang ditampilkan bersifat publik. Informasi sensitif tidak boleh dicatat kecuali PB bersedia mempublikasikannya.
- Fitur berbagi dibatasi dalam aplikasi, tetapi informasi yang sudah publik tetap dapat disalin dan dibagikan di luar aplikasi.
- Antarmuka dashboard dan transaksi dapat digunakan pada perangkat seluler dan desktop, serta menyediakan label formulir dan pesan kesalahan yang jelas.
- Ketika Google Sheets atau autentikasi tidak tersedia, aplikasi menampilkan kondisi gagal secara jelas dan tidak menyajikan data yang tidak berhasil dimuat sebagai data terkini.
- Target waktu respons, kebutuhan aksesibilitas terukur, dan kebijakan penanganan data perlu disepakati sebelum implementasi.

## 5. Metrik Keberhasilan

Metrik berikut perlu diukur setelah peluncuran; target numeriknya belum ditetapkan.

- Jumlah dan persentase transaksi keuangan PB yang dicatat melalui aplikasi dibandingkan total transaksi PB.
- Persentase percobaan pencatatan valid oleh editor yang berhasil tersimpan.
- Jumlah ketidaksesuaian antara saldo aplikasi dan hasil perhitungan transaksi pada Google Sheets.
- Jumlah percobaan perubahan oleh pengguna tanpa izin yang berhasil: target **nol**.
- Jumlah penggunaan fitur membuka WhatsApp dengan ringkasan laporan. Pengiriman pesan tidak dapat disimpulkan hanya dari pembukaan WhatsApp.
- Waktu yang dibutuhkan pengurus untuk mencatat transaksi dan menyiapkan ringkasan laporan.

## 6. Di Luar Cakupan

- Pengelolaan pemain dan sesi, biaya lapangan atau bola, serta perhitungan kurang atau lebih bayar.
- Pengubahan atau penghapusan transaksi melalui aplikasi.
- Saldo awal terpisah dari transaksi.
- Ekspor berkas CSV, Excel, atau PDF; kebutuhan berbagi pada epic ini berupa teks WhatsApp.
- Pengiriman pesan WhatsApp secara otomatis atau pelacakan pesan terkirim.
- Pembatasan akses baca kepada pemilik izin Google Sheets.

## 7. Nilai Bisnis

**Tinggi.** Epic ini memberikan catatan keuangan terpusat, saldo otomatis, transparansi bagi pengunjung, dan kontrol perubahan bagi pengurus. Ringkasan WhatsApp mengurangi pekerjaan pengurus saat berbagi informasi keuangan.

## 8. Keputusan yang Masih Diperlukan

- Format mata uang, zona waktu, dan aturan penyajian tanggal.
- Apakah keterangan transaksi wajib diisi dan batas panjangnya.
- Struktur Google Sheets, penanganan data yang sudah ada, dan siapa yang mengelola izin editor.
- Rentang tanggal awal laporan dan batas ukuran atau jumlah transaksi yang ditampilkan.
- Target kinerja, ketersediaan, dan aksesibilitas yang terukur.
