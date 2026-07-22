# 19. Arsitektur Folder: Laravel & React

Halaman ini adalah **kamus referensi** — bukan tutorial berurutan. Setiap kali lupa "folder ini buat apa?", buka halaman ini. Sebagian contoh Laravel di sini diambil dari pola nyata sebuah proyek Laravel produksi berskala menengah (aplikasi pendaftaran dengan banyak role: student/admin/jury/editor), supaya terasa konkret, bukan folder kosong dari tutorial generik.

## Bagian 1 — Arsitektur Folder Laravel

### Peta Lengkap

```
├── app/
│   ├── Actions/            # Single-purpose action class (mis. Fortify\CreateNewUser)
│   ├── Console/Commands/   # Custom Artisan command
│   ├── Http/
│   │   ├── Controllers/    # Terima request, orkestrasi, kembalikan response
│   │   │   ├── Admin/      # Dikelompokkan per-role (proyek ini: Admin, Auth, Editor, Jury, Student)
│   │   │   ├── Auth/
│   │   │   ├── Editor/
│   │   │   ├── Jury/
│   │   │   └── Student/
│   │   ├── Middleware/     # Filter request sebelum sampai Controller (auth, role check, dll)
│   │   ├── Requests/       # Form Request — validasi & otorisasi input
│   │   │   ├── Admin/
│   │   │   ├── Auth/
│   │   │   └── Jury/
│   │   └── Responses/      # Custom response class (mis. LoginResponse — override alur Fortify)
│   ├── Mail/                # Mailable class — bentuk & isi email
│   ├── Models/               # Eloquent Model
│   │   ├── Content/          # Sub-domain: model terkait konten (artikel, testimoni, dll)
│   │   └── ReRegist/          # Sub-domain: model terkait daftar ulang
│   ├── Providers/             # Service Provider — tempat "mendaftarkan" service ke aplikasi
│   └── Services/               # Service Layer — logic bisnis (lihat modul 10)
├── bootstrap/                  # Bootstrap framework, cache konfigurasi
├── config/                     # File konfigurasi (app.php, database.php, cors.php, dll)
├── database/
│   ├── factories/               # Blueprint data dummy untuk testing/seeding
│   ├── migrations/              # Struktur tabel sebagai kode (lihat modul 08)
│   └── seeders/                  # Pengisi data awal/dummy
├── public/                       # Satu-satunya folder yang diakses langsung dari web
│   └── index.php                 # Entry point tunggal semua request PHP
├── resources/
│   ├── css/                       # CSS mentah sebelum di-build Vite
│   ├── js/                         # JS mentah (Alpine.js dll) sebelum di-build Vite
│   └── views/                       # Blade template
│       ├── admin/ · editor/ · jury/ · student/  # View dikelompokkan per-role, konsisten dengan Controllers/
│       ├── components/                # Blade Component (lihat modul 11)
│       ├── emails/                     # Template email
│       ├── layouts/                     # Layout dasar (@extends, @yield)
│       └── pdf/                          # Template untuk generate PDF
├── routes/
│   ├── web.php                            # Route halaman (HTML/Blade)
│   ├── api.php                             # Route API (JSON) — lihat modul 14 (belum ada di proyek ini, baru ditambahkan saat fitur API dibangun)
│   └── console.php                          # Definisi Artisan command berbasis closure
├── storage/                                  # File upload privat, log, cache
├── tests/                                      # Automated test (Feature & Unit)
└── artisan                                      # CLI tool
```

### Detail per Folder

#### `app/Http/Controllers/`
Menerima HTTP request, memanggil Service/Model, mengembalikan response. Proyek ini mengelompokkan Controller **per-role** (`Admin/`, `Student/`, `Jury/`, `Editor/`) — pola ini masuk akal untuk aplikasi dengan banyak jenis pengguna yang punya alur berbeda-beda, dibanding menaruh semua Controller rata di satu folder.

> Contoh: `app/Http/Controllers/ArticleController.php` untuk Controller di luar sub-folder role (konten publik yang tidak terikat role tertentu).

#### `app/Http/Requests/`
Form Request class (modul 09) — proyek dengan banyak role biasanya juga dikelompokkan per-role, contoh: `app/Http/Requests/Auth/StoreRegistrationRequest.php` dan `app/Http/Requests/Jury/StoreAssessmentRequest.php`.

#### `app/Http/Responses/`
Folder ini **tidak selalu ada** di proyek Laravel baru — muncul saat kita perlu **override** perilaku bawaan sebuah package. Contoh: `app/Http/Responses/LoginResponse.php` meng-override ke mana user diarahkan setelah login berhasil, sebuah kontrak yang disediakan Laravel Fortify (package autentikasi).

#### `app/Actions/`
Class yang membungkus **satu tindakan spesifik** — lebih sempit cakupannya dibanding Service. Contoh: `app/Actions/Fortify/` berisi action seperti "buat user baru saat register" atau "update password" — dipakai Laravel Fortify secara internal.

#### `app/Services/`
Logic bisnis (modul 10). Contoh: `app/Services/ImageOptimizationService.php`.

#### `app/Models/`
Representasi tabel database. Sub-folder seperti `Models/Content/` dan `Models/ReRegist/` dipakai untuk mengelompokkan model berdasarkan **domain/konteks bisnis** ketika jumlah model sudah banyak (proyek berskala menengah ke atas bisa punya 20+ model) — bukan aturan wajib Laravel, tapi konvensi tim untuk menjaga folder tetap terbaca.

#### `app/Providers/`
Service Provider — "titik pendaftaran" berbagai hal ke dalam aplikasi (binding interface ke implementasi, event listener, dll), dijalankan sekali saat aplikasi boot. `AppServiceProvider.php` adalah provider utama aplikasi; `FortifyServiceProvider.php` mengonfigurasi package autentikasi Fortify.

#### `routes/`
| File | Isi |
|---|---|
| `web.php` | Route dengan middleware `web` (session, CSRF) — untuk halaman Blade |
| `api.php` | Route dengan middleware `api` (stateless) — untuk JSON API (modul 14) |
| `console.php` | Mendefinisikan Artisan command custom dengan closure, terpisah dari `app/Console/Commands/` |

#### `resources/views/`
Struktur folder view **mengikuti struktur Controller** (per-role) — ini konvensi yang baik: kalau tahu Controller-nya di `Admin/`, view-nya hampir pasti di `views/admin/`.

#### `database/`
| Folder | Isi | Modul Terkait |
|---|---|---|
| `migrations/` | Struktur tabel | Modul 08 |
| `factories/` | Blueprint data dummy | Modul 08 |
| `seeders/` | Skrip pengisi data | Modul 08 |

#### `public/`
Folder ini **satu-satunya** yang boleh diakses langsung oleh web server (Apache/Nginx document root mengarah ke sini). Semua request PHP masuk lewat `public/index.php`, kode aplikasi sesungguhnya di `app/` **tidak pernah** diakses langsung dari URL — ini alasan keamanan penting: kredensial di `.env` dan kode di `app/` tidak bisa "diketik langsung" di browser.

### Prinsip Umum Penempatan Kode di Laravel

| Kalau kamu menulis... | Taruh di |
|---|---|
| Query/struktur tabel | `database/migrations/`, `app/Models/` |
| Aturan validasi input | `app/Http/Requests/` |
| Logic bisnis (aturan, orkestrasi) | `app/Services/` |
| Terima request, panggil Service, kembalikan response | `app/Http/Controllers/` |
| Tampilan HTML | `resources/views/` |
| Pemetaan URL | `routes/web.php` atau `routes/api.php` |
| Perintah CLI custom | `app/Console/Commands/` |
| Filter sebelum request diproses (auth, role check) | `app/Http/Middleware/` |

---

## Bagian 2 — Arsitektur Folder React (Vite)

### Peta Lengkap (Proyek yang Sudah Berkembang)

Struktur dasar Vite (modul 15) sangat minim. Begitu proyek berkembang (seperti studi kasus modul 18), konvensi berikut umum dipakai:

```
src/
├── api/                 # Konfigurasi HTTP client (axios instance, base URL)
│   └── axios.js
├── assets/               # Gambar, font, file statis yang di-import di kode
├── components/            # Komponen UI yang dipakai ulang di banyak tempat
│   ├── MahasiswaCard.jsx
│   └── MahasiswaForm.jsx
├── hooks/                  # Custom hook (logic yang dipakai ulang lintas komponen)
│   └── useMahasiswa.js
├── pages/                   # Komponen level halaman (biasanya 1:1 dengan route)
│   ├── HomePage.jsx
│   └── MahasiswaPage.jsx
├── context/                   # React Context (state global — auth user, tema, dll)
│   └── AuthContext.jsx
├── utils/                       # Fungsi bantu murni (format tanggal, format angka, dll)
├── App.jsx                       # Root komponen, biasanya berisi routing
├── main.jsx                       # Entry point aplikasi
└── index.css                       # CSS global
```

### Detail per Folder

#### `src/api/`
Konfigurasi terpusat untuk komunikasi dengan backend — lihat modul 17-18 (`axios.js`). Kalau base URL API berubah (misal pindah dari `localhost:8000` ke domain produksi), cukup ubah **satu file**, bukan mencari-cari di seluruh komponen.

#### `src/components/`
Komponen kecil, **fokus pada tampilan**, idealnya tidak tahu-menahu soal "dari mana data berasal" — cukup terima lewat props (modul 16). Contoh: `MahasiswaCard` tidak peduli apakah datanya dari API asli atau data dummy untuk testing.

#### `src/hooks/`
Logic yang melibatkan state + effect, diekstrak supaya bisa dipakai ulang tanpa duplikasi (modul 17-18, contoh `useMahasiswa`). Analogi: **hooks di React ≈ Service Layer di Laravel** — sama-sama memisahkan "logic" dari "tampilan/kontrol alur".

#### `src/pages/`
Kalau proyek pakai routing (React Router — di luar scope journey ini, tapi lazim ditambahkan), tiap halaman biasanya jadi 1 komponen di sini, yang **merangkai** beberapa `components/` lebih kecil — mirip peran Controller di Laravel yang "merangkai" View + data.

#### `src/context/`
Untuk data yang perlu diakses banyak komponen di level berbeda tanpa mengoper props berlapis-lapis (disebut "prop drilling") — misalnya data user yang sedang login. Di luar scope journey ini, tapi penting diketahui namanya untuk belajar lanjutan.

#### `src/assets/` vs `public/`
| | `src/assets/` | `public/` (di root proyek Vite) |
|---|---|---|
| Diproses build tool? | Ya (di-hash nama file, dioptimasi) | Tidak, disalin apa adanya |
| Cara akses di kode | `import logo from './assets/logo.png'` | `<img src="/logo.png" />` (path absolut) |
| Cocok untuk | Gambar yang dipakai di komponen | Favicon, `robots.txt`, file yang butuh nama tetap |

### Perbandingan Konseptual: "Siapa Analog Siapa?"

Tabel ini membantu transfer intuisi dari yang sudah dipelajari (Laravel) ke yang baru (React), dan sebaliknya:

| Laravel | React | Kesamaan Peran |
|---|---|---|
| Controller | Page component (`src/pages/`) | Merangkai/orkestrasi, titik masuk sebuah "halaman" |
| Service | Custom Hook (`src/hooks/`) | Memisahkan logic dari lapisan tampilan/kontrol |
| Blade Component (`<x-card>`) | Component (`src/components/`) | Potongan UI kecil yang dipakai ulang |
| Model (Eloquent) | (tidak ada langsung — data datang dari API sebagai JSON) | Representasi data |
| Form Request (validasi) | Validasi di dalam handler `onSubmit` / library seperti Zod | Memastikan input valid sebelum diproses |
| `routes/web.php` / `routes/api.php` | React Router config (di luar scope journey ini) | Memetakan "alamat" ke komponen/handler |
| `.env` | File `.env` Vite (`VITE_API_URL=...`) | Konfigurasi yang beda per environment |

### Prinsip Umum Penempatan Kode di React

| Kalau kamu menulis... | Taruh di |
|---|---|
| Tampilan kecil, dipakai ulang (card, button, form) | `src/components/` |
| Logic fetch/mutasi data + state terkait | `src/hooks/` |
| Komponen besar yang mewakili 1 halaman penuh | `src/pages/` |
| Konfigurasi axios/fetch | `src/api/` |
| State yang perlu diakses banyak komponen | `src/context/` |
| Fungsi murni tanpa state (format tanggal, dll) | `src/utils/` |

## Kesimpulan

Baik Laravel maupun React pada akhirnya menganut prinsip arsitektur yang sama: **pisahkan tampilan, logic bisnis/data, dan orkestrasi/kontrol alur ke lapisan masing-masing**. Begitu prinsip ini melekat (bukan sekadar hafal nama foldernya), pindah ke framework/library lain di masa depan (Vue, Next.js, Symfony, dll) akan jauh lebih cepat dikuasai — karena yang berubah hanya sintaks, bukan cara berpikirnya.

---
⬅️ [18. Studi Kasus: Integrasi Full-Stack](../18-studi-kasus-fullstack-integrasi/README.md) | 🏠 [Kembali ke Daftar Isi Journey](../README.md)
