---
goal: "Menyiapkan project dan UI Foundation sebelum integrasi layanan keuangan PB"
version: "1.1"
date_created: "2026-11-21"
last_updated: "2026-11-21"
owner: "Tim pengembangan PB"
status: "Planned"
tags: ["feature", "foundation", "authentication", "google-sheets"]
---

# Introduction

![Status: Planned](https://img.shields.io/badge/status-Planned-blue)

Rencana ini memperluas urutan PRD Foundation tanpa mengubah requirement fitur. Urutan: project foundation, design system, layout, skeleton halaman, gerbang UI, kemudian integrasi layanan Foundation. Hasil akhirnya adalah aplikasi dasar siap menerima implementasi fitur keuangan, dengan login Google, koneksi baca Google Sheets privat, diagnostik, image Docker, dan hasil spike akses editor.

**Hasil review:** rencana sebelumnya langsung menuju integrasi dan belum menentukan struktur UI, halaman, dependensi antartahap, atau kriteria setiap task. Proyek bukan lagi kosong: Next.js 16.4.6, React 19.3.0, TypeScript, Yarn 1, Tailwind CSS 4.3.3, dan App Router sudah tersedia. Gunakan setup yang ada, jangan scaffold ulang.

Skeleton adalah tampilan statis, bukan fitur dashboard, transaksi, saldo, atau laporan yang berfungsi. Data asli dan mock keuangan tidak ditampilkan; tindakan penyimpanan dan berbagi tetap nonaktif. Integrasi layanan setelah gerbang UI tidak boleh menghapus layout atau skeleton.

Semua jalur pada dokumen ini relatif terhadap akar repositori. Kolom `Completed` dan `Date` diisi hanya setelah tugas benar-benar lulus kriteria. Tugas yang membutuhkan layanan nyata tidak boleh ditandai selesai berdasarkan mock.

## 1. Requirements & Constraints

- **REQ-001**: Pertahankan proyek Next.js App Router, React, TypeScript, Yarn 1, dan jalur publik tanpa login.
- **REQ-002**: Sediakan login Google, pengenalan sesi, logout, dan pesan kesalahan melalui Stack Auth/Hexclave cloud; login tidak membuktikan izin editor.
- **REQ-003**: Backend membaca spreadsheet uji privat melalui akun layanan dengan izin pembaca; pengunjung tidak memerlukan izin spreadsheet.
- **REQ-004**: Sediakan diagnostik lokal untuk memverifikasi koneksi Stack Auth dan Google Sheets tanpa menampilkan data spreadsheet, token, atau secret.
- **REQ-005**: Buktikan dalam lingkungan uji apakah token Google terhubung dapat memeriksa `capabilities.canEdit` pada spreadsheet yang ditentukan, termasuk persetujuan, pemilihan file, pencabutan izin, dan token kedaluwarsa.
- **REQ-006**: Dokumentasikan setup lokal, konfigurasi layanan, build dan run Docker, pemeriksaan, serta batasan yang belum terselesaikan.
- **REQ-007**: Siapkan struktur folder, environment, dan konvensi kode sebelum UI dan integrasi.
- **REQ-008**: Siapkan design tokens serta komponen UI, formulir, tabel, dan feedback yang digunakan skeleton; hindari abstraksi tanpa pemakaian.
- **REQ-009**: Siapkan layout dengan sidebar, header, breadcrumb, navigasi responsif, dan routing App Router.
- **REQ-010**: Siapkan skeleton dashboard, transaksi, dan laporan tanpa backend/API atau implementasi bisnis; seluruh skeleton dapat dibuka publik.
- **SEC-001**: Akun layanan hanya memiliki izin baca; tidak ada endpoint atau operasi tulis transaksi dan berbagi.
- **SEC-002**: Jangan mencatat, melakukan commit, atau mengirim secret, token, dan kredensial server ke browser. Token Google yang diperlukan Picker hanya boleh dipakai sementara pada browser pengguna yang memberikan consent.
- **SEC-003**: Diagnostik berjalan sebagai proses CLI operator, bukan endpoint HTTP publik; tangani error dengan pesan aman dan exit code bukan nol.
- **SEC-004**: Semua kegagalan autentikasi dan koneksi harus terlihat sebagai kegagalan, bukan data kosong yang menyerupai keberhasilan.
- **CON-001**: Jangan mengimplementasikan fungsi dashboard keuangan, saldo, transaksi, laporan, pengelolaan izin, atau deployment produksi; skeleton tampilan statis untuk dashboard, transaksi, dan laporan diperbolehkan.
- **CON-002**: Jangan membuat database transaksi, Turborepo, n8n, Redis, atau self-host Stack Auth.
- **CON-003**: Spreadsheet uji dan akun uji terpisah dari data produksi; jangan mengubah izin atau data produksi.
- **CON-004**: Konfigurasi `NEXT_PUBLIC_*` disematkan saat `next build`; secret dan konfigurasi server diberikan saat runtime. Jangan membangun image menggunakan secret.
- **CON-005**: Sampai gerbang TASK-032 selesai, aplikasi skeleton tidak melakukan integrasi auth/backend atau request jaringan ke API, dan tidak menggunakan data transaksi nyata. Semua kontrol penulisan, berbagi, dan login pada skeleton berstatus nonaktif dan menjelaskan bahwa fungsi belum tersedia.
- **GUD-001**: Sebelum implementasi, baca panduan relevan di `node_modules/next/dist/docs/`, terutama `01-app/02-guides/authentication-with-cache-components.md`, `01-app/02-guides/self-hosting.md`, dan `01-app/02-guides/testing/vitest.md`.
- **GUD-002**: Verifikasi dokumentasi dan tipe SDK Hexclave yang terpasang sebelum memilih API konkret; jangan menganggap contoh dokumentasi menjamin akses token server untuk alur ini.
- **GUD-003**: Ikuti dokumentasi App Router Next.js 16 yang terpasang untuk `layout`, `page`, `Link`, Client Components, dan CSS. Pertahankan React 19, konfigurasi Tailwind 4, `cacheComponents`, dan TypeScript strict.
- **PAT-001**: Pisahkan UI, adapter server Google, dan integrasi autentikasi; data sesi dan pembacaan Sheet tidak boleh menggunakan cache bersama antar pengguna.
- **PAT-002**: Gunakan Server Components secara default; gunakan Client Components hanya untuk navigasi aktif atau interaksi browser. Gunakan elemen HTML semantik dan komponen sederhana tanpa sistem tema atau tabel generik yang tidak dibutuhkan.

## 2. Implementation Steps

Setiap fase dimulai setelah semua dependensi eksplisitnya selesai. Tugas dalam fase boleh berjalan paralel hanya jika dependensi pada baris tersebut terpenuhi dan tidak mengubah file yang sama. Bila kredensial, izin, Docker, atau koneksi eksternal tidak tersedia, tandai pengujian terkait sebagai terblokir; jangan membuat hasil sintetis.

### Implementation Phase 1

- GOAL-001: Menetapkan baseline dan menyiapkan konfigurasi serta dependensi tanpa mengekspos secret.

| Task | Description | Completed | Date |
|------|-------------|-----------|------|
| TASK-001 | Catat hasil awal `git status --short`, `node --version`, `yarn --version`, dan `yarn lint`; jangan ubah atau pulihkan perubahan pengguna. Gunakan Node 24.18.0 dan Yarn 1.22.22 untuk menjalankan rencana. | | |
| TASK-002 | Setelah TASK-001, tambahkan ke `package.json` dependensi `@hexclave/next@1.0.137`, `@trpc/server@11.29.5`, `googleapis@190.0.0`, `@next/env@16.4.6`, dan `server-only@0.0.1`; tambahkan dev dependency `vitest@5.1.1` dan `tsx@4.23.15`. Gunakan Yarn dan perbarui `yarn.lock`. Tambahkan skrip `test` dengan `vitest run` dan `diagnose` dengan `tsx scripts/diagnose.ts`. Periksa kompatibilitas peer dependency; jika gagal, berhenti dan catat error, jangan mengabaikannya. | | |
| TASK-003 | Setelah TASK-002, buat `config/example.env` berisi nama konfigurasi tanpa nilai rahasia: konfigurasi publik dan server yang benar-benar diminta SDK Hexclave, `GOOGLE_APPLICATION_CREDENTIALS`, `GOOGLE_SHEETS_SPREADSHEET_ID`, dan `GOOGLE_SHEETS_TEST_RANGE`. Tetapkan range uji `Foundation!A1`. Terapkan validasi lazy di `lib/config.ts`: validasi konfigurasi saat fitur terkait digunakan, kembalikan nama variabel yang hilang tanpa nilainya, dan jangan memblokir jalur publik hanya karena konfigurasi layanan belum tersedia. | | |
| TASK-004 | Setelah TASK-003, buat `vitest.config.ts` untuk tes Node dan `tests/config.test.ts` untuk konfigurasi lengkap, variabel yang hilang, serta tidak bocornya nilai. Pastikan `config/example.env` dapat dilacak meski pola `.env*` di `.gitignore` sudah ada; jangan mengubah entri `prompt` milik pengguna. | | |
| TASK-019 | Setelah TASK-001, tambahkan ke `README.md` bagian `Struktur dan konvensi`: route dan layout di `app/`, komponen UI di `app/components/ui/`, layout di `app/components/layout/`, integrasi server di `lib/`, tes di `tests/`, dan skrip operator di `scripts/`. Gunakan TypeScript strict, alias `@/*`, nama komponen PascalCase, nama file kebab-case kecuali file konvensi Next.js, dan ESLint yang ada. Dokumentasikan direktori sebagai target, bukan klaim semuanya sudah ada. Jangan memindahkan file tanpa kebutuhan. | | |

**Kriteria fase:** TASK-001 sampai TASK-004 dan TASK-019 selesai; `yarn test`, `yarn lint`, dan `yarn tsc --noEmit` lulus. Kegagalan baseline tetap dicatat dan harus diselesaikan atau dieskalasikan sebelum fase berikutnya.

### Implementation Phase 2

- GOAL-005: Menyiapkan design system dan komponen dasar yang dipakai langsung oleh halaman.

**Dependensi fase:** semua task Phase 1 selesai.

| Task | Description | Completed | Date |
|------|-------------|-----------|------|
| TASK-020 | Setelah TASK-019 dan TASK-004, perbarui `app/globals.css` dan `app/layout.tsx`: gunakan Geist yang sudah ada sebagai font utama, ukuran body `1rem`, judul halaman `1.875rem`. Mode terang: latar `#f8fafc`, surface `#ffffff`, teks `#0f172a`, teks sekunder `#334155`, border `#cbd5e1`, aksi `#1d4ed8` dengan teks putih. Pertahankan dukungan preferensi dark yang sudah ada: latar `#0a0a0a`, surface `#171717`, teks `#ededed`, teks sekunder `#cbd5e1`, border `#64748b`, aksi `#93c5fd` dengan teks `#0f172a`. Gunakan CSS variables dan integrasi `@theme inline` Tailwind 4; sediakan focus ring yang jelas. Atur `lang="id"`; jangan menambah font atau library UI. | | |
| TASK-021 | Setelah TASK-020, buat `app/components/ui/button.tsx` dan `app/components/ui/card.tsx`: `Button` mendukung atribut HTML asli, varian primary/secondary, focus dan disabled; `Card` menyediakan container sederhana. Gunakan hanya prop yang diperlukan skeleton. | | |
| TASK-022 | Setelah TASK-020, buat `app/components/ui/form-field.tsx` dengan `FormField`, `Input`, `Select`, dan `Textarea`: sediakan label terhubung melalui `id`, teks bantuan, dan pesan kesalahan opsional melalui `aria-describedby`. Komponen meneruskan atribut native, termasuk `disabled`; jangan membuat form engine atau validasi bisnis. | | |
| TASK-023 | Setelah TASK-020, buat `app/components/ui/table.tsx` sebagai wrapper tabel HTML semantik dengan `thead`, `tbody`, `th`, dan `td` native; izinkan scroll horizontal pada layar sempit. Jangan menambah sorting, pagination, atau abstraksi data. | | |
| TASK-024 | Setelah TASK-020, buat `app/components/ui/feedback.tsx` dengan `LoadingState`, `ErrorState`, dan `EmptyState` yang menerima pesan. Gunakan status/alert yang sesuai dan hindari indikasi bahwa data gagal dimuat adalah kosong. | | |

**Kriteria fase:** TASK-020 sampai TASK-024 selesai; komponen tersedia untuk dipakai Phase 3 dan Phase 4, serta warna teks dan fokus dapat dibaca pada surface yang ditentukan.

### Implementation Phase 3

- GOAL-006: Menyiapkan application shell dan navigasi responsif.

**Dependensi fase:** semua task Phase 2 selesai.

| Task | Description | Completed | Date |
|------|-------------|-----------|------|
| TASK-025 | Setelah TASK-024, buat `app/components/layout/navigation.ts` berisi satu daftar route: `/` berlabel `Dashboard`, `/transactions` berlabel `Transaksi`, dan `/reports` berlabel `Laporan`. Gunakan daftar yang sama untuk sidebar dan breadcrumb; jangan membuat halaman pemain atau sesi. | | |
| TASK-026 | Setelah TASK-021 dan TASK-025, buat `app/components/layout/app-shell.tsx`, `app/components/layout/sidebar.tsx`, dan `app/components/layout/header.tsx`; pasang shell di `app/layout.tsx`. Sidebar menautkan tiga route dengan `next/link`, menandai route aktif dengan `aria-current`, dan pada lebar kurang dari 768px dibuka/ditutup dengan tombol bernama dan `aria-expanded`. Header menampilkan nama PB dan kontrol login nonaktif berlabel fungsi belum tersedia. Sisakan satu elemen `main` untuk konten halaman. | | |
| TASK-027 | Setelah TASK-025 dan TASK-026, buat `app/components/layout/breadcrumb.tsx`; gunakan daftar route yang sama, `nav` berlabel `Breadcrumb`, tautan `Dashboard` pada halaman selain `/`, dan `aria-current="page"` untuk halaman aktif. Integrasikan di shell tanpa menambahkan breadcrumb palsu. | | |

**Kriteria fase:** tiga tautan navigasi dan breadcrumb terhubung ke target Phase 4; shell dapat digunakan dengan keyboard dan responsif pada lebar 320px, 768px, dan 1280px.

### Implementation Phase 4

- GOAL-007: Menyelesaikan skeleton halaman dan memverifikasi gerbang sebelum integrasi.

**Dependensi fase:** semua task Phase 3 selesai.

| Task | Description | Completed | Date |
|------|-------------|-----------|------|
| TASK-028 | Setelah TASK-026 dan TASK-027, ubah `app/page.tsx` menjadi skeleton dashboard: tampilkan judul, kartu pemasukan, pengeluaran, dan saldo dengan nilai `—`, serta penjelasan bahwa data belum tersedia. Jangan menghitung atau menampilkan angka finansial. | | |
| TASK-029 | Setelah TASK-022, TASK-023, dan TASK-027, buat `app/transactions/page.tsx`: tampilkan tabel dengan kolom tanggal, jenis, jumlah, dan keterangan serta `EmptyState`; tampilkan form statis dengan tanggal, jenis pemasukan/pengeluaran, jumlah, dan keterangan. Nonaktifkan seluruh input dan tombol simpan, dan jelaskan bahwa pencatatan belum tersedia. | | |
| TASK-030 | Setelah TASK-022, TASK-024, dan TASK-027, buat `app/reports/page.tsx`: tampilkan judul laporan, input tanggal awal/akhir nonaktif, ringkasan pemasukan/pengeluaran periode dan saldo terkini dengan nilai `—`, serta tombol berbagi WhatsApp nonaktif dengan penjelasan belum tersedia. Buat `app/loading.tsx` menggunakan `LoadingState` dan `app/error.tsx` menggunakan `ErrorState` dengan pesan kesalahan jelas. Jangan membuat tautan WhatsApp atau perhitungan. | | |
| TASK-031 | Setelah TASK-028, TASK-029, dan TASK-030, tambahkan `@playwright/test@1.65.1` sebagai dev dependency, skrip `test:ui` dengan `playwright test`, `playwright.config.ts`, dan `tests/ui-foundation.spec.ts`. Batasi Playwright pada file tersebut dan exclude file tersebut di `vitest.config.ts` agar dua runner tidak saling menjalankan tes. Jalankan `yarn dev` melalui `webServer` Playwright; uji direct navigation ketiga route, tautan dan breadcrumb, status aktif, mobile toggle, disabled form/login/share, keyboard, serta tidak adanya request ke `/api/` saat membuka skeleton. | | |
| TASK-032 | Setelah TASK-031, jalankan `yarn lint`, `yarn tsc --noEmit`, `yarn test`, `yarn test:ui`, dan `yarn build`. Verifikasi tampilan pada 320px, 768px, dan 1280px, termasuk scroll tabel, focus, dan tidak ada overflow halaman. Catat hasil di ringkasan kerja; jika ada tes atau build gagal, blokir Phase 5. | | |

**Kriteria fase / gerbang:** TASK-019 sampai TASK-032 dan task Phase 1 selesai; seluruh perintah TASK-032 lulus. Ketiga halaman berfungsi sebagai navigasi statis tanpa login, data asli, backend/API, atau kontrol yang berpura-pura berhasil.

### Implementation Phase 5

- GOAL-002: Mengintegrasikan autentikasi dasar, API bertipe, dan pembacaan Sheet tanpa operasi tulis.

**Dependensi fase:** gerbang TASK-032 lulus; Phase 1 sampai Phase 4 selesai. Integrasi ini adalah implementasi Foundation yang sudah ada dalam PRD, bukan implementasi fitur transaksi.

| Task | Description | Completed | Date |
|------|-------------|-----------|------|
| TASK-005 | Setelah TASK-003, buat `lib/hexclave/client.ts` dan `lib/hexclave/server.ts` mengikuti dokumentasi SDK versi terpasang. Gunakan konfigurasi server hanya di server dan URL auth hosted; arahkan login dan logout kembali ke `/`. Sediakan sesi Google melalui OAuth client milik aplikasi, bukan shared OAuth keys. | | |
| TASK-006 | Setelah TASK-005, integrasikan `app/components/auth-controls.tsx` pada header dari TASK-026, ubah `app/layout.tsx` hanya untuk provider yang diperlukan, dan perluas `app/error.tsx` dari TASK-030 untuk kegagalan autentikasi: semua halaman publik dan skeleton tetap utuh; tampilkan tindakan masuk, status masuk, dan keluar; sediakan pesan pembatalan/kegagalan tanpa mengklaim izin editor. Pertahankan `cacheComponents: true` di `next.config.ts`; tempatkan pembacaan sesi request-time di bawah `Suspense` sesuai panduan Next.js. Perbarui `tests/ui-foundation.spec.ts` untuk autentikasi yang sebenarnya: stub layanan auth di batas tes, dan ganti hanya assertion login nonaktif serta larangan semua request API dari tahap skeleton; pertahankan verifikasi navigasi, form dan berbagi nonaktif. | | |
| TASK-007 | Setelah TASK-003, buat `lib/google/sheets.ts` dengan `readFoundationCell()`: gunakan `googleapis` dan kredensial default dari `GOOGLE_APPLICATION_CREDENTIALS`, scope `https://www.googleapis.com/auth/spreadsheets.readonly`, spreadsheet dari `GOOGLE_SHEETS_SPREADSHEET_ID`, serta range `GOOGLE_SHEETS_TEST_RANGE`. Baca satu sel dari Sheet uji; sukses hanya jika nilainya tepat `FOUNDATION_READY`. Pada error, kembalikan kategori aman tanpa isi respons Google atau nilai sel. Jangan menambah operasi tulis. | | |
| TASK-008 | Setelah TASK-002, buat `lib/trpc/init.ts`, `lib/trpc/router.ts`, dan `app/api/trpc/[trpc]/route.ts`. Ekspos hanya query publik `foundation.ping` yang mengembalikan `{ ok: true }` sebagai pemeriksaan proses aplikasi, bukan bukti koneksi layanan. Jangan mengekspos data Sheet, diagnostik, secret, atau mutasi. | | |
| TASK-009 | Setelah TASK-005 dan TASK-007, buat `scripts/diagnose.ts`: muat konfigurasi lokal melalui `@next/env`; periksa konfigurasi Stack Auth dengan operasi server SDK terautentikasi yang benar-benar memanggil layanan, tanpa mencetak hasil atau data akun; panggil `readFoundationCell()`. Cetak hanya nama pemeriksaan, status, dan kategori kesalahan aman; exit `0` hanya jika kedua pemeriksaan berhasil, selain itu `1`. Gunakan API server SDK yang tersedia pada versi terpasang; jika tidak ada pemeriksaan terautentikasi yang dapat dilakukan dengan aman, tandai tugas terblokir dan jelaskan alasannya. | | |
| TASK-010 | Setelah TASK-006, TASK-007, TASK-008, dan TASK-009, buat `tests/sheets.test.ts` dan `tests/trpc.test.ts`. Gunakan adapter Google palsu untuk menguji sel benar, sel salah/kosong, penolakan izin, dan layanan gagal; uji `foundation.ping` serta tidak adanya mutasi. Uji hasil error dan output diagnostik agar tidak mengandung kredensial atau data sel. | | |

**Kriteria fase:** tes TASK-010 lulus; jalur publik berfungsi; `yarn diagnose` tidak mengembalikan sukses jika salah satu koneksi gagal. Koneksi nyata tetap memerlukan Phase 7.

### Implementation Phase 6

- GOAL-003: Menyediakan image Docker dan dokumentasi operasional yang aman.

**Dependensi fase:** semua task Phase 5 selesai.

| Task | Description | Completed | Date |
|------|-------------|-----------|------|
| TASK-011 | Setelah TASK-006 dan TASK-008, buat `Dockerfile` berbasis `node:24.18.0-slim`: install dependensi dengan `yarn install --frozen-lockfile`, jalankan `yarn build`, lalu jalankan `yarn start` pada port `3000` sebagai pengguna non-root. Terima konfigurasi publik nonrahasia yang memang diperlukan Next.js sebagai build argument; secret hanya saat runtime. Jangan gunakan static export. | | |
| TASK-012 | Setelah TASK-011, buat `.dockerignore` untuk `.git`, `.next`, `node_modules`, file `.env*`, `*.pem`, dan direktori kredensial lokal; pastikan kredensial tidak disalin ke image. Dokumentasikan pemasangan file kredensial baca-saja ke container dan `GOOGLE_APPLICATION_CREDENTIALS` yang menunjuk ke lokasi mount. | | |
| TASK-013 | Setelah TASK-009 dan TASK-012, perbarui `README.md` tanpa menghapus struktur dan konvensi TASK-019: syarat Node/Yarn/Docker; konfigurasi `config/example.env`; setup Stack Auth cloud dan OAuth Google milik aplikasi; aktivasi API; spreadsheet uji privat dengan tab `Foundation`, sel `A1` bernilai `FOUNDATION_READY`, dan akun layanan viewer; perintah lokal, `yarn diagnose`, build/run container, penanganan kegagalan, serta pemisahan variabel build-time dan runtime. Jangan sertakan secret atau ID aktual. | | |

**Kriteria fase:** dokumentasi mencakup semua langkah dan tidak mengandung kredensial. Pada mesin dengan Docker dan konfigurasi yang sesuai, image dibangun dan halaman publik dapat diakses.

### Implementation Phase 7

- GOAL-004: Memvalidasi integrasi nyata dan mendokumentasikan hasil spike editor tanpa mengaktifkan fitur bisnis.

**Dependensi fase:** semua task Phase 6 selesai.

| Task | Description | Completed | Date |
|------|-------------|-----------|------|
| TASK-014 | Setelah TASK-010 dan TASK-013, jalankan `yarn lint`, `yarn tsc --noEmit`, `yarn test`, `yarn test:ui`, dan `yarn build`; catat status masing-masing. Dengan Docker tersedia, jalankan `docker build` dengan build argument publik yang dibutuhkan dan `docker run` dengan konfigurasi server serta kredensial mount read-only; verifikasi HTTP `/` dan hentikan container yang dibuat untuk tes. Jangan menampilkan nilai konfigurasi atau kredensial pada output bukti. | | |
| TASK-015 | Setelah TASK-014 dan ketersediaan DEP-003, uji langsung login Google, sesi, logout, dan pembatalan; jalankan `yarn diagnose` terhadap spreadsheet privat uji. Cabut lalu pulihkan hanya izin viewer akun layanan uji dan verifikasi diagnostik gagal lalu kembali berhasil. Dengan akun layanan uji yang sama, gunakan proses uji terpisah dan token berscope `https://www.googleapis.com/auth/spreadsheets` untuk mencoba menulis `WRITE_TEST` pada sel `Foundation!B1` yang semula kosong. Pastikan Google menolak operasi karena izin file, bukan karena scope baca. Jika operasi berhasil, tandai gagal, hentikan uji, dan minta operator menghapus nilai uji serta memperbaiki izin. Jangan gunakan scope tulis dalam aplikasi. | | |
| TASK-016 | Setelah TASK-015, lakukan spike terisolasi untuk OAuth client milik aplikasi: hubungkan akun Google login dengan scope `https://www.googleapis.com/auth/drive.file`, gunakan Picker untuk memilih spreadsheet uji yang dikonfigurasi, lalu coba memperoleh token pada backend dan panggil Drive `files.get` dengan `fields=capabilities(canEdit)` dan `supportsAllDrives=true`. Uji editor, viewer, izin editor dicabut, consent ditolak, serta token kedaluwarsa/dicabut. Pastikan pemeriksaan gagal tertutup bila izin tidak dapat dipastikan. Jangan menambah endpoint uji yang tersedia di produksi. | | |
| TASK-017 | Setelah TASK-016, buat `docs/ways-of-work/plan/pencatatan-keuangan-pb/foundation-initial-setup/editor-access-spike.md` berisi tanggal, versi SDK, scope, alur consent/Picker, hasil setiap skenario, perilaku refresh/revocation, batasan, dan keputusan `LAYAK` atau `BELUM LAYAK`. Jangan sertakan token, ID akun, isi spreadsheet, atau secret. Jika token server, akses file, atau pemeriksaan izin gagal dibuktikan, tetapkan `BELUM LAYAK` dan blokir fitur editor sampai keputusan arsitektur baru dibuat. | | |
| TASK-018 | Setelah TASK-014 sampai TASK-017, cocokkan hasil terhadap seluruh kriteria PRD Foundation. Catat per kriteria `LULUS`, `GAGAL`, atau `TERBLOKIR` di bagian hasil `editor-access-spike.md`, dengan alasan aman dan perintah atau skenario yang digunakan. Jangan tandai plan selesai jika ada `GAGAL` atau `TERBLOKIR`. | | |

**Kriteria fase:** semua tes otomatis dan skenario nyata lulus, hasil spike terdokumentasi, dan tidak ada operasi bisnis khusus editor. Hasil spike `BELUM LAYAK` dapat menjadi hasil valid dari investigasi, tetapi fitur editor tetap terblokir.

### Kontrak Setiap Task

Kolom `Scope` merujuk ke file atau operasi yang disebut pada baris task di atas. Kolom `Acceptance criteria` adalah syarat minimum tambahan; instruksi detail dan dependensi pada baris task tetap wajib.

| Referensi task | Objective | Scope | Acceptance criteria | Output |
|---|---|---|---|---|
| `TASK-001` | Menetapkan baseline. | Status Git, Node, Yarn, ESLint. | Versi dan hasil lint dicatat; perubahan pengguna tidak diubah. | Catatan baseline. |
| `TASK-002` | Menyiapkan dependensi integrasi dan tes. | `package.json`, `yarn.lock`. | Versi, skrip, dan peer dependency sesuai instruksi task. | Manifest dan lockfile. |
| `TASK-003` | Menyiapkan environment aman. | `config/example.env`, `lib/config.ts`. | Konfigurasi divalidasi saat digunakan; nama variabel hilang terlihat tanpa nilainya. | Contoh konfigurasi dan validator. |
| `TASK-004` | Menguji environment. | `vitest.config.ts`, `tests/config.test.ts`. | Tes konfigurasi lengkap, hilang, dan kebocoran nilai lulus. | Konfigurasi tes dan tes environment. |
| `TASK-019` | Menetapkan struktur dan konvensi. | `README.md`. | Struktur target dan aturan penamaan, TypeScript, alias, dan lint tertulis sesuai instruksi. | Bagian konvensi proyek. |
| `TASK-020` | Menyatukan tipografi dan warna. | `app/globals.css`, `app/layout.tsx`. | Font, ukuran, warna, focus ring, dan bahasa sesuai instruksi. | Design tokens dan root layout. |
| `TASK-021` | Menyediakan elemen umum. | `app/components/ui/button.tsx`, `card.tsx`. | Button mendukung dua varian, atribut native, focus, dan disabled; Card menampilkan children. | Button dan Card. |
| `TASK-022` | Menyediakan dasar formulir. | `app/components/ui/form-field.tsx`. | Input, Select, Textarea dan label terhubung, bantuan dan error dapat diakses, tanpa validasi bisnis. | FormField, Input, Select, dan Textarea. |
| `TASK-023` | Menyediakan tabel responsif. | `app/components/ui/table.tsx`. | Markup tabel semantik dan scroll horizontal bekerja tanpa sorting atau pagination. | Wrapper tabel. |
| `TASK-024` | Menyediakan feedback. | `app/components/ui/feedback.tsx`. | Loading, error, dan empty memiliki pesan serta peran status/alert yang sesuai. | Tiga komponen feedback. |
| `TASK-025` | Mendefinisikan navigasi. | `app/components/layout/navigation.ts`. | Tepat tiga route dan label sesuai task; tidak ada route pemain/sesi. | Daftar navigasi bersama. |
| `TASK-026` | Menyediakan shell responsif. | `app/layout.tsx`, `app/components/layout/app-shell.tsx`, `sidebar.tsx`, `header.tsx`. | Tautan aktif dan tombol menu aksesibel; header dan satu `main` tersedia pada tiga viewport. | Shell, sidebar, dan header. |
| `TASK-027` | Menyediakan breadcrumb. | `app/components/layout/breadcrumb.tsx`, shell. | Route aktif ditandai dan breadcrumb menggunakan daftar route bersama. | Breadcrumb terintegrasi. |
| `TASK-028` | Menyediakan skeleton dashboard. | `app/page.tsx`. | Tiga kartu dan penjelasan tampil; semua nilai `—`, tanpa perhitungan. | Halaman dashboard statis. |
| `TASK-029` | Menyediakan skeleton transaksi. | `app/transactions/page.tsx`. | Empat kolom dan empat field tampil; form dan simpan nonaktif dengan penjelasan. | Halaman transaksi statis. |
| `TASK-030` | Menyediakan skeleton laporan dan feedback route. | `app/reports/page.tsx`, `app/loading.tsx`, `app/error.tsx`. | Periode dan berbagi nonaktif; nilai `—`; loading dan error memakai komponen feedback. | Halaman laporan dan state route. |
| `TASK-031` | Memverifikasi interaksi dasar UI. | `package.json`, `yarn.lock`, `vitest.config.ts`, `playwright.config.ts`, `tests/ui-foundation.spec.ts`. | Tes route, navigasi, keyboard, disabled, dan larangan request API pada skeleton lulus; Vitest tidak menjalankan tes Playwright. | Konfigurasi dan tes Playwright. |
| `TASK-032` | Memastikan UI siap sebelum integrasi. | Perintah validasi dan browser tiga viewport. | Seluruh perintah dan pemeriksaan visual task lulus; skeleton tidak memanggil backend. | Bukti gerbang UI. |
| `TASK-005` | Menyiapkan koneksi SDK auth. | `lib/hexclave/client.ts`, `lib/hexclave/server.ts`. | Konfigurasi client/server terpisah dan OAuth memakai client aplikasi. | Adapter autentikasi. |
| `TASK-006` | Menghubungkan kontrol sesi ke shell. | `app/layout.tsx`, `app/components/auth-controls.tsx`, `app/error.tsx`, `tests/ui-foundation.spec.ts`. | Masuk/keluar dan error terlihat; skeleton tetap utuh; tidak ada klaim editor; tes UI disesuaikan. | Kontrol sesi dan tes regresi UI. |
| `TASK-007` | Membaca sel uji privat. | `lib/google/sheets.ts`. | Hanya nilai sentinel menghasilkan sukses; error aman dan tidak ada operasi tulis. | Adapter Sheets baca-saja. |
| `TASK-008` | Menyiapkan API bertipe minimal. | `lib/trpc/init.ts`, `lib/trpc/router.ts`, `app/api/trpc/[trpc]/route.ts`. | Hanya `foundation.ping` diekspos; tidak ada mutasi atau diagnostik publik. | Router dan handler tRPC. |
| `TASK-009` | Mendiagnosis layanan secara lokal. | `scripts/diagnose.ts`. | Exit `0` hanya ketika kedua koneksi berhasil; output aman dan nonrahasia. | CLI diagnostik. |
| `TASK-010` | Menguji adapter dan API. | `tests/sheets.test.ts`, `tests/trpc.test.ts`. | Kasus berhasil dan gagal, tidak ada mutasi, dan output aman lulus. | Tes integrasi terisolasi. |
| `TASK-011` | Menyiapkan image aplikasi. | `Dockerfile`. | Image menggunakan Node yang ditentukan, build Next.js, non-root, dan tidak menerima secret saat build. | Dockerfile. |
| `TASK-012` | Melindungi build context. | `.dockerignore`, instruksi mount. | File environment, sertifikat, kredensial lokal, dependensi, dan output build tidak disalin ke image. | `.dockerignore` dan ketentuan mount. |
| `TASK-013` | Mendokumentasikan operasi. | `README.md`. | Setup, perintah, autentikasi, Sheet uji, Docker, error, dan konvensi terdokumentasi tanpa secret. | README operasional. |
| `TASK-014` | Memvalidasi build dan container. | Toolchain, Docker, tiga route publik. | Lima perintah lulus; container build/run dan HTTP berhasil jika Docker tersedia. | Hasil validasi dan bukti container. |
| `TASK-015` | Membuktikan koneksi nyata dan hak baca. | Stack Auth, Google, Sheet dan akun uji. | Login/logout, kegagalan dan pemulihan baca, serta penolakan tulis karena izin file terbukti. | Hasil uji layanan nyata. |
| `TASK-016` | Menilai kelayakan akses editor. | OAuth, Picker, token server, Drive API, akun uji. | Seluruh skenario editor/viewer/revocation/consent diuji dan error gagal tertutup. | Bukti spike nonrahasia. |
| `TASK-017` | Mendokumentasikan hasil spike. | `docs/ways-of-work/plan/pencatatan-keuangan-pb/foundation-initial-setup/editor-access-spike.md`. | Hasil per skenario, scope, refresh/revocation, batasan, dan keputusan tercatat tanpa secret. | Dokumen spike. |
| `TASK-018` | Memverifikasi seluruh PRD Foundation. | Kriteria PRD dan dokumen spike. | Tiap kriteria memiliki status dan bukti; task gagal atau terblokir tidak ditandai selesai. | Matriks hasil penerimaan. |

## 3. Alternatives

- **ALT-001**: Menggunakan spreadsheet publik langsung dari browser ditolak karena kredensial/akses data harus dikendalikan server dan spreadsheet uji tetap privat.
- **ALT-002**: Menggunakan akun layanan berizin tulis untuk semua pengguna ditolak karena login saja tidak membuktikan izin editor.
- **ALT-003**: Menjalankan Stack Auth sendiri ditunda karena cloud dipilih dan self-host menambah PostgreSQL, ClickHouse, scheduler, dan operasi.
- **ALT-004**: Menggunakan mock untuk menyatakan integrasi nyata selesai ditolak karena tidak membuktikan OAuth, izin Google, atau jaringan.
- **ALT-005**: Menggunakan angka finansial contoh pada skeleton ditolak agar pengunjung tidak menganggap data contoh sebagai saldo PB.

## 4. Dependencies

- **DEP-001**: Node 24.18.0, Yarn 1.22.22, serta dependensi dan versi dalam TASK-002; paket perlu dipasang saat implementasi, bukan saat penyusunan rencana.
- **DEP-002**: Docker Engine yang dapat membangun dan menjalankan container; pada saat rencana disusun, perintah `docker` tidak tersedia di lingkungan ini.
- **DEP-003**: Operator menyediakan Stack Auth cloud, OAuth client Google milik aplikasi, Google Sheets API, Google Drive API, Google Picker API, spreadsheet uji privat, akun layanan viewer, dan akun editor serta viewer uji. Jangan meminta atau menyalin secret melalui chat; gunakan mekanisme secret lingkungan.
- **DEP-004**: Akses jaringan ke Stack Auth dan Google untuk uji integrasi nyata; izin untuk mengubah akses hanya pada spreadsheet uji.
- **DEP-005**: `@playwright/test@1.65.1` dan browser Chromium Playwright tersedia saat TASK-031 dan TASK-032; instalasi browser dilakukan saat implementasi, bukan saat penyusunan rencana.

## 5. Files

- **FILE-001**: `package.json`, `yarn.lock`, `vitest.config.ts`, dan `config/example.env`: dependensi, skrip, tes, dan nama konfigurasi.
- **FILE-002**: `lib/config.ts`, `lib/hexclave/client.ts`, `lib/hexclave/server.ts`, `lib/google/sheets.ts`, `scripts/diagnose.ts`: konfigurasi dan adapter layanan.
- **FILE-003**: `lib/trpc/init.ts`, `lib/trpc/router.ts`, `app/api/trpc/[trpc]/route.ts`: API bertipe dengan query proses nonrahasia.
- **FILE-004**: `app/layout.tsx`, `app/page.tsx`, `app/components/auth-controls.tsx`, `app/error.tsx`: jalur publik dan UI autentikasi dasar.
- **FILE-005**: `Dockerfile`, `.dockerignore`, `README.md`: image dan panduan operasi.
- **FILE-006**: `tests/config.test.ts`, `tests/sheets.test.ts`, `tests/trpc.test.ts`: tes otomatis Foundation.
- **FILE-007**: `docs/ways-of-work/plan/pencatatan-keuangan-pb/foundation-initial-setup/editor-access-spike.md`: hasil uji nyata, batasan, dan status penerimaan.
- **FILE-008**: `app/globals.css`, `app/components/ui/button.tsx`, `card.tsx`, `form-field.tsx`, `table.tsx`, dan `feedback.tsx`: token dan komponen dasar.
- **FILE-009**: `app/components/layout/navigation.ts`, `app-shell.tsx`, `sidebar.tsx`, `header.tsx`, dan `breadcrumb.tsx`: shell dan navigasi.
- **FILE-010**: `app/transactions/page.tsx`, `app/reports/page.tsx`, `app/loading.tsx`, dan `app/error.tsx`: skeleton halaman dan feedback.
- **FILE-011**: `playwright.config.ts`, `tests/ui-foundation.spec.ts`: pengujian UI dan routing.

## 6. Testing

- **TEST-001**: Tes otomatis pada TASK-004 dan TASK-010 mencakup konfigurasi hilang, data uji benar/salah, layanan gagal, error aman, dan tidak adanya mutasi.
- **TEST-002**: Jalankan seluruh gate pada TASK-014; jangan mengganti kegagalan Docker dengan klaim keberhasilan.
- **TEST-003**: TASK-015 memverifikasi login/logout, pembatalan, akses baca tanpa login pengunjung, pencabutan akses, dan penolakan tulis akun layanan pada layanan nyata.
- **TEST-004**: TASK-016 memverifikasi scope, Picker, token server, `canEdit`, viewer, pencabutan izin, consent ditolak, dan kedaluwarsa/pencabutan token.
- **TEST-005**: Periksa halaman publik serta kontrol login pada viewport desktop dan seluler; uji label dan pesan kegagalan tanpa mengklaim izin editor.
- **TEST-006**: TASK-031 dan TASK-032 menguji tiga route, tautan langsung, active state, breadcrumb, keyboard, menu seluler, komponen disabled, dan tidak adanya request `/api/` sebelum integrasi.
- **TEST-007**: TASK-032 memeriksa layout pada 320px, 768px, dan 1280px; semua konten dapat dijangkau, tabel dapat di-scroll, dan tidak ada overflow halaman.
- **TEST-008**: TASK-014 mengulang tes UI setelah integrasi; stub auth pada batas tes tidak menggantikan uji login nyata pada TASK-015.

## 7. Risks & Assumptions

- **RISK-001**: Stack Auth mungkin tidak menyediakan token server atau perilaku refresh yang dibutuhkan dalam alur Picker; TASK-017 harus menandai `BELUM LAYAK` jika tidak terbukti.
- **RISK-002**: Scope `drive.file` tidak otomatis memberi aplikasi akses ke spreadsheet yang sudah dibagikan dengan pengguna; pemilihan file dan consent perlu diuji.
- **RISK-003**: Kegagalan jaringan, persetujuan OAuth, kuota Google, atau verifikasi aplikasi dapat memblokir uji nyata. Jangan menganggap mock sebagai pengganti.
- **RISK-004**: `NEXT_PUBLIC_*` di-inline saat build; image tidak dapat memakai nilai publik baru hanya dengan mengganti environment runtime.
- **RISK-005**: Docker tidak tersedia pada lingkungan saat rencana disusun; verifikasi container terblokir sampai tersedia.
- **RISK-006**: Skeleton dapat disalahartikan sebagai fitur keuangan yang sudah siap; tampilkan status belum tersedia dan jangan gunakan data asli atau kontrol bisnis aktif.
- **ASSUMPTION-001**: Operator dapat menyediakan akun dan spreadsheet uji serta mengizinkan perubahan sementara pada izin spreadsheet uji.
- **ASSUMPTION-002**: Dependensi yang tercantum tersedia dan kompatibel pada saat implementasi; jika tidak, hentikan tugas dan catat ketidakcocokan sebelum mengubah versi.
- **ASSUMPTION-003**: Satu spreadsheet uji digunakan per deployment; struktur transaksi produksi belum ditentukan.
- **ASSUMPTION-004**: `/reports` adalah route presentasi untuk kebutuhan laporan yang sudah tercantum di epic, bukan penambahan requirement fitur baru.

## 8. Related Specifications / Further Reading

- [PRD Foundation](../docs/ways-of-work/plan/pencatatan-keuangan-pb/foundation-initial-setup/prd.md)
- [PRD Epic](../docs/ways-of-work/plan/pencatatan-keuangan-pb/epic.md)
- [Arsitektur Epic](../docs/ways-of-work/plan/pencatatan-keuangan-pb/arch.md)
- [Stack Auth / Hexclave connected accounts](https://docs.hexclave.com/guides/apps/authentication/connected-accounts)
- [Google Drive scopes](https://developers.google.com/workspace/drive/api/guides/api-specific-auth)
- [Google Picker](https://developers.google.com/workspace/drive/picker/guides/overview)
- [Google Drive capabilities](https://developers.google.com/workspace/drive/api/guides/manage-sharing)
- [Next.js: layouts dan pages](../node_modules/next/dist/docs/01-app/01-getting-started/03-layouts-and-pages.md)
- [Next.js: authentication dengan Cache Components](../node_modules/next/dist/docs/01-app/02-guides/authentication-with-cache-components.md)
