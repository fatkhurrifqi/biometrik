# Product Requirements Document (PRD)

# Sistem Absensi Biometrik Berbasis Web (Absen Mandiri via HP Karyawan)

| Field                 | Detail                                 |
| --------------------- | -------------------------------------- |
| Nama Produk           | SiAbsen Bio (Sistem Absensi Biometrik) |
| Versi Dokumen         | 2.0                                    |
| Status                | Draft                                  |
| Tech Stack (Frontend) | React + Vite                           |
| Pembuat               | Product/Engineering Team               |

---

## 1. Latar Belakang

Perusahaan/instansi membutuhkan sistem absensi yang akurat, anti-titip absen (buddy punching), dan tidak memerlukan alat khusus (kiosk/mesin fisik) di kantor. Karyawan absen menggunakan **HP masing-masing**, memanfaatkan sensor biometrik yang sudah tersedia di HP tersebut (sidik jari/Face ID) ditambah verifikasi wajah melalui kamera HP dan validasi lokasi (GPS), sehingga proses absensi tetap sulit dimanipulasi tanpa perlu investasi perangkat keras tambahan.

## 2. Tujuan (Objectives)

1. Menyediakan sistem absensi mandiri (self-service) berbasis HP karyawan, tanpa perlu alat kiosk khusus.
2. Menggunakan verifikasi biometrik ganda: **konfirmasi sensor bawaan HP** (sidik jari/Face ID via WebAuthn) **dan** **selfie wajah** yang dicocokkan dengan foto profil karyawan.
3. Mencegah absen titip/palsu dengan validasi lokasi (GPS) agar absen hanya bisa dilakukan di area kantor.
4. Memberikan dashboard real-time untuk HR/Admin memantau kehadiran karyawan.
5. Otomatisasi perhitungan jam kerja, keterlambatan, lembur, dan rekap absensi bulanan.
6. Membatasi jumlah karyawan terdaftar sesuai kuota yang ditetapkan (default 30 karyawan, dapat diubah admin).
7. Menyediakan laporan absensi yang dapat diekspor untuk keperluan payroll.
8. Dibangun sepenuhnya menggunakan **layanan/tools gratis (open-source & free tier)** agar dapat di-deploy dan diakses publik tanpa biaya operasional.

## 3. Ruang Lingkup (Scope)

### In-Scope

- Web application (React + Vite), diakses via browser HP karyawan (mobile web/PWA), terdiri dari:
  - Portal Admin/HR (manajemen karyawan, laporan, konfigurasi jam kerja, kuota karyawan)
  - Portal Karyawan / Halaman Absen Mandiri (absen masuk/pulang via HP sendiri)
- Backend API untuk autentikasi, manajemen data, dan pemrosesan absensi.
- Verifikasi biometrik memakai kemampuan bawaan HP (browser API), **bukan** perangkat/kiosk fisik tambahan:
  - **WebAuthn** — konfirmasi identitas via sensor sidik jari/Face ID yang sudah ada di HP.
  - **Kamera HP** — ambil selfie, dicocokkan dengan foto profil (face matching).
  - **GPS HP** — memastikan karyawan absen dari lokasi kantor (geofencing).
- Notifikasi (email/WhatsApp/push) untuk keterlambatan atau ketidakhadiran.
- Modul laporan & ekspor (PDF/Excel).
- Manajemen kuota jumlah karyawan yang boleh terdaftar di sistem.

### Out-of-Scope (Fase 1)

- Aplikasi mobile native (Android/iOS) — Fase 1 cukup web/PWA yang diakses lewat browser HP.
- Integrasi payroll via API real-time (Fase 1 hanya menyediakan ekspor data; API real-time menjadi rencana Fase 2, lihat Bagian 7.4).
- Dukungan multi-lokasi/multi-cabang — Fase 1 hanya untuk **1 lokasi kantor**.
- Perangkat kiosk/mesin fisik khusus — tidak dibutuhkan, karena absen memakai HP karyawan sendiri.

### Catatan Skala Sistem

- Sistem dirancang untuk **1 lokasi kantor** saja pada Fase 1.
- Sistem membatasi jumlah total karyawan yang dapat terdaftar melalui **kuota yang dapat dikonfigurasi admin** (misal maksimal 50/100/200 karyawan, tergantung paket/kapasitas server).
- Sistem **tidak bergantung pada perangkat keras tambahan** — cukup HP karyawan (Android/iOS) dengan browser modern yang mendukung kamera, GPS, dan WebAuthn.

## 4. Target Pengguna (User Personas)

| Persona            | Deskripsi                                   | Kebutuhan Utama                                           |
| ------------------ | ------------------------------------------- | --------------------------------------------------------- |
| Admin/HR           | Mengelola data karyawan & kebijakan absensi | Dashboard, laporan, konfigurasi jam kerja, kuota karyawan |
| Karyawan           | Melakukan absensi harian lewat HP sendiri   | Proses absen cepat & mudah, riwayat transparan            |
| Supervisor/Manager | Memantau tim                                | Laporan tim, approval izin/cuti                           |
| IT/Superadmin      | Mengelola sistem                            | Manajemen akun, log sistem, keamanan, kuota               |

## 5. User Stories & Functional Requirements

### 5.1 Autentikasi & Manajemen Akun

- FR-01: Admin dapat membuat, mengedit, menonaktifkan akun karyawan.
- FR-02: Login menggunakan email/username + password dengan role-based access control (RBAC): Superadmin, Admin/HR, Supervisor, Karyawan.
- FR-03: Sistem mendukung reset password.
- FR-03a: Sistem membatasi jumlah akun karyawan aktif sesuai **kuota** yang ditetapkan (default: 30 karyawan pada Fase 1); jika kuota penuh, admin harus menonaktifkan/menghapus akun lama sebelum menambah karyawan baru, atau menaikkan kuota melalui pengaturan (kuota dapat diubah naik/turun kapan saja oleh admin).

### 5.2 Registrasi Perangkat & Biometrik (via HP Karyawan)

- FR-04: Saat pertama kali login, karyawan diminta melakukan **registrasi HP** yang akan dipakai untuk absen (device binding), termasuk:
  - Mendaftarkan konfirmasi biometrik HP (sidik jari/Face ID) melalui WebAuthn.
  - Mengunggah/mengambil foto wajah acuan (foto profil) untuk keperluan pencocokan selfie saat absen.
- FR-05: Sistem menyimpan kredensial WebAuthn (bukan data sidik jari asli — data itu tetap tersimpan di HP karyawan, sistem hanya menerima "bukti konfirmasi" dari HP) dan template wajah terenkripsi hasil ekstraksi dari foto profil.
- FR-06: Karyawan dapat mengganti/mendaftar ulang HP jika ganti perangkat (dengan approval admin, untuk mencegah penyalahgunaan).
- FR-07: Satu akun karyawan hanya terhubung ke satu HP terdaftar dalam satu waktu (mencegah satu HP dipakai absenkan banyak orang / titip absen).

### 5.3 Proses Absensi Mandiri (via HP Karyawan)

- FR-08: Karyawan membuka halaman absen di browser HP-nya, lalu melakukan 2 langkah verifikasi:
  1. **Konfirmasi biometrik HP** (sidik jari/Face ID lewat WebAuthn) — memastikan HP sedang dipegang pemiliknya.
  2. **Selfie wajah** — dicocokkan dengan foto profil (face matching) untuk memastikan wajah yang absen sesuai data karyawan.
- FR-09: Sistem memeriksa **lokasi GPS HP** saat absen; absen hanya diterima jika berada dalam radius kantor yang dikonfigurasi (misal 100 meter dari titik kantor).
- FR-10: Kedua verifikasi (WebAuthn + selfie) **wajib berhasil**, dan lokasi wajib valid, sebelum absen tercatat. Kebijakan ini berlaku sama untuk seluruh karyawan (tidak ada pengecualian per individu).
- FR-11: Jika salah satu verifikasi gagal (WebAuthn tidak cocok, wajah tidak dikenali, atau di luar radius kantor), sistem menampilkan pesan error yang jelas dan menawarkan opsi coba lagi; jika berulang kali gagal, karyawan dapat mengajukan **absen manual** yang wajib disetujui oleh **Admin/HR** (bukan supervisor tim) sebelum tercatat sebagai kehadiran sah.
- FR-12: Sistem mendeteksi status: Hadir, Terlambat, Pulang Cepat, berdasarkan jam kerja yang dikonfigurasi.
- FR-13: Menampilkan konfirmasi visual (nama karyawan + foto selfie + waktu) setelah absen berhasil.

### 5.4 Manajemen Jadwal & Kebijakan

- FR-14: Admin dapat mengatur shift kerja (reguler, shift, fleksibel).
- FR-15: Admin dapat mengatur toleransi keterlambatan, jam istirahat, dan hari libur/cuti bersama.
- FR-16: Admin dapat menetapkan jadwal berbeda per departemen/individu.
- FR-16a: Admin dapat mengatur titik lokasi kantor (koordinat GPS) dan radius toleransi absen (geofencing).

### 5.5 Pengajuan Izin, Cuti, dan Lembur

- FR-17: Karyawan dapat mengajukan izin/cuti/sakit melalui portal self-service dengan upload dokumen pendukung.
- FR-18: Supervisor dapat menyetujui/menolak pengajuan dengan catatan.
- FR-19: Pengajuan lembur dicatat dan dihitung otomatis ke rekap jam kerja.

### 5.6 Dashboard & Laporan

- FR-20: Dashboard admin menampilkan ringkasan kehadiran real-time (hadir/telat/absen/cuti) per hari.
- FR-21: Laporan bulanan per karyawan/departemen dapat diekspor ke Excel/PDF.
- FR-22: Grafik tren kehadiran & keterlambatan.
- FR-23: Log audit setiap perubahan data absensi manual (siapa, kapan, alasan).
- FR-23a: Dashboard menampilkan sisa kuota karyawan (misal "45/50 karyawan terdaftar").

### 5.7 Notifikasi

- FR-24: Notifikasi otomatis ke karyawan jika belum absen pulang setelah jam kerja selesai.
- FR-25: Notifikasi ke supervisor jika ada anggota tim yang belum hadir setelah batas toleransi.
- FR-25a: Notifikasi ke admin saat kuota karyawan mendekati batas maksimal.

### 5.8 Integrasi

- FR-26: Ekspor data absensi ke format Excel/PDF untuk keperluan input manual ke sistem payroll.
- FR-27: (Fase 2, opsional) Webhook/API untuk event penting (absen masuk, absen pulang, gagal verifikasi), jika di masa depan sistem payroll sudah mendukung integrasi otomatis.

## 6. Non-Functional Requirements (NFR)

| Kategori          | Requirement                                                                                                                                           |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Performa          | Proses absen (WebAuthn + selfie + validasi GPS) selesai < 5 detik total; dashboard load < 3 detik                                                     |
| Skalabilitas      | Mendukung kuota karyawan yang dapat dikonfigurasi (contoh Fase 1: hingga 200 karyawan dalam 1 lokasi)                                                 |
| Keamanan          | Kredensial WebAuthn & template wajah terenkripsi at-rest & in-transit (AES-256, TLS 1.2+); kepatuhan pada regulasi perlindungan data pribadi (UU PDP) |
| Ketersediaan      | Uptime sistem ≥ 99.5%                                                                                                                                 |
| Kompatibilitas    | Mendukung browser mobile modern (Chrome/Safari) di Android & iOS yang mendukung WebAuthn, kamera (getUserMedia), dan Geolocation API                  |
| Audit & Kepatuhan | Semua akses ke data biometrik/lokasi tercatat dalam audit log                                                                                         |
| Usability         | Alur absen sederhana (maksimal 3 langkah), dapat digunakan tanpa training                                                                             |
| Anti-spoofing     | Sistem melakukan liveness check dasar pada selfie (mendeteksi foto dari foto/video agar tidak mudah dipalsukan)                                       |

## 7. Arsitektur Teknis (Diusulkan)

### 7.1 Frontend

- **Framework**: React + Vite
- **State Management**: Redux Toolkit atau Zustand
- **Routing**: React Router
- **UI Library**: Tailwind CSS + shadcn/ui atau Ant Design
- **Data Fetching**: React Query (TanStack Query)
- **PWA**: dikonfigurasi sebagai Progressive Web App agar dapat "diinstal" ke home screen HP karyawan seperti aplikasi native, meski tetap berbasis web.
- **Browser API yang dipakai untuk absen mandiri**:
  - `navigator.credentials` (WebAuthn) — konfirmasi sidik jari/Face ID bawaan HP.
  - `navigator.mediaDevices.getUserMedia` — akses kamera untuk selfie.
  - `navigator.geolocation` — ambil koordinat GPS untuk validasi lokasi kantor.

### 7.2 Backend

- **API**: Node.js (Express/NestJS) atau sesuai preferensi tim.
- **Database**: PostgreSQL.
- **Object Storage**: menyimpan foto selfie & foto profil (bukan data biometrik mentah dari sensor HP).
- **Auth**: JWT + refresh token, RBAC middleware, plus verifikasi WebAuthn di sisi server (menyimpan public key credential, bukan data sidik jari).
- **Face Matching**: menggunakan **face-api.js** (library open-source, gratis, berjalan di sisi browser/client menggunakan model TensorFlow.js) untuk mencocokkan selfie dengan foto profil — tidak memerlukan layanan cloud berbayar seperti AWS Rekognition/Azure Face API.
- **Realtime**: WebSocket (Socket.io) untuk update dashboard admin secara real-time.

### 7.3 Rencana Hosting/Deployment (Gratis)

Agar sistem dapat diakses publik tanpa biaya operasional (sesuai kebutuhan), digunakan kombinasi layanan gratis (free tier) berikut:

| Komponen                | Layanan yang Diusulkan                          | Catatan                                                                                                                                                |
| ----------------------- | ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Frontend (React + Vite) | Vercel atau Netlify                             | Deploy otomatis dari repository Git, gratis untuk trafik skala kecil-menengah                                                                          |
| Backend API             | Render.com (free tier) atau Railway (free tier) | Cukup untuk skala kuota ~30 karyawan; perlu diperhatikan free tier biasanya "tidur" saat tidak ada trafik dan butuh beberapa detik untuk aktif kembali |
| Database                | Supabase (PostgreSQL gratis) atau Firebase      | Free tier cukup untuk skala kecil (puluhan karyawan, ribuan baris data absensi)                                                                        |
| Face Matching           | face-api.js (client-side, gratis)               | Tidak ada biaya API karena proses berjalan di browser pengguna                                                                                         |

Catatan: karena menggunakan free tier, perlu diperhatikan adanya batasan kapasitas (misal limit request per bulan, database size, atau server yang "tidur" saat idle). Ini sudah cukup untuk skala 1 lokasi dengan kuota puluhan karyawan, namun jika di masa depan skala bertambah besar, mungkin perlu upgrade ke paket berbayar.

### 7.4 Alur Verifikasi Absen (Ringkas)

```
[HP Karyawan]
   ├─ Sensor bawaan HP (sidik jari/Face ID) → WebAuthn → Backend (verifikasi kredensial)
   ├─ Kamera HP → Selfie → Backend (face matching vs foto profil)
   └─ GPS HP → Koordinat lokasi → Backend (cek radius kantor)
                     ↓
        Semua verifikasi valid? → Catat AttendanceLog → Dashboard Admin Real-time
```

Catatan penting: WebAuthn tidak pernah mengirim data sidik jari asli ke server — yang dikirim hanya "bukti kriptografis" bahwa sensor HP sudah berhasil mencocokkan pemiliknya. Data sidik jari/Face ID tetap tersimpan aman di dalam HP masing-masing karyawan, tidak pernah keluar dari perangkat. Ini juga mengurangi beban tanggung jawab keamanan data biometrik di sisi server.

### 7.5 Rencana Integrasi Payroll (Fase Bertahap)

- **Fase 1 (cakupan dokumen ini)**: sistem menyediakan fitur **ekspor data absensi** (rekap kehadiran, keterlambatan, lembur, izin/cuti per periode) dalam format Excel/PDF, yang kemudian diunggah manual ke sistem payroll perusahaan.
- **Fase 2 (rencana pengembangan lanjutan, di luar cakupan Fase 1)**: apabila sistem payroll yang digunakan di masa depan sudah mendukung API, dapat dibangun modul integrasi otomatis (REST API/webhook).
- Pendekatan bertahap ini dipilih karena sistem payroll spesifik yang akan digunakan belum ditentukan; desain ekspor generik (Excel/PDF/CSV) memastikan data absensi tetap bisa dipakai oleh payroll software apa pun.

### 7.6 Struktur Folder Frontend (Contoh)

```
src/
├── app/                 # App config, router, providers
├── features/
│   ├── auth/
│   ├── attendance/      # Halaman & logic absen mandiri (WebAuthn, selfie, GPS)
│   ├── employees/
│   ├── leave/
│   └── reports/
├── components/          # Shared UI components
├── hooks/
├── services/            # API clients, webauthn helper, geolocation helper
├── store/               # State management
└── utils/
```

## 8. Data Model (Ringkasan Entitas Utama)

| Entitas          | Atribut Utama                                                                                                                                          |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| User/Employee    | id, nama, email, role, departemen, status, foto_profil_url                                                                                             |
| DeviceCredential | id, employee_id, webauthn_credential_id, public_key, device_label, registered_at                                                                       |
| AttendanceLog    | id, employee_id, tipe (in/out), timestamp, status (hadir/telat/manual), webauthn_verified (bool), face_match_score, gps_lat, gps_lng, gps_valid (bool) |
| OfficeLocation   | id, nama_lokasi, latitude, longitude, radius_meter                                                                                                     |
| Shift            | id, nama_shift, jam_mulai, jam_selesai, toleransi_menit                                                                                                |
| LeaveRequest     | id, employee_id, tipe (izin/cuti/sakit), tanggal_mulai, tanggal_selesai, status, dokumen_url                                                           |
| Quota            | id, max_employees, current_employees, updated_at                                                                                                       |
| AuditLog         | id, user_id, aksi, target, timestamp, keterangan                                                                                                       |

## 9. Alur Utama (Key User Flows)

### 9.1 Flow Absensi Mandiri (via HP Karyawan)

1. Karyawan membuka halaman absen di browser HP (bisa lewat PWA yang sudah di-install di home screen).
2. Sistem meminta konfirmasi sidik jari/Face ID bawaan HP (WebAuthn).
3. Sistem meminta ambil selfie via kamera HP.
4. Sistem meminta akses lokasi (GPS) untuk validasi radius kantor.
5. Jika ketiganya valid → absen tercatat, tampilkan konfirmasi (nama, foto selfie, waktu, status hadir/telat).
6. Jika salah satu gagal → tampilkan pesan error spesifik + opsi coba lagi atau ajukan absen manual.

### 9.2 Flow Registrasi HP & Biometrik (Sekali di Awal)

1. Karyawan login pertama kali dengan akun yang dibuat admin.
2. Sistem meminta karyawan mendaftarkan HP: konfirmasi sidik jari/Face ID (WebAuthn) + ambil foto profil.
3. Data kredensial WebAuthn & template wajah tersimpan terenkripsi dan tertaut ke akun karyawan.
4. Karyawan siap menggunakan HP tersebut untuk absen mandiri sehari-hari.

### 9.3 Flow Pengajuan Izin

1. Karyawan login ke portal self-service.
2. Isi form pengajuan izin/cuti + upload dokumen.
3. Supervisor menerima notifikasi → review → approve/reject.
4. Status pengajuan otomatis memengaruhi rekap absensi.

## 10. Keamanan & Privasi Data Biometrik

- Data sidik jari/Face ID **tidak pernah meninggalkan HP karyawan** — server hanya menerima bukti kriptografis dari WebAuthn, bukan data biometrik mentah.
- Foto selfie & foto profil disimpan terenkripsi; hanya digunakan untuk proses pencocokan (face matching), bukan disebarluaskan.
- Data lokasi (GPS) hanya disimpan sebagai bagian dari log absensi, tidak dipakai untuk pelacakan di luar jam absen.
- Kebijakan retensi data: data biometrik/kredensial dihapus otomatis saat karyawan resign, sesuai regulasi perlindungan data pribadi.
- Perlu persetujuan tertulis (consent) dari karyawan sebelum registrasi HP/biometrik, sesuai UU Perlindungan Data Pribadi (UU PDP) di Indonesia.
- Role-based access: hanya Admin/IT tertentu yang dapat mengakses menu registrasi/hapus data biometrik karyawan.

## 11. Metrik Keberhasilan (Success Metrics)

| Metrik                                                 | Target                                                                                           |
| ------------------------------------------------------ | ------------------------------------------------------------------------------------------------ |
| Akurasi verifikasi (WebAuthn + face match)             | ≥ 98%                                                                                            |
| Waktu rata-rata proses absen                           | ≤ 5 detik                                                                                        |
| Pengurangan kasus titip absen                          | Signifikan (secara teknis sulit dilakukan karena butuh HP terdaftar + wajah + lokasi yang cocok) |
| Adopsi pengguna (karyawan aktif memakai absen mandiri) | ≥ 90% dalam 1 bulan setelah rilis                                                                |
| Downtime sistem                                        | ≤ 0.5% per bulan                                                                                 |
| Kepatuhan kuota karyawan                               | 0% insiden melebihi kuota tanpa persetujuan admin                                                |

## 12. Milestone & Timeline (Estimasi)

| Fase                  | Durasi     | Deliverable                                                             |
| --------------------- | ---------- | ----------------------------------------------------------------------- |
| Discovery & Desain    | 1–2 minggu | Wireframe, arsitektur final, pemilihan layanan face matching            |
| Development Sprint 1  | 3 minggu   | Auth, manajemen karyawan, kuota, registrasi HP (WebAuthn + foto profil) |
| Development Sprint 2  | 3 minggu   | Halaman absen mandiri (WebAuthn + selfie + GPS), attendance logging     |
| Development Sprint 3  | 2 minggu   | Dashboard, laporan, notifikasi                                          |
| Development Sprint 4  | 2 minggu   | Modul izin/cuti, ekspor data payroll                                    |
| Testing (QA + UAT)    | 2 minggu   | Bug fixing, uji keamanan data biometrik & lokasi                        |
| Deployment & Training | 1 minggu   | Rilis produksi, pelatihan admin & karyawan                              |

## 13. Risiko & Mitigasi

| Risiko                                                      | Dampak                                   | Mitigasi                                                                                                               |
| ----------------------------------------------------------- | ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| HP karyawan tidak mendukung WebAuthn (HP lama/browser lama) | Karyawan tidak bisa registrasi/absen     | Sediakan daftar HP/browser minimum yang didukung; fallback absen manual dengan approval admin                          |
| Selfie dipalsukan pakai foto/video orang lain               | Absen tidak valid tercatat sebagai valid | Terapkan liveness check dasar pada proses selfie                                                                       |
| GPS tidak akurat (di dalam gedung, sinyal lemah)            | Karyawan gagal absen meski di kantor     | Atur radius toleransi yang wajar (misal 100–150 meter), sediakan opsi absen manual dengan approval jika GPS bermasalah |
| Karyawan ganti HP tanpa lapor admin                         | Tidak bisa absen sampai registrasi ulang | Sediakan flow "daftar ulang HP" yang mudah namun tetap perlu approval admin untuk keamanan                             |
| Kuota karyawan penuh saat rekrutmen baru                    | Karyawan baru tidak bisa didaftarkan     | Notifikasi otomatis saat kuota mendekati batas (FR-25a), admin bisa menaikkan kuota                                    |
| Kebocoran data foto profil/selfie                           | Pelanggaran privasi                      | Enkripsi penyimpanan, akses terbatas RBAC, audit log ketat                                                             |

## 14. Keputusan Final (Sebelumnya Open Questions)

Seluruh pertanyaan terbuka pada draft sebelumnya telah diputuskan sebagai berikut:

1. **Layanan face matching**: menggunakan **face-api.js** (open-source, gratis, berjalan di sisi client) — bukan layanan cloud berbayar, agar tidak ada biaya operasional.
2. **Kuota karyawan**: default **30 karyawan** pada Fase 1, dapat diubah naik/turun kapan saja oleh admin melalui pengaturan.
3. **Mode absen manual/cadangan**: tersedia sebagai fallback, dan wajib disetujui oleh **Admin/HR**.
4. **Sistem payroll**: belum ditentukan; Fase 1 cukup menyediakan ekspor Excel/PDF (gratis, tanpa integrasi berbayar). Integrasi API otomatis (Fase 2) baru dipertimbangkan jika di masa depan ada sistem payroll spesifik yang mendukung API.
5. **Hosting/Deployment**: menggunakan kombinasi layanan gratis — Vercel/Netlify (frontend), Render/Railway free tier (backend), Supabase/Firebase free tier (database) — lihat Bagian 7.3.

### Catatan Keterbatasan Free Tier

Karena seluruh stack memakai layanan gratis, perlu disadari beberapa keterbatasan:

- Backend free tier bisa "tidur" saat tidak ada aktivitas dan butuh beberapa detik untuk aktif kembali saat diakses pertama kali.
- Kapasitas database & jumlah request per bulan terbatas — cukup untuk skala kuota 30 karyawan di 1 lokasi, namun perlu dipantau seiring pertumbuhan penggunaan.
- Jika ke depannya kebutuhan bertambah besar (lebih banyak karyawan, lebih banyak lokasi), perlu dipertimbangkan upgrade ke paket berbayar pada komponen yang menjadi bottleneck.

---

_Dokumen ini merupakan draft PRD lengkap dengan seluruh keputusan teknis dan kebijakan sudah difinalisasi (Bagian 14), berfokus pada solusi berbiaya nol (free tier/open-source) agar dapat dibangun dan diakses publik tanpa biaya operasional._
