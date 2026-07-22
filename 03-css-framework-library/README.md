# 03. CSS Framework & Library

Setelah paham CSS murni, saatnya kenal alat bantu yang mempercepat styling: **library** dan **framework** CSS. Modul ini fokus ke **Tailwind CSS v4** (pilihan paling umum di proyek Laravel modern saat ini), tapi tetap membandingkan dengan alternatif populer supaya kamu paham pilihan yang ada.

## Tujuan Belajar

- Paham beda **library** vs **framework** (konsep ini juga akan muncul lagi saat bicara React vs Laravel).
- Paham 2 pendekatan utama CSS framework: **komponen siap pakai** (Bootstrap) vs **utility-first** (Tailwind).
- Bisa setup dan pakai Tailwind CSS dasar.

## 1. Library vs Framework — Konsep Penting

| | Library | Framework |
|---|---|---|
| **Kontrol alur** | Kode **kamu** yang memanggil library, kapan pun kamu mau | **Framework** yang memanggil kode kamu, sesuai aturan/struktur yang ditentukan ("Inversion of Control") |
| **Kebebasan** | Bebas dipakai sebagian saja, dicampur bebas | Harus ikut struktur & konvensi yang sudah ditetapkan |
| **Contoh di CSS** | Tailwind CSS (kumpulan utility class, kamu yang susun) | Bootstrap (komponen jadi dengan struktur HTML tertentu) |
| **Contoh di JS** | React (kamu tentukan struktur sendiri) | Angular, Laravel (di sisi backend) |

Analogi sederhana: **library itu seperti perkakas** (obeng, palu) yang kamu ambil sesuai kebutuhan. **Framework itu seperti cetakan rumah** — kamu ikuti aturan mainnya, tapi jadi lebih cepat & konsisten membangun sesuatu yang lengkap.

> React sebenarnya secara teknis adalah *library* (untuk UI saja), bukan framework penuh — tapi sering disebut "framework" secara longgar. Ini akan dibahas lagi di modul 15.

## 2. Bootstrap — Component-Based Framework

Bootstrap menyediakan **komponen siap pakai** dengan class bernama semantik.

```html
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">

<button class="btn btn-primary">Tambah Mata Kuliah</button>

<div class="card" style="width: 18rem;">
    <div class="card-body">
        <h5 class="card-title">Pemrograman Web</h5>
        <p class="card-text">Kode: IF101 · 3 SKS</p>
    </div>
</div>
```

Kelebihan: cepat untuk prototyping, konsisten. Kekurangan: perlu override CSS kalau mau desain custom, semua project Bootstrap cenderung "terlihat mirip".

## 3. Tailwind CSS — Utility-First Framework

Tailwind tidak menyediakan komponen jadi, tapi **utility class kecil** yang dikombinasikan langsung di HTML.

```html
<button class="bg-blue-600 hover:bg-blue-700 text-white font-semibold py-2 px-4 rounded-lg">
    Tambah Mata Kuliah
</button>

<div class="border border-gray-200 rounded-lg p-6 shadow-sm">
    <h5 class="text-lg font-bold">Pemrograman Web</h5>
    <p class="text-gray-600">Kode: IF101 · 3 SKS</p>
</div>
```

Kelebihan: sangat fleksibel, desain custom tidak perlu tulis CSS terpisah, ukuran file produksi kecil (unused class dibuang otomatis). Kekurangan: HTML jadi "ramai" class, ada kurva belajar menghafal nama utility.

### Setup Tailwind

```bash
npm install tailwindcss @tailwindcss/vite
```

`vite.config.js`:
```js
import { defineConfig } from 'vite';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
    plugins: [tailwindcss()],
});
```

`resources/css/app.css`:
```css
@import "tailwindcss";
```

> Proyek Laravel + Tailwind v4 di produksi biasanya memakai persis pola ini — cek `tailwind.config.js` dan `vite.config.js` di root proyek untuk melihat konfigurasi nyatanya.

### Tabel Padanan Class Umum

| CSS Murni | Tailwind |
|---|---|
| `display: flex;` | `flex` |
| `justify-content: center;` | `justify-center` |
| `padding: 16px;` | `p-4` |
| `margin-top: 8px;` | `mt-2` |
| `background-color: #2563eb;` | `bg-blue-600` |
| `border-radius: 8px;` | `rounded-lg` |
| `font-weight: bold;` | `font-bold` |

## 4. Kapan Pakai yang Mana?

- **Belajar cepat prototipe / tim non-frontend spesialis** → Bootstrap.
- **Butuh desain custom, kontrol penuh, dan proyek jangka panjang** → Tailwind.
- Keduanya **valid** — pilihan tergantung kebutuhan tim, bukan "salah satu lebih benar".

## Studi Mini: Ubah Card Mata Kuliah ke Tailwind

```html
<div class="max-w-sm mx-auto bg-white rounded-xl shadow-md overflow-hidden">
    <div class="p-5">
        <span class="text-xs font-semibold text-blue-600">IF101</span>
        <h3 class="text-xl font-bold text-gray-800">Pemrograman Web</h3>
        <p class="text-gray-500 mt-1">3 SKS</p>
        <button class="mt-4 w-full bg-blue-600 hover:bg-blue-700 text-white py-2 rounded-lg transition">
            Lihat Detail
        </button>
    </div>
</div>
```

## Latihan

1. Ubah semua styling `tambah-mahasiswa.html` (dari modul 02) menjadi Tailwind CSS via CDN: `<script src="https://cdn.tailwindcss.com"></script>` (cukup untuk latihan, meski untuk produksi harus pakai Vite build seperti di atas).
2. Buat 3 card mata kuliah (kode, nama mata kuliah, SKS) memakai grid Tailwind (`grid grid-cols-1 md:grid-cols-3 gap-6`).
3. Bandingkan: tulis satu komponen yang sama dengan Bootstrap dan Tailwind, rasakan bedanya.

---
⬅️ [02. CSS Dasar](../02-css-dasar/README.md) | ➡️ Lanjut ke [04. JavaScript Dasar](../04-javascript-dasar/README.md)
