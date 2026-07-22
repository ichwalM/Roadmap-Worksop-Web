# 01. HTML Dasar

HTML (HyperText Markup Language) adalah **struktur/kerangka** dari setiap halaman web. Kalau website itu rumah, HTML adalah pondasi dan temboknya.

> 📌 Mulai modul ini, seluruh journey memakai **satu studi kasus konsisten**: **Sistem Informasi Akademik (SIAKAD) Sederhana** dengan 3 data inti — **Mahasiswa**, **Dosen**, dan **Mata Kuliah** — persis mengikuti struktur tabel `mst_mahasiswa`, `mst_dosen`, `mst_matakuliah` di database `db_pendidikan` yang dipakai di modul 08 dan seterusnya.

## Tujuan Belajar

- Paham struktur dasar dokumen HTML.
- Bisa memakai tag-tag umum (heading, paragraf, list, gambar, link, tabel, form).
- Paham konsep **semantic HTML** dan kenapa itu penting.

## 1. Struktur Dasar Dokumen

```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Halaman Pertamaku</title>
</head>
<body>
    <h1>Halo, Dunia!</h1>
    <p>Ini paragraf pertamaku.</p>
</body>
</html>
```

Penjelasan tiap bagian:
- `<!DOCTYPE html>` — memberitahu browser ini dokumen HTML5.
- `<html lang="id">` — root elemen, `lang` menentukan bahasa (membantu screen reader & SEO).
- `<head>` — metadata: judul tab, charset, link CSS, meta tag untuk SEO. Tidak tampil di halaman.
- `<body>` — semua konten yang tampil di layar.

## 2. Tag-Tag Umum

### Teks
```html
<h1>Judul Paling Besar</h1>
<h2>Sub Judul</h2>
<h6>Judul Terkecil</h6>
<p>Paragraf biasa. <strong>Tebal (penting)</strong>, <em>miring (penekanan)</em>.</p>
```

### List
```html
<ul>
    <li>Item tanpa urutan (bullet)</li>
    <li>Item lain</li>
</ul>

<ol>
    <li>Item pertama (bernomor)</li>
    <li>Item kedua</li>
</ol>
```

### Link & Gambar
```html
<a href="https://laravel.com" target="_blank">Kunjungi Laravel</a>
<img src="logo.png" alt="Logo SIAKAD" width="200">
```
> `alt` wajib diisi — dipakai screen reader dan tampil kalau gambar gagal dimuat.

### Tabel

Tabel adalah cara paling natural menampilkan data tabular seperti daftar mata kuliah — bentuk ini nanti persis yang direplikasi lewat `<table>` di Blade (modul 11) dan lewat `.map()` di React (modul 16).

```html
<table>
    <thead>
        <tr>
            <th>Kode</th>
            <th>Nama Mata Kuliah</th>
            <th>SKS</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>IF101</td>
            <td>Pemrograman Web</td>
            <td>3</td>
        </tr>
        <tr>
            <td>IF102</td>
            <td>Basis Data</td>
            <td>3</td>
        </tr>
    </tbody>
</table>
```

### Form (paling penting — bakal dipakai terus sampai Laravel & React)
```html
<form action="/mahasiswa" method="POST">
    <label for="stambuk">Stambuk (NIM)</label>
    <input type="text" id="stambuk" name="stambuk" placeholder="Contoh: 20210011" required>

    <label for="name">Nama Lengkap</label>
    <input type="text" id="name" name="name" placeholder="Masukkan nama" required>

    <label for="jurusan">Jurusan</label>
    <select id="jurusan" name="jurusan">
        <option value="Teknik Informatika">Teknik Informatika</option>
        <option value="Sistem Informasi">Sistem Informasi</option>
        <option value="Ilmu Komputer">Ilmu Komputer</option>
    </select>

    <button type="submit">Simpan Data Mahasiswa</button>
</form>
```

Atribut penting pada `<form>`:
- `action` — URL tujuan saat form disubmit.
- `method` — `GET` (data di URL, untuk pencarian/filter) atau `POST` (data di body, untuk kirim/ubah data).

## 3. Semantic HTML

Dulu orang membungkus semua dengan `<div>`. Sekarang gunakan tag yang **menjelaskan makna**, bukan cuma tampilan — ini membantu SEO, aksesibilitas, dan keterbacaan kode.

```html
<header>
    <nav>...menu navigasi...</nav>
</header>

<main>
    <article>
        <h2>Pengumuman Akademik</h2>
        <p>Isi pengumuman...</p>
    </article>

    <aside>Konten pendukung, misal jadwal kuliah minggu ini</aside>
</main>

<footer>
    <p>&copy; 2026 SIAKAD Mini</p>
</footer>
```

| Tag | Kapan Dipakai |
|---|---|
| `<header>` | Bagian atas halaman/section (logo, judul, menu) |
| `<nav>` | Kumpulan link navigasi |
| `<main>` | Konten utama halaman (hanya 1 per halaman) |
| `<section>` | Kelompok konten yang punya tema tersendiri |
| `<article>` | Konten mandiri (artikel berita, post) |
| `<aside>` | Konten pendukung/sampingan |
| `<footer>` | Bagian bawah halaman |

## Studi Mini: Halaman Profil Mahasiswa Sederhana

Buat file `belajar-web/latihan-html/profil.html`:

```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Profil Mahasiswa</title>
</head>
<body>
    <header>
        <h1>Sistem Informasi Akademik (SIAKAD)</h1>
    </header>

    <main>
        <article>
            <h2>Profil Mahasiswa</h2>
            <img src="https://placehold.co/150" alt="Foto mahasiswa">
            <p>Stambuk: <strong>20210011</strong></p>
            <p>Nama: <strong>Ahmad Fauzi</strong></p>
            <p>Jurusan: Teknik Informatika</p>
            <ul>
                <li>Semester 5</li>
                <li>Status: Aktif</li>
            </ul>
        </article>
    </main>

    <footer>
        <p>&copy; 2026 SIAKAD Mini</p>
    </footer>
</body>
</html>
```

Buka file ini langsung di browser (klik dua kali, atau drag ke tab browser) — tanpa server sekalipun HTML statis bisa langsung jalan.

## Latihan

1. Buat halaman `tambah-mahasiswa.html` berisi form tambah data mahasiswa (stambuk, nama, jurusan) — persis field yang ada di tabel `mst_mahasiswa`.
2. Tambahkan tabel daftar 3 mata kuliah beserta kode dan jumlah SKS masing-masing (field `kode`, `nama_matakuliah`, `sks` — sesuai tabel `mst_matakuliah`).
3. Gunakan minimal 4 tag semantic (`header`, `main`, `section`/`article`, `footer`).

---
⬅️ [00. Persiapan Tools](../00-persiapan-tools/README.md) | ➡️ Lanjut ke [02. CSS Dasar](../02-css-dasar/README.md)
