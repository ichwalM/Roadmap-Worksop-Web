# 02. CSS Dasar

CSS (Cascading Style Sheets) mengatur **tampilan** dari HTML: warna, ukuran, jarak, posisi, dan layout secara keseluruhan.

## Tujuan Belajar

- Paham cara menghubungkan CSS ke HTML.
- Paham selector, box model, dan unit ukuran.
- Bisa membuat layout dengan Flexbox dan Grid.
- Paham dasar responsive design.

## 1. Cara Menghubungkan CSS

```html
<head>
    <!-- Cara 1: file eksternal (paling direkomendasikan) -->
    <link rel="stylesheet" href="style.css">

    <!-- Cara 2: inline di dalam <style> -->
    <style>
        body { font-family: sans-serif; }
    </style>
</head>
<body>
    <!-- Cara 3: inline attribute (hindari, susah maintain) -->
    <p style="color: red;">Teks merah</p>
</body>
```

## 2. Selector

```css
/* Selector tag */
p { color: #333; }

/* Selector class (paling sering dipakai) */
.card { border: 1px solid #ddd; padding: 16px; }

/* Selector id (unik, satu elemen saja) */
#header-utama { background: #1e3a8a; }

/* Selector turunan */
.card h2 { font-size: 20px; }

/* Selector pseudo-class */
a:hover { text-decoration: underline; }
input:focus { border-color: blue; }

/* Selector atribut */
input[type="email"] { background: #f9fafb; }
```

## 3. Box Model

Setiap elemen HTML adalah sebuah "kotak" yang terdiri dari 4 lapisan:

```
┌─────────────────────────────┐
│         margin              │
│  ┌───────────────────────┐  │
│  │       border           │  │
│  │  ┌─────────────────┐  │  │
│  │  │    padding       │  │  │
│  │  │  ┌───────────┐  │  │  │
│  │  │  │  content   │  │  │  │
│  │  │  └───────────┘  │  │  │
│  │  └─────────────────┘  │  │
│  └───────────────────────┘  │
└─────────────────────────────┘
```

```css
.card {
    width: 300px;
    padding: 16px;       /* jarak dalam, antara border dan konten */
    border: 1px solid #ccc;
    margin: 8px;         /* jarak luar, antar elemen */
    box-sizing: border-box; /* width sudah termasuk padding+border, hindari kejutan ukuran */
}
```

> **Tips penting:** selalu tambahkan `* { box-sizing: border-box; }` di awal CSS-mu — ini kebiasaan standar industri supaya perhitungan lebar elemen tidak membingungkan.

## 4. Flexbox — Layout 1 Dimensi

Paling sering dipakai untuk navbar, card horizontal, atau align konten.

```css
.navbar {
    display: flex;
    justify-content: space-between; /* elemen tersebar rata */
    align-items: center;             /* rata tengah vertikal */
    gap: 16px;
}
```

```html
<nav class="navbar">
    <div class="logo">SIAKAD</div>
    <ul style="display: flex; gap: 12px;">
        <li>Beranda</li>
        <li>Mahasiswa</li>
        <li>Dosen</li>
        <li>Mata Kuliah</li>
    </ul>
</nav>
```

## 5. CSS Grid — Layout 2 Dimensi

Cocok untuk layout halaman atau grid kartu.

```css
.grid-mahasiswa {
    display: grid;
    grid-template-columns: repeat(3, 1fr); /* 3 kolom sama lebar */
    gap: 20px;
}
```

```html
<div class="grid-mahasiswa">
    <div class="card">Ahmad Fauzi — Teknik Informatika</div>
    <div class="card">Siti Rahma — Sistem Informasi</div>
    <div class="card">Budi Santoso — Ilmu Komputer</div>
</div>
```

## 6. Responsive Design

```css
/* Default: mobile-first */
.grid-mahasiswa {
    grid-template-columns: 1fr; /* 1 kolom di HP */
}

/* Tablet ke atas */
@media (min-width: 768px) {
    .grid-mahasiswa {
        grid-template-columns: repeat(2, 1fr);
    }
}

/* Desktop */
@media (min-width: 1024px) {
    .grid-mahasiswa {
        grid-template-columns: repeat(3, 1fr);
    }
}
```

Jangan lupa meta viewport di HTML (lihat modul 01) — tanpa ini, media query tidak akan berfungsi benar di HP.

## Studi Mini: Styling Halaman Profil Mahasiswa

Lanjutkan file `profil.html` dari modul sebelumnya, tambahkan `style.css`:

```css
* { box-sizing: border-box; margin: 0; padding: 0; }

body {
    font-family: 'Segoe UI', sans-serif;
    background: #f3f4f6;
    color: #1f2937;
}

header {
    background: #1e3a8a;
    color: white;
    padding: 20px;
    text-align: center;
}

main {
    max-width: 700px;
    margin: 24px auto;
    background: white;
    border-radius: 8px;
    padding: 24px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

article img {
    border-radius: 50%;
    display: block;
    margin-bottom: 16px;
}

footer {
    text-align: center;
    padding: 16px;
    color: #6b7280;
    font-size: 14px;
}
```

## Latihan

1. Ubah halaman `tambah-mahasiswa.html` dari modul 01 supaya form-nya rapi: label di atas input, input full-width, tombol submit berwarna.
2. Buat grid 3 kolom untuk menampilkan daftar mata kuliah (pakai CSS Grid), yang otomatis jadi 1 kolom di layar HP (pakai media query).
3. Tambahkan efek `:hover` pada tombol dan card.

---
⬅️ [01. HTML Dasar](../01-html-dasar/README.md) | ➡️ Lanjut ke [03. CSS Framework & Library](../03-css-framework-library/README.md)
