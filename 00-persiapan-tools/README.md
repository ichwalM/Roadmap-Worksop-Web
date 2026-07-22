# 00. Persiapan Tools

Sebelum menulis kode apapun, siapkan dulu "meja kerja"-nya. Modul ini daftar semua software yang dibutuhkan sepanjang journey ini, dari HTML polos sampai Laravel + React.

## Tujuan Belajar

- Tahu tools apa saja yang dibutuhkan dan kenapa.
- Berhasil menginstal semuanya dan memverifikasi versinya lewat terminal.

## Daftar Tools

| Tools | Fungsi | Dibutuhkan mulai modul |
|---|---|---|
| **Text Editor / IDE** (VS Code direkomendasikan) | Menulis kode | 01 |
| **Browser** (Chrome/Firefox + DevTools) | Menjalankan & debug HTML/CSS/JS | 01 |
| **Git** | Version control | 01 (opsional), wajib sejak 06 |
| **PHP** (≥ 8.2) | Menjalankan kode PHP & Laravel | 05 |
| **Composer** | Package manager PHP (dipakai Laravel) | 06 |
| **Node.js + NPM** (≥ 18) | Menjalankan tooling frontend (Vite, build asset) | 04, 15 |
| **Database** (MySQL/MariaDB, atau SQLite untuk belajar) | Penyimpanan data | 08 |
| **Laragon / XAMPP / Herd** (opsional, Windows/Mac) | Paket instan PHP+MySQL+Apache/Nginx | 06 |
| **Postman / Insomnia / Thunder Client** | Menguji API tanpa frontend | 13 |

## Instalasi Cepat

### 1. VS Code
Download di [code.visualstudio.com](https://code.visualstudio.com). Extension yang berguna: `PHP Intelephense`, `Laravel Blade Snippets`, `ES7+ React/Redux/React-Native snippets`, `Tailwind CSS IntelliSense`.

### 2. Git
```bash
git --version
```
Kalau belum ada, download di [git-scm.com](https://git-scm.com).

### 3. PHP & Composer
Cara termudah di Windows: install **Laragon** ([laragon.org](https://laragon.org)) — sudah termasuk PHP, MySQL, Composer, dan Nginx/Apache dalam satu installer.

Verifikasi:
```bash
php -v
composer -V
```

### 4. Node.js
Download versi **LTS** di [nodejs.org](https://nodejs.org).

Verifikasi:
```bash
node -v
npm -v
```

### 5. Database
Untuk belajar, **SQLite** paling praktis (tidak perlu server terpisah, cukup 1 file). Untuk yang menyamai environment produksi proyek ini, pakai **MySQL** (sudah termasuk di Laragon/XAMPP).

## Verifikasi Semua Terinstal

Jalankan semua perintah ini, pastikan tidak ada error:

```bash
git --version
php -v
composer -V
node -v
npm -v
```

Contoh output yang diharapkan (versi boleh beda, yang penting bukan pesan "command not found"):
```
git version 2.44.0
PHP 8.3.6 (cli)
Composer version 2.7.1
v20.11.0
10.2.4
```

## Struktur Folder Kerja

Buat satu folder khusus untuk latihan sepanjang journey ini, terpisah dari proyek utama:

```bash
mkdir belajar-web
cd belajar-web
```

Tiap modul nanti akan menyebut folder latihan sendiri di dalam sini (mis. `belajar-web/latihan-html`, `belajar-web/todo-laravel`, dst).

## Latihan

1. Instal semua tools di atas.
2. Buat file `cek-tools.txt` berisi output dari 5 perintah verifikasi di atas — ini jadi bukti environment sudah siap sebelum lanjut ke modul berikutnya.

---
⬅️ [Kembali ke Daftar Isi](../README.md) | ➡️ Lanjut ke [01. HTML Dasar](../01-html-dasar/README.md)
